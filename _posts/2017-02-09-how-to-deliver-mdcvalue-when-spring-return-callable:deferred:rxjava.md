---
layout: post
title:  "Spring异步返回时如何将线程值由请求线程传递到工作线程"
date:   2017-02-09 20:17:59
categories: 
- code 
tags:
- springmvc
- async
- rxjava
- callable
- defferedresult
---
自从servlet3.0后，可以利用**AsyncContext**来进行异步处理，释放请求线程。在spring中随后也推出了**WebAsyncTask**跟**DeferredResult**来简化程序的编写，这个在上文将rxjava引入springmvc中也有写明。

但一般同步时我们可能会写一堆***Filter，然后在这些过滤器可能会塞一些值到 [MDC](https://www.slf4j.org/api/org/slf4j/MDC.html) 中，像用来标识一次请求的TrackID等等，然后在业务逻辑将其取出出错时可以记录，但如果此时返回的是Callable等异步时，因为Spring的请求线程跟处理线程已经不是同一个，这种情况下去拿就肯定为空了，我们要怎么处理呢？

在这其实方案还是蛮多的，当然针对返回的类型不同，像Callable跟WebAsyncTask可以一样对待，而DeferredResult又是另一种方案，返回rx.java却又是另一种方案，在这除了DefferredResult不能透明掉外其它几种对业务开发人员来讲都可以透明掉，即对他们来讲还是正常返回即可，下面将具体分析。

#### Callable/WebAsyncTask

* 方案一，继承CallableProcessingInterceptorAdapter类重写beforeConcurrentHandling跟preProcess方法

<pre>public class MDCCallableProcessingInterceptor extends CallableProcessingInterceptorAdapter {    
   @Override    
   public &lt;T> void beforeConcurrentHandling(NativeWebRequest request, Callable&lt;T> task) throws Exception {          
   		request.setAttribute("mdcmap", MDC.getCopyOfContextMap(), -1);        
   		super.beforeConcurrentHandling(request, task);
   }    
   @Override    
   public &lt;T> void preProcess(NativeWebRequest request, Callable&lt;T> task) throws Exception {        
       Map map = (Map) request.getAttribute("mdcmap", -1);                
       MDC.setContextMap(map);        
       super.preProcess(request, task);    
   }
}</pre>
然后在继承WebMvcConfigurerAdapter类的configureAsyncSupport方法中添加自定义的这个MDCCallableProcessingInterceptor类
<pre>override def configureAsyncSupport(configurer: AsyncSupportConfigurer) = {
    configurer.registerCallableInterceptors(new MDCCallableProcessingInterceptor)
}</pre>

* 方案二，继承SimpleAsyncTaskExecutor类重写execute方法

<pre>public class MDCSimpleAsyncTaskExecutor extends SimpleAsyncTaskExecutor {
    @Override
    public void execute(Runnable task, long startTimeout) {
        super.execute(wrap(task, MDC.getCopyOfContextMap()), startTimeout);
    }
    private static Runnable wrap(final Runnable runnable, final Map&lt;String, String> context) {
        return () -> {
            Map previous = MDC.getCopyOfContextMap();
            if (context == null) {
                MDC.clear();
            } else {
                MDC.setContextMap(context);
            }
            try {
                runnable.run();
            } finally {
                if (previous == null) {
                    MDC.clear();
                } else {
                    MDC.setContextMap(previous);
                }
            }
        };
    }
}
</pre>
然后在继承WebMvcConfigurerAdapter类的configureAsyncSupport方法中设置自定义的这个SimpleAsyncTaskExecutor类
<pre>override def configureAsyncSupport(configurer: AsyncSupportConfigurer) = {
	configurer.setTaskExecutor(new MDCSimpleAsyncTaskExecutor)
}</pre>

#### rx.Observable
rx.Observable相对麻烦点，因为本身spring就不支持，在[上文](http://bobxwang.github.io/code/combilne-rxjava-in-spring-mvc/)的基础上，我们还需要做如下处理，分别继承Action0跟Func1&lt;Action0, Action0>两个类
<pre>public class MdcPropagatingAction0 implements Action0 {
    private final Action0 action0;
    private final Map&lt;String, String> context;
    public MdcPropagatingAction0(final Action0 action0) {
        this.action0 = action0;
        this.context = MDC.getCopyOfContextMap();
    }
    @Override
    public void call() {
        final Map&lt;String, String> originalMdc = MDC.getCopyOfContextMap();
        if (context != null) {
            MDC.setContextMap(context);
        }
        try {
            this.action0.call();
        } finally {
            if (originalMdc != null) {
                MDC.setContextMap(originalMdc);
            }
        }
    }
}
public class MdcPropagatingOnScheduleAction implements Func1&lt;Action0, Action0> {
    @Override
    public Action0 call(final Action0 action0) {
        return new MdcPropagatingAction0(action0);
    }
}</pre>

如果是springboot项目的话，在main方法中添加如下代码进行rxjava的钩子处理
<pre>RxJavaHooks.setOnScheduleAction(new MdcPropagatingOnScheduleAction)</pre>
但如果不是springboot的项目，就需要在监听器添加了

#### DeferredResult
本来以为DeferredResult可以像Callable一样处理，继承DeferredResultProcessingInterceptorAdapter这个类然后重写里面的beforeConcurrentHandling跟preProcess方法，但经过尝试，不行，后来想通了这其实也就是Callable跟DeferredResult的区别，那么我们要怎么做的，其实大致方案跟上面的差不多，就是在执行前对这个runable进行处理，把当前的MDC里的东西想办法传进去，最最简单的还是利用SimpleAsyncTaskExecutor这个类，让其来执行你的这个线程然后在里面设置其Result

#### 写在最后
正式环境中我们还需要处理超时时间，这样可以快速释放，来保证所有请求线程都可以在多久内返回或释放
<pre>override def configureAsyncSupport(configurer: AsyncSupportConfigurer) = {
	configurer.setDefaultTimeout(4000L)
}</pre>

