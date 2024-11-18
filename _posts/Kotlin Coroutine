Kotlin协程原理
kotlin中理解协程：
理解为函数多了resume和suspend状态，遇到suspend函数不管三七二十一，先挂起，然后通过调度器调度到目标线程，然后resume继续执行！
Java中理解协程：
通过回调接口+状态机的方式实现。
状态机意味着继续执行代码，记录label，也就是记录上次执行到的位置，下次调用的时候通过label回到对应位置继续执行。
匿名类是为了重新执行一次本函数，回到状态机的switch-case中。

前言：
类比thread和操作系统关系，类比到协程和语言中调度器
something i wanna say：⬇️
协程的本质是为了简化回调，让异步回调便于阅读，仅此而已，无他。
一个任务或者操作本质是交给对应平台的事件循环或者另外的线程去做的，其他就是一些优化
对于suspend修饰的函数他们多了两个操作分别是：suspend和resume。
协程在常规函数的基础上添加了两项操作，用于处理长时间运行的任务。在 invoke（或 call）和 return 之外，协程添加了 suspend 和 resume，一共四个操作。
协程中所有操作都是委托事件循环或者线程（包括主线程）
以前对协程还是有些先入为主的偏见。

关于整个Kotlin协程框架核心的点是：（我认为问题更能让人深刻理解某一个事物，so...好吧也有陈述句）
●挂起函数的挂起是如何实现的？
●挂起函数里面又执行一个挂起函数会如何执行呢？
●挂起函数的函数内容是什么时候执行的？
●协程的调度是如何实现的？！
●协程的拦截器的原理又是什么呢？
●实际上协程还是把操作或者任务委托给了其他人处理，比如事件循环，线程
●kotlin协程是无栈协程，go是有栈协程，有栈无栈区别在于上下文保存，koltin通过contiuation实现上下文传输，也就是一个对象，持有状态和、要调用的方法。
●协程作用域，为了让协程便于使用。
●协程异常的传递，全局获取异常，如何传递的呢？
●协程是如何取消的呢？
●关于Flow
回调：
what：
回调（Callback）是编程中的一种常见模式，指的是将一个函数作为参数传递给另一个函数，以便在特定事件发生或某个操作完成时调用这个函数。回调函数通常用于处理异步操作、事件处理或某些特定条件下的逻辑。
人话来说就是给一个对象作为函数的参数，在这个函数某些特定的时机执行这个对象里面的函数。
why：
这里强调场景：
●在许多情况下，程序需要执行耗时的操作（如网络请求、文件读取等），而这些操作可能会导致程序阻塞。回调函数允许程序在等待这些操作完成时继续执行其他代码。
●在事件驱动编程中，回调函数用于响应用户的操作（如点击按钮、输入文本等）。当事件发生时，系统会调用相应的回调函数来处理该事件。
●回调函数使得代码更加灵活和可重用。通过将不同的回调函数传递给同一个函数，可以实现不同的行为，而无需修改函数本身的实现。
●在复杂的异步操作中，回调函数可以帮助管理控制流，确保在某个操作完成后再执行后续操作。这种方式使得代码逻辑更加清晰。
how：
异步回调：
interface Callback{
    fun error()
    fun log()
}
fun fetchData(callback: Callback) {
    if(...){
         handler.post{
                callback.error("Error:", error);
        }
    }else{
        handler.post{
                callback.log(result); // 输出: Fetched Data
        }
    }
}
//这个是android中handler事件循环的情况
fetchData(Callback());

同步回调：
function fetchData(callback) {
    if(...){
        callback.error("Error:", error);
    }else{
        callback.log(result); // 输出: Fetched Data
    }
}
//这个是android中handler事件循环的情况
fetchData(Callback());


挂起函数：
what：
使用suspend关键字修饰
why：
google官方通过设计，给函数添加了两个操作：suspend和resume
how：
用suspend修饰普通函数即可
挂起函数可以通过反射和协程调度
那么挂起是怎么实现的呢？
挂起函数的字节码反编程Java，会发现，函数传递了continuation的回调函数，内部维护label和resume函数，执行函数的时候，顺序执行，可以这样理解：
Object change(Continuation con){
    ...;
    Thread{
        con.resume();
        return INSTANCE;
    }.start()
    ...;
    return SUSPEND;
}
public class ContinuationImpl implements Continuation<Object> {

    private int label = 0;
    private final Continuation<Unit> completion;

    public ContinuationImpl(Continuation<Unit> completion) {
        this.completion = completion;
    }

    @Override
    public CoroutineContext getContext() {
        return EmptyCoroutineContext.INSTANCE;
    }

    @Override
    public void resumeWith(@NotNull Object o) {
        try {
            Object result = o;
            switch (label) {
                case 0: {
                    LogKt.log(1);
                    result = SuspendFunctionsKt.returnSuspended( this);
                    label++;
                    if (isSuspended(result)) return;
                }
                case 1: {
                    LogKt.log(result);
                    LogKt.log(2);
                    result = DelayKt.delay(1000, this);
                    label++;
                    if (isSuspended(result)) return;
                }
                case 2: {
                    LogKt.log(3);
                    result = SuspendFunctionsKt.returnImmediately( this);
                    label++;
                    if (isSuspended(result)) return;
                }
                case 3:{
                    LogKt.log(result);
                    LogKt.log(4);
                }
            }
            completion.resumeWith(Unit.INSTANCE);
        } catch (Exception e) {
            completion.resumeWith(e);
        }
    }

