---
layout: post
title:  "改造SpringMVC使其支持返回rx.Observable的类型"
date:   2017-02-06 20:17:59
categories: 
- code 
tags:
- springmvc
- async
- rxjava

---
[rxjava](https://github.com/ReactiveX/RxJava)是[netflix](https://github.com/Netflix/)开源出来的一款响应式编程类库，是Reactive Extensions(Rx)在**jvm**平台上的一个实现，是一个集成了异步，基于可观察序列的事件驱动，如果是**net**平台，还支持**LING-style**查询操作。

自从servlet3.0后，可以利用**AsyncContext**来进行异步处理，释放请求线程。在spring中随后也推出了**WebAsyncTask**跟**DeferredResult**来简化程序的编写。

但怎么把rxjava跟springmvc结合起来呢，查看源码可以知道，当返回**DeferredResult**的时候关键处理的类是**DeferredResultMethodReturnValueHandler**，此类实现了**AsyncHandlerMethodReturnValueHandler**接口，那么解决方案就已经很清楚了，我们同样可以编写一个类继承此接口用来处理当返回类型是**rx.Observable**的处理器。
<pre>
class ObservableReturnValueHandler implements AsyncHandlerMethodReturnValueHandler {
  @Override
  public boolean isAsyncReturnValue(Object returnValue, MethodParameter returnType) {
    return returnValue != null && supportsReturnType(returnType);
  }
  @Override
  public boolean supportsReturnType(MethodParameter returnType) {
    return Observable.class.isAssignableFrom(returnType.getParameterType());
  }
  @Override
  public void handleReturnValue(Object returnValue, MethodParameter returnType, ModelAndViewContainer mavContainer, NativeWebRequest webRequest) 
  throws Exception {
    if (returnValue == null) {
      mavContainer.setRequestHandled(true);
      return;
    }
    final Observable<?> observable = Observable.class.cast(returnValue);
    WebAsyncUtils.getAsyncManager(webRequest)
            .startDeferredResultProcessing(new ObservableAdapter<>(observable), mavContainer);
  }
  public class ObservableAdapter<T> extends DeferredResult<T> {
    public ObservableAdapter(Observable<T> observable) {
      log.debug("observableAdapter begin to process");
      observable.subscribeOn(Schedulers.newThread())
              .subscribe(
                      t -> setResult(t),
                      throwable -> setErrorResult(throwable)
              );
    }
  }
}</pre>

> rxjava默认是单线程，所以我们在订阅的时候即subscribeOn的时候传入一个Schedulers.newThread来告诉它需要在一个新线程中运行接下来的代码。
> 这里有多种选项，具体请参考[Scheduler](http://reactivex.io/documentation/scheduler.html)

当然，我们最后还要把上面的类添加到spring体系中，通过重写**WebMvcConfigurerAdapter**类的**addReturnValueHandlers**方法
<pre>
@Configuration
class WebConfig extends WebMvcConfigurerAdapter {
  def addReturnValueHandlers(returnValueHandlers:util.List[HandlerMethodReturnValueHandler]) = {
    super.addReturnValueHandlers(returnValueHandlers)
    returnValueHandlers.add(new ObservableReturnValueHandler)
    }
}</pre>
当然，本文所述只是一个简单的实现，正式环境中还需要考虑超时的处理。