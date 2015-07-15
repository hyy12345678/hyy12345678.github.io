---
layout: post
title:  Struts2 Core Workflow And Principle
date:   2015-07-15 15:03:40
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---
# struts2核心工作流程与原理

官方站点提供的struts2的结构图   
![](http://www.evget.com/images/article/08072801.png)

一个请求在Struts2框架中的处理大概分为以下几个步骤:

- 1.客户端提起一个（HttpServletRequest）请求,如上文在浏览器中输入”http://localhost:8080/TestMvc/add.action”就是提起一个（HttpServletRequest）请求。

- 2.请求被提交到一系列（主要是三层）的过滤器（Filter），如（ActionContextCleanUp、其他过滤器（SiteMesh等）、 FilterDispatcher）。注意这里是有顺序的，先ActionContextCleanUp，再其他过滤器（SiteMesh等）、最后到FilterDispatcher。

- 3.FilterDispatcher是控制器的核心，就是mvc中c控制层的核心。下面粗略的分析下我理解的FilterDispatcher工作流程和原理：FilterDispatcher进行初始化并启用核心doFilter

doFilter的代码如下:   

    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException ...{
            HttpServletRequest request = (HttpServletRequest) req;
            HttpServletResponse response = (HttpServletResponse) res;
            ServletContext servletContext = filterConfig.getServletContext();
            // 在这里处理了HttpServletRequest和HttpServletResponse。
            DispatcherUtils du = DispatcherUtils.getInstance();
            du.prepare(request, response);
            //正如这个方法名字一样进行locale、encoding以及特殊request parameters设置
            try ...{
                request = du.wrapRequest(request, servletContext);//对request进行包装
            } catch (IOException e) ...{
                String message = "Could not wrap servlet request with MultipartRequestWrapper!";
                LOG.error(message, e);
                throw new ServletException(message, e);
            }
                    ActionMapperIF mapper = ActionMapperFactory.getMapper();//得到action的mapper
            ActionMapping mapping = mapper.getMapping(request);
            // 得到action 的 mapping
            if (mapping == null) ...{
                // there is no action in this request, should we look for a static resource?
                String resourcePath = RequestUtils.getServletPath(request);
                if ("".equals(resourcePath) && null != request.getPathInfo()) ...{
                    resourcePath = request.getPathInfo();
                }
                if ("true".equals(Configuration.get(WebWorkConstants.WEBWORK_SERVE_STATIC_CONTENT)) 
                        && resourcePath.startsWith("/webwork")) ...{
                    String name = resourcePath.substring("/webwork".length());
                    findStaticResource(name, response);
                } else ...{
                    // this is a normal request, let it pass through
                    chain.doFilter(request, response);
                }
                // WW did its job here
                return;
            }
            Object o = null;
            try ...{
                //setupContainer(request);
                o = beforeActionInvocation(request, servletContext);
                //整个框架最最核心的方法，下面分析
                du.serviceAction(request, response, servletContext, mapping);
            } finally ...{
                afterActionInvocation(request, servletContext, o);
                ActionContext.setContext(null);
            }
        }

> du.serviceAction(request, response, servletContext, mapping);
//这个方法询问ActionMapper是否需要调用某个Action来处理这个（request）请求，如果ActionMapper决定需要调用某个Action，FilterDispatcher把请求的处理交给ActionProxy

    public void serviceAction(HttpServletRequest request, HttpServletResponse response, String namespace, String actionName, Map requestMap, Map parameterMap, Map sessionMap, Map applicationMap) ...{ 
            HashMap extraContext = createContextMap(requestMap, parameterMap, sessionMap, applicationMap, request, response, getServletConfig());  
            //实例化Map请求,询问ActionMapper是否需要调用某个Action来处理这个（request）请求
            extraContext.put(SERVLET_DISPATCHER, this); 
            OgnlValueStack stack = (OgnlValueStack) request.getAttribute(ServletActionContext.WEBWORK_VALUESTACK_KEY); 
            if (stack != null) ...{ 
                extraContext.put(ActionContext.VALUE_STACK,new OgnlValueStack(stack)); 
            } 
            try ...{ 
                ActionProxy proxy = ActionProxyFactory.getFactory().createActionProxy(namespace, actionName, extraContext); 
                
                //这里actionName是通过两道getActionName解析出来的, FilterDispatcher把请求的处理交给ActionProxy，下面是ServletDispatcher的 TODO: 
                
                request.setAttribute(ServletActionContext.WEBWORK_VALUESTACK_KEY, proxy.getInvocation().getStack()); 
                proxy.execute(); 
                //通过代理模式执行ActionProxy
                
                if (stack != null)...{ 
                    request.setAttribute(ServletActionContext.WEBWORK_VALUESTACK_KEY,stack); 
                } 
            } catch (ConfigurationException e) ...{ 
                log.error("Could not find action", e); 
                sendError(request, response, HttpServletResponse.SC_NOT_FOUND, e); 
            } catch (Exception e) ...{ 
                log.error("Could not execute action", e); 
                sendError(request, response, HttpServletResponse.SC_INTERNAL_SERVER_ERROR, e); 
            } 
    } 

> FilterDispatcher询问ActionMapper是否需要调用某个Action来处理这个（request）请求，如果ActionMapper决定需要调用某个Action，FilterDispatcher把请求的处理交给ActionProxy。

- 4.ActionProxy通过Configuration Manager（struts.xml）询问框架的配置文件，找到需要调用的Action类.

如上文的struts.xml配置

    <?xml version="1.0" encoding="GBK"?>
     <!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN" "http://struts.apache.org/dtds/struts-2.0.dtd">
     <struts>
         <include file="struts-default.xml"/>
         <package name="struts2" extends="struts-default">
             <action name="add" 
                 class="edisundong.AddAction" >
                 <result>add.jsp</result>
             </action>    
         </package>
     </struts> 

> 如果提交请求的是add.action，那么找到的Action类就是edisundong.AddAction。

- 5.ActionProxy创建一个ActionInvocation的实例，同时ActionInvocation通过代理模式调用Action。但在调用之前ActionInvocation会根据配置加载Action相关的所有Interceptor。（Interceptor是struts2另一个核心级的概念）

下面我们来看看ActionInvocation是如何工作的：
ActionInvocation 是Xworks 中Action 调度的核心。而对Interceptor 的调度，也正是由ActionInvocation负责。ActionInvocation 是一个接口， 而DefaultActionInvocation 则是Webwork 对ActionInvocation的默认实现。

Interceptor 的调度流程大致如下：

- ActionInvocation初始化时，根据配置，加载Action相关的所有Interceptor。
- 通过ActionInvocation.invoke方法调用Action实现时，执行Interceptor。

Interceptor将很多功能从我们的Action中独立出来，大量减少了我们Action的代码，独立出来的行为具有很好的重用性。XWork、WebWork的许多功能都是有Interceptor实现，可以在配置文件中组装Action用到的Interceptor，它会按照你指定的顺序，在Action执行前后运行。   
那什么是拦截器?拦截器就是AOP（Aspect-Oriented Programming）的一种实现。（AOP是指用于在某个方法或字段被访问之前，进行拦截然后在之前或之后加入某些操作。）   
拦截器的例子这里就不展开了   
struts-default.xml文件摘取的内容：

    < interceptor name ="alias" class ="com.opensymphony.xwork2.interceptor.AliasInterceptor" /> 
    < interceptor name ="autowiring" class ="com.opensymphony.xwork2.spring.interceptor.ActionAutowiringInterceptor" /> 
    < interceptor name ="chain" class ="com.opensymphony.xwork2.interceptor.ChainingInterceptor" /> 
    < interceptor name ="conversionError" class ="org.apache.struts2.interceptor.StrutsConversionErrorInterceptor" /> 
    < interceptor name ="createSession" class ="org.apache.struts2.interceptor.CreateSessionInterceptor" /> 
    < interceptor name ="debugging" class ="org.apache.struts2.interceptor.debugging.DebuggingInterceptor" /> 
    < interceptor name ="external-ref" class ="com.opensymphony.xwork2.interceptor.ExternalReferencesInterceptor" /> 
    < interceptor name ="execAndWait" class ="org.apache.struts2.interceptor.ExecuteAndWaitInterceptor" /> 
    < interceptor name ="exception" class ="com.opensymphony.xwork2.interceptor.ExceptionMappingInterceptor" /> 
    < interceptor name ="fileUpload" class ="org.apache.struts2.interceptor.FileUploadInterceptor" /> 
    < interceptor name ="i18n" class ="com.opensymphony.xwork2.interceptor.I18nInterceptor" /> 
    < interceptor name ="logger" class ="com.opensymphony.xwork2.interceptor.LoggingInterceptor" /> 
    < interceptor name ="model-driven" class ="com.opensymphony.xwork2.interceptor.ModelDrivenInterceptor" /> 
    < interceptor name ="scoped-model-driven" class ="com.opensymphony.xwork2.interceptor.ScopedModelDrivenInterceptor" /> 
    < interceptor name ="params" class ="com.opensymphony.xwork2.interceptor.ParametersInterceptor" /> 
    < interceptor name ="prepare" class ="com.opensymphony.xwork2.interceptor.PrepareInterceptor" /> 
    < interceptor name ="static-params" class ="com.opensymphony.xwork2.interceptor.StaticParametersInterceptor" /> 
    < interceptor name ="scope" class ="org.apache.struts2.interceptor.ScopeInterceptor" /> 
    < interceptor name ="servlet-config" class ="org.apache.struts2.interceptor.ServletConfigInterceptor" /> 
    < interceptor name ="sessionAutowiring" class ="org.apache.struts2.spring.interceptor.SessionContextAutowiringInterceptor" /> 
    < interceptor name ="timer" class ="com.opensymphony.xwork2.interceptor.TimerInterceptor" /> 
    < interceptor name ="token" class ="org.apache.struts2.interceptor.TokenInterceptor" /> 
    < interceptor name ="token-session" class ="org.apache.struts2.interceptor.TokenSessionStoreInterceptor" /> 
    < interceptor name ="validation" class ="com.opensymphony.xwork2.validator.ValidationInterceptor" /> 
    < interceptor name ="workflow" class ="com.opensymphony.xwork2.interceptor.DefaultWorkflowInterceptor" /> 
    < interceptor name ="store" class ="org.apache.struts2.interceptor.MessageStoreInterceptor" /> 
    < interceptor name ="checkbox" class ="org.apache.struts2.interceptor.CheckboxInterceptor" /> 
    < interceptor name ="profiling" class ="org.apache.struts2.interceptor.ProfilingActivationInterceptor" /> 

一旦Action执行完毕，ActionInvocation负责根据struts.xml中的配置找到对应的返回结果。如上文中将结构返回“add.jsp”，但大部分时候都是返回另外一个action，那么流程又得走一遍……