    private boolean isSuspended(Object result) {
        return result == IntrinsicsKt.getCOROUTINE_SUSPENDED();
    }
}

suspend：
suspend关键字：表示接受协程调度的函数
协程作用域和suspend函数可以调用suspend函数
实现挂起函数的关键在于能够暂停函数的执行并在稍后恢复，同时保持函数的状态。
究竟如何挂起？


CPS：
一种编程风格
cps转换：
协程：
what：
why：
how：
协程的启动：
协程的启动需要三个条件：
上下文：
启动模式：
协程体：即一个高阶函数，内部是需要执行的代码
协程的启动模式：
模式	功能
DEFAULT	立即执行协程体
ATOMIC	立即执行协程体，但在开始运行之前无法取消
UNDISPATCHED	立即在当前线程执行协程体，直到第一个 suspend 调用
LAZY	只有在需要的情况下运行

四个启动模式我们最常用是DEFAULT和LAZY：
DEFAULT：
 是饿汉式启动，launch 调用后，会立即进入待调度状态，一旦调度器 OK 就可以开始执行。
LAZY：
是懒汉式启动，launch 后并不会有任何调度行为，协程体也自然不会进入执行状态，直到我们需要它执行的时候。 launch 调用后会返回一个 Job 实例，对于这种情况，我们可以：
●调用 job.start()，主动触发协程的调度执行
●调用 job.join()，隐式的触发协程的调度执行
ATOMIC：
ATOMIC 只有涉及 cancel 的时候才有意义，在这里我们就简单认为 cancel 后协程会被取消掉，也就是不再执行了。那么调用 cancel 的时机不同，结果也是有差异的，例如协程调度之前、开始调度但尚未执行、已经开始执行、执行完毕等等。
执行下面代码：
log("1")
val job = GlobalScope.launch(start = CoroutineStart.DEFAULT) {
    log("2")
}
job.cancel()
log("3")

此时的启动模式是DEFAULT，所以2是有可能输出，有可能不输出。
原因：在第一次调度该协程时如果 cancel 就已经调用，那么协程就会直接被 cancel 而不会有任何调用，当然也有可能协程开始时尚未被 cancel，那么它就可以正常启动了。所以前面的例子如果改用 DEFAULT 模式，那么 2 有可能会输出，也可能不会。
注意的是，cancel 调用一定会将该 job 的状态置为 cancelling，只不过ATOMIC 模式的协程在启动时无视了这一状态。
也就是说ATOMIC状态一定会启动
UNDISPATCHED：
协程在这种模式下会直接开始在当前线程下执行，直到第一个挂起点，这听起来有点儿像前面的 ATOMIC，不同之处在于 UNDISPATCHED 不经过任何调度器即开始执行协程体。当然遇到挂起点之后的执行就取决于挂起点本身的逻辑以及上下文当中的调度器了。
log(1)
val job = GlobalScope.launch(start = CoroutineStart.UNDISPATCHED) {
    log(2)
    delay(100)
    log(3)
}
log(4)
job.join()
log(5)


注意，挂起点之后，由调度器执行，挂起之前由当前代码直接执行：

22:00:31:693 [main] 1
22:00:31:782 [main] 2
22:00:31:800 [main] 4
22:00:31:914 [DefaultDispatcher-worker-1 @coroutine#1] 3

注意：2直接在main中打印，并没有其他线程。
在挂起点之前是直接在协程调用所在线程直接执行，挂起点之后由协程调度器接管
协程的调度：
协程上下文：
协程上下文CoroutineContext 作为一个集合，它的元素就是源码中看到的 Element，每一个 Element 都有一个 key，因此它可以作为元素出现，同时它也是 CoroutineContext 的子接口，因此也可以作为集合出现。
上下文可以通过对应Key的方式拿到，这类似于使用Thread.currentThread()。
如果有多个上下文需要添加，直接用 + 就可以了：	
GlobalScope.launch(Dispatchers.Main + CoroutineName("Hello")) {
    ...
}

协程拦截器：
public interface ContinuationInterceptor : CoroutineContext.Element {
    companion object Key : CoroutineContext.Key<ContinuationInterceptor>
    
    public fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T>
    ...
}


拦截器也是一个上下文的实现方向，拦截器可以左右你的协程的执行，同时为了保证它的功能的正确性，协程上下文集合永远将它放在最后面。
它拦截协程的方法也很简单，因为协程的本质就是回调 + 调度，而这个回调就是被拦截的 Continuation 了。用过 OkHttp 的小伙伴一下就兴奋了，拦截器我常用的啊，OkHttp 用拦截器做缓存，打日志，还可以模拟请求，协程拦截器也是一样的道理。调度器就是基于拦截器实现的，换句话说调度器就是拦截器的一种。

（Soon......）
协程的取消：

协程作用域：
顶层作用域
之下的作用域
协程的本质：CPS+状态机
协程异常处理：
两种作用域，
最佳实践：
Flow：


