title: reivew_spring_mvc
tags:
  - springmvc
categories:
  - springmvc
date: 2018-01-18 21:19:00
---
# springmvc是什么
- Spring Web MVC是一种基于Java的实现了Web MVC设计模式的请求驱动类型的轻量级Web框架
- 将web层进行职责解耦
- 工作者模式的实现???
# 帮我们做什么
- 让我们能非常简单的设计出干净的Web层和薄薄的Web层；
- 进行更简洁的Web层的开发；
- 天生与Spring框架集成（如IoC容器、AOP等）；
- 提供强大的约定大于配置的契约式编程支持；
- 能简单的进行Web层的单元测试；
- 支持灵活的URL到页面控制器的映射；
- 非常容易与其他视图技术集成，如Velocity、FreeMarker等等，因为模型数据不放在特定的API里，而是放在一个Model里（Map数据结构实现，因此很容易被其他框架使用）；
- 非常灵活的数据验证、格式化和数据绑定机制，能使用任何对象进行数据绑定，不必实现特定框架的API；
- 提供一套强大的JSP标签库，简化JSP开发；
- 支持灵活的本地化、主题等解析；
- 更加简单的异常处理；
- 对静态资源的支持；
- 支持Restful风格。

# 架构

![upload successful](/images/pasted-28.png)

![upload successful](/images/pasted-29.png)

![upload successful](/images/pasted-85.png)

```
//前端控制器分派方法  
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {  
        HttpServletRequest processedRequest = request;  
        HandlerExecutionChain mappedHandler = null;  
        int interceptorIndex = -1;  
  
        try {  
            ModelAndView mv;  
            boolean errorView = false;  
  
            try {  
                   //检查是否是请求是否是multipart（如文件上传），如果是将通过MultipartResolver解析  
                processedRequest = checkMultipart(request);  
                   //步骤2、请求到处理器（页面控制器）的映射，通过HandlerMapping进行映射  
                mappedHandler = getHandler(processedRequest, false);  
                if (mappedHandler == null || mappedHandler.getHandler() == null) {  
                    noHandlerFound(processedRequest, response);  
                    return;  
                }  
                   //步骤3、处理器适配，即将我们的处理器包装成相应的适配器（从而支持多种类型的处理器）  
                HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());  
  
                  // 304 Not Modified缓存支持  
                //此处省略具体代码  
  
                // 执行处理器相关的拦截器的预处理（HandlerInterceptor.preHandle）  
                //此处省略具体代码  
  
                // 步骤4、由适配器执行处理器（调用处理器相应功能处理方法）  
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());  
  
                // Do we need view name translation?  
                if (mv != null && !mv.hasView()) {  
                    mv.setViewName(getDefaultViewName(request));  
                }  
  
                // 执行处理器相关的拦截器的后处理（HandlerInterceptor.postHandle）  
                //此处省略具体代码  
            }  
            catch (ModelAndViewDefiningException ex) {  
                logger.debug("ModelAndViewDefiningException encountered", ex);  
                mv = ex.getModelAndView();  
            }  
            catch (Exception ex) {  
                Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);  
                mv = processHandlerException(processedRequest, response, handler, ex);  
                errorView = (mv != null);  
            }  
  
            //步骤5 步骤6、解析视图并进行视图的渲染  
//步骤5 由ViewResolver解析View（viewResolver.resolveViewName(viewName, locale)）  
//步骤6 视图在渲染时会把Model传入（view.render(mv.getModelInternal(), request, response);）  
            if (mv != null && !mv.wasCleared()) {  
                render(mv, processedRequest, response);  
                if (errorView) {  
                    WebUtils.clearErrorRequestAttributes(request);  
                }  
            }  
            else {  
                if (logger.isDebugEnabled()) {  
                    logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +  
                            "': assuming HandlerAdapter completed request handling");  
                }  
            }  
  
            // 执行处理器相关的拦截器的完成后处理（HandlerInterceptor.afterCompletion）  
            //此处省略具体代码  
  
  
        catch (Exception ex) {  
            // Trigger after-completion for thrown exception.  
            triggerAfterCompletion(mappedHandler, interceptorIndex, processedRequest, response, ex);  
            throw ex;  
        }  
        catch (Error err) {  
            ServletException ex = new NestedServletException("Handler processing failed", err);  
            // Trigger after-completion for thrown exception.  
            triggerAfterCompletion(mappedHandler, interceptorIndex, processedRequest, response, ex);  
            throw ex;  
        }  
  
        finally {  
            // Clean up any resources used by a multipart request.  
            if (processedRequest != request) {  
                cleanupMultipart(processedRequest);  
            }  
        }  
    }  
```


核心架构的具体流程步骤如下：

1、  首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；

2、  DispatcherServlet——><b>HandlerMapping</b>， HandlerMapping将会把请求映射为HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象，通过这种策略模式，很容易添加新的映射策略；

3、  DispatcherServlet——>HandlerAdapter，HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；

4、  HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个ModelAndView对象（包含模型数据、逻辑视图名）；

5、  ModelAndView的逻辑视图名——> ViewResolver， ViewResolver将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；

6、  View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此很容易支持其他视图技术；

7、返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户，到此一个流程结束。

 

此处我们只是讲了核心流程，没有考虑拦截器、本地解析、文件上传解析等，后边再细述。

# hello world

![upload successful](/images/pasted-30.png)


![upload successful](/images/pasted-60.png)

- DispatcherServlet：Spring提供的前端控制器，所有的请求都有经过它来统一分发。在DispatcherServlet将请求分发给Spring Controller之前，需要借助于Spring提供的HandlerMapping定位到具体的Controller。
- HandlerMapping：能够完成客户请求到Controller映射。
- Controller：需要为并发用户处理上述请求，因此实现Controller接口时，必须保证线程安全并且可重用。Controller将处理用户请求，这和Struts Action扮演的角色是一致的。一旦Controller处理完用户请求，则返回ModelAndView对象给DispatcherServlet前端控制器，ModelAndView中包含了模型（Model）和视图（View）。从宏观角度考虑，DispatcherServlet是整个Web应用的控制器；从微观考虑，Controller是单个Http请求处理过程中的控制器，而ModelAndView是Http请求过程中返回的模型（Model）和视图（View）。
- ViewResolver：Spring提供的视图解析器（ViewResolver）在Web应用中查找View对象，从而将相应结果渲染给客户。

 
# DispatcherServlet与HttpMessageConverter的关系
//TODO

 

http://sishuok.com/forum/blogPost/list/5160.html