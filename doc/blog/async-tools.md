# AsyncTools解读

[toc]

## 简介

实现计算单元间的串行并行编排，通过异步回调解耦，支持任务组超时限制、条件触发和计算结果传递。

* 多任务串行、并行编排：

    1，具有依赖关系的任务组编排。比如`a-->(b-->c, d-->e-->f)-->g ）`，`a`触发`b`和`d`，`b`触发`c`,`d`触发`e`，`e`触发`f`，最后`c,f`共同触发`g`；

    2，前置任务可以设置强制依赖和非强制依赖。强制依赖任务必须全部正常返回才会触发后续任务，非强制依赖不对后续触发产生影响。比如`(optional(a, b),must(c, d))-->e`，`a,b`为强制依赖必须全部正常完成才能触发`e`，非强制依赖`c,d`不对`e`的触发产生影响；

    3，任务组间线程隔离。

* 任务回调：每个任务都可以有回调，用于实现监控、日志、设置默认值、异常处理等功能；
* 短路特性：当设置任一前置任务执行成功即触发后续任务时，如果某一前置任务成功执行，即触发后续任务，其余前置任务将被跳过不再执行。例如`any(a, b-->c, d-->e-->f)-->g`中任务`a`成功执行触发`g`，其余前置任务支路` b-->c, d-->e-->f`将被短路，跳过不执行；
* 可以为任务组设置超时限制，超时后还未被执行的任务将不会被处理；
* 上游任务结果依赖：同一个任务组内下游任务可以获取任一上游任务计算结果作为自己的输入;
* 无锁编程、线程资源复用。

## 组件

* `Worker`：最基本计算单元，无状态节点，执行计算任务，支持设置默认计算结果。
* `Callback`：为`Worker`执行前置准备和后置回调操作，无状态节点，通过异步回调避免任务调用方阻塞。
* `WorkerWrapper`：对每个`worker`及`callback`进行一对一包装。有状态节点，保存包装节点运行状态、前置依赖节点列表、后续调用计算节点列表、计算节点输入参数和输出返回值。同时决定当前任务节点是否被触发，并发起对后续计算任务节点的调用。
* `Async`：任务执行入口，传入任务组起始节点，并控制任务组超时。

## `Worker`

* `Worker`接受输入参数，同时可以通过共享容器获取到其它计算节点的返回值，最后将自己的计算结果放入共享容器中

    ```java
    @FunctionalInterface
    public interface IWorker<T, V> {
    	// 计算任务
        V action(T object, Map<String, WorkerWrapper> allWrappers);
    	// 设置默认值，用于计算节点未正常结束时的返回值
        default V defaultValue() {
            return null;
        }
    }
    
    ```

    

## `Callback`

* `Callback`为`Worker`执行前置准备和后置回调操作，回调方法中将根据执行成功与否、输入参数与执行结果执行具体的回调逻辑。

    ```java
    @FunctionalInterface
    public interface IWorker<T, V> {
    	// 计算任务
        V action(T object, Map<String, WorkerWrapper> allWrappers);
    	// 设置默认值，用于计算节点未正常结束时的返回值
        default V defaultValue() {
            return null;
        }
    }
    
    ```

    

## `WorkerWrapper`

### 节点状态

* 对每个`Worker`及`Callback`进行一对一包装，由于`Worker`和`Callback`都是无状态类型，所以需要包装节点保存节点状态。主要处理前后节点间联系，以及当前节点与后续节点的触发执行。

    ```java
    public class WorkerWrapper<T, V> {
        // 唯一标识
        private String id;
        // 任务参数
        private T param;
        private IWorker<T, V> worker;
        private ICallback<T, V> callback;
        // 后续任务
        private List<WorkerWrapper<?, ?>> nextWrappers;
        // 前置任务DependWrapper=<WorkerWrapper, isMust>，
        // 有强制依赖isMust=true，和非强制依赖isMust=false之分
        private List<DependWrapper> dependWrappers;
        // 任务执行状态
        // 0-init, 1-finish, 2-error, 3-working
        private AtomicInteger state = new AtomicInteger(0);
        // 存放所有节点计算结果
        private Map<String, WorkerWrapper> forParamUseWrappers;
    }
    ```

    

### 当前任务触发

 ==当前任务组未超时+当前任务未被执行+当前任务支线有执行的必要+当前任务的所有强制依赖已全部完成==时，当前任务将会触发执行。

##### 任务组超时控制

* 超时限制：可以为整个任务组设置时间限制，从源头任务开始计时，随着任务的执行运行时间逐渐增加，超时控制分为任务内部，以及任务外部。

* 在任务执行内部，每次执行当前任务前需要检查是否超时，如果超时将跳过当前任务，继续执行后续处理。

    ```java
    /**
     * 执行当前任务
     * executorService：执行任务的线程池
     * fromWrapper：代表当前work是由上游哪个wrapper发起
     * remainTime: 当前任务组剩余执行时间
     */
    private void work(ExecutorService executorService, WorkerWrapper fromWrapper, long remainTime, Map<String, WorkerWrapper> forParamUseWrappers) {
        long now = SystemClock.now();
        // 执行当前任务前超时检查：当前任务组已超时，跳过执行，快速失败，进行后续节点调用
        if (remainTime <= 0) {
            // 设置默认值并执行回调
            fastFail(INIT, null);
            beginNext(executorService, now, remainTime);
            return;
        }
        dosomething()
    }
    
    ```

    

* 同时执行完当前任务处理，触发后续任务处理时，在后续任务提交执行后，开启超时等待，超时后将不再等待后续任务返回。

    ```java
    private void beginNext(ExecutorService executorService, long now, long remainTime) {
        // 已经花费的时间
        long costTime = SystemClock.now() - now;
        // 后续任务提交线程池异步执行，由于后续任务存在多条支路，需要在外部统一控制超时
        CompletableFuture[] futures = new CompletableFuture[nextWrappers.size()];
        for (int i = 0; i < nextWrappers.size(); i++) {
            int finalI = i;
            futures[i] = CompletableFuture.runAsync(() -> nextWrappers.get(finalI)
                    .work(executorService, WorkerWrapper.this, remainTime - costTime, forParamUseWrappers),
                    executorService);
        }
        try {
            // 任务外部控制器：超时等待所有任务在指定时限内返回，超时后将不再等待，
            // 注意这里未强制中断超时任务
            CompletableFuture.allOf(futures).get(remainTime - costTime, TimeUnit.MILLISECONDS);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    ```

    

##### 避免重复执行

* 执行状态检查：由上述的超时控制可知，任务执行内部只在执行前检查，无法在任务进行中实时监测超时状态。同时任务外部控制器的超时等待超时后，将不再继续等待，直接继续进行下一阶段任务处理。==正在执行的超时任务未被强制终止，仍然有可能正常返回==，再次触发下一阶段任务处理。所以下一阶段任务将可能被多次触发，需要额外处理，防止任务被重复触发。

    ```java
    private void work(ExecutorService executorService, WorkerWrapper fromWrapper, long remainTime, Map<String, WorkerWrapper> forParamUseWrappers) {
        dosomething()
        // 如果当前节点已经执行过，则不再执行
        if (getState() == FINISH || getState() == ERROR) {
            beginNext(executorService, now, remainTime);
            return;
        }
        dosomething()
    }
    
    ```

    

##### 计算任务短路

* 执行必要性判断：对于存在多个可选前置依赖的任务，如果它已被某个依赖触发，则其他前置依赖支线任务没必要再执行。如任务组`optional(a->b->c,d->e)->f`中任务`f`可以被支路`a->b->c`触发，也可以被`d->e`触发。当任务`f`被`e`触发后，任务支线`a->b->c`没继续执行必要。

    ```java
    private void work(ExecutorService executorService, WorkerWrapper fromWrapper, long remainTime, Map<String, WorkerWrapper> forParamUseWrappers) {
       dosomething()
        // 如果当前任务的next链上有已经出结果或有已经开始执行的任务，当前任务不用继续执行
        // 短路特性，后续任务已被其它节点触发，不再执行当前支路任务
        if (!checkNextWrapperResult()) {
            fastFail(INIT, new SkippedException());
            beginNext(executorService, now, remainTime);
            return;
        }
        dosomething()
    }
    ```

    

* `checkNextWrapperResult`方法将从当前节点开始以`DFS` 方式遍历后续任务，如果出现已经出结果或有已经开始执行的任务，证明当前任务不用继续执行

    ```java
    
    /**
     * 判断自己下游链路上，是否存在已经出结果的或已经开始执行的节点
     * 上述判断只针对后续任务数=1等情形成立（如a->b->c）
     */
    private boolean checkNextWrapperResult() {
        // 如果当前任务就是最后一个，或者后面有并行的多个后置任务，表示当前节点必须被执行
        if (nextWrappers == null || nextWrappers.size() != 1) {
            return getState() == INIT;
        }
        // 查看当前节点是否未被执行
        WorkerWrapper nextWrapper = nextWrappers.get(0);
        boolean state = nextWrapper.getState() == INIT;
        // DFS方式继续校验自后续节点执行状态，保证后续链路任务都未被执行
        return state && nextWrapper.checkNextWrapperResult();
    }
    ```

    

##### 条件触发

* 强制依赖检查：只有当前结点是任务组的最初始节点，或者全部强制依赖已正常完成，才能证明当前节点以满足执行条件，才会开始执行。

    ```java
    private void work(ExecutorService executorService, WorkerWrapper fromWrapper, long remainTime, Map<String, WorkerWrapper> forParamUseWrappers) {
        dosomething()
        // 如果没有任何依赖，说明自己就是起始任务
        if (dependWrappers == null || dependWrappers.size() == 0) {
            // 执行当前任务
            fire();
            beginNext(executorService, now, remainTime);
            return;
        }
        // 只有一个依赖，则说明当前任务的全部依赖已运行完毕，需要进一步判断依赖是否正常结束
        if (dependWrappers.size() == 1) {
            doDependsOneJob(fromWrapper);
            beginNext(executorService, now, remainTime);
        } else {
            // 有多个依赖时，当前节点被触发，需要检查全部必须依赖是否已正常完成
            doDependsJobs(executorService, dependWrappers, fromWrapper, now, remainTime);
        }
    }
    ```

    

* 当只有一个强制依赖，只需该依赖任务正常完成，即可执行当前任务；

    ```java
    /**
     * 如果只有一个强制依赖，只需该依赖正常完成
     */
    private void doDependsOneJob(WorkerWrapper dependWrapper) {
        // 唯一前置依赖节点处于超时或者异常态，则当前任务支线异常，跳过当前任务
        if (ResultState.TIMEOUT == dependWrapper.getWorkResult().getResultState()) {
            workResult = defaultResult();
            fastFail(INIT, null);
        } else if (ResultState.EXCEPTION == dependWrapper.getWorkResult().getResultState()) {
            workResult = defaultExResult(dependWrapper.getWorkResult().getEx());
            fastFail(INIT, null);
        } else {
            // 唯一前置依赖节点正常执行，执行当前节点
            fire();
        }
    }
    
    ```

    

* 如果存在多个依赖，需要保证触发当前任务的前置任务必须是强制依赖，全部强制依赖才有可能已完成，之后再进行进一步检测强制依赖是否都正常完成。

    同时需要对该方法加锁串行化访问，防止在存在多个依赖时，可能存在多个依赖同时触发当前任务执行判断，所以需要加锁串行化访问，防止并发读写，同时==当前任务只等待其最后一个强制依赖完成==，所以其他依赖触发`doDependsJobs`是无意义的，也没有并行化的意义。
    
    ```java
    /**
     * 如果存在多个强制依赖，需要保证触发当前任务的前置任务必须是强制依赖，之后再检查依赖是否都正常完成
     */
    private synchronized void doDependsJobs(ExecutorService executorService, List<DependWrapper> dependWrappers,WorkerWrapper fromWrapper, long now, long remainTime) {
        // 加锁序列化访问，用于多个依赖同时触发下一节点时的情形
        // 如果当前节点已被其他依赖触发，则无需重复执行
        if (getState() != INIT) {
            return;
        }
        // 只有触发触发当前任务的前置节点fromWrapper是当前节点的必须依赖时，才有可能全部强制依赖已完成
        boolean nowDependIsMust = false;
        // 必须完成的上游wrapper集合
        Set<DependWrapper> mustWrapper = new HashSet<>();
        for (DependWrapper dependWrapper : dependWrappers) {
            if (dependWrapper.isMust()) {
                mustWrapper.add(dependWrapper);
            }
            if (dependWrapper.getDependWrapper().equals(fromWrapper)) {
                nowDependIsMust = dependWrapper.isMust();
            }
        }
    
        // 前置节点fromWrapper不是当前节点的必须依赖，且存在强制依赖
        // 则可能存在未完成强制依赖，直接返回
        if (!nowDependIsMust) {
            return;
        }
    
        // 如果前置fromWrapper是必须的，则有可能触发当前节点，具体取决于全部前置依赖是否完成
        boolean existNoFinish = false;
        boolean hasError = false;
    
        // 判断必须要执行的依赖任务的执行结果，如果有任何一个超时或者异常，证明当前任务不会被触发，直接跳过当前任务
        for (DependWrapper dependWrapper : mustWrapper) {
            WorkerWrapper workerWrapper = dependWrapper.getDependWrapper();
            WorkResult tempWorkResult = workerWrapper.getWorkResult();
            // 未完成
            if (workerWrapper.getState() == INIT || workerWrapper.getState() == WORKING) {
                existNoFinish = true;
                break;
            }
            // 超时
            if (ResultState.TIMEOUT == tempWorkResult.getResultState()) {
                workResult = defaultResult();
                hasError = true;
                break;
            }
            // 异常
            if (ResultState.EXCEPTION == tempWorkResult.getResultState()) {
                workResult = defaultExResult(workerWrapper.getWorkResult().getEx());
                hasError = true;
                break;
            }
    
        }
    
        // 如果有任何一个超时或者异常，证明当前任务不会被触发，直接跳过当前任务
        if (hasError) {
            fastFail(INIT, null);
            beginNext(executorService, now, remainTime);
            return;
        }
    
        // 强制依赖全部正常完成，不存在未执行或正在执行的节点，则当前节点被触发
        if (!existNoFinish) {
            fire();
            beginNext(executorService, now, remainTime);
            return;
        }
    }
    ```
    
    

### 当前任务执行

* 任务的执行分为三部分：==前置准备+任务调用+后置回调==。在执行的最开始以及回调前，通过`CAS`将节点的状态设置为`WORKING`以及`FINISH`。后续如果再被触发，将通过状态判断出已执行，提前返回，防止被重复执行。通过`CAS`方式避免加锁，避免不需要的线程竞争。

* ==为什么需要`CAS`并发写入控制==：上面的任务触发部分已经保证每个节点只会被执行一次，理论上不需要通过`CAS`并发控制，但是有两种特殊情况将导致并发修改节点状态：

    1，某个任务导致任务组超时，此时外部超时控制机制将执行所有节点的快速失败方法，将修改节点状态；同时该超时任务并未结束，将可能导致并发写入节点状态。

    2，某个任务导致任务组超时，此时外部超时控制机制将执行所有节点的快速失败方法，将修改节点状态；同时该超时任务成功结束，触发后续节点的执行，后续节点的`work`方法将发现任务组已超时，执行后续节点的快速失败方法，后续节点失败方法可能并发执行，导致并发写入节点状态。
    
    ```java
    private WorkResult<V> workerDoJob() {
        try {
            // 将当前状态通过CAS方式从INIT改为WORKING
            if (!compareAndSetState(INIT, WORKING)) {
                return workResult;
            }
            // 前置任务
            callback.begin();
            V resultValue = worker.action(param, forParamUseWrappers);
            // 将当前状态通过CAS方式从WORKING改为FINISH
            if (!compareAndSetState(WORKING, FINISH)) {
                return workResult;
            }
            // 任务回调
            callback.result(true, param, workResult);
            return workResult;
        } catch (Exception e) {
        }
    }
    ```
    
    

### 后续任务触发

* 当前任务执行完成后需要触发后续任务的执行，根据后续任务的数量可写分为：只有一个后续任务，则复用当前任务线程；如果存在多个后续任务，则将他们交由线程池异步执行，当前任务线程将执行外部超时控制逻辑，等待至超时时限后返回。

    ```java
    private void beginNext(ExecutorService executorService, long now, long remainTime) {
        // 如果只有一个后置任务，直接使用当前线程执行，再后续任务中执行超时控制
        if (nextWrappers.size() == 1) {
            nextWrappers.get(0).work(executorService, WorkerWrapper.this, remainTime - costTime, forParamUseWrappers);
            return;
        }
        // 如果存在多个后续任务，线程池异步执行多个任务，同时当前线程在任务外部统一控制超时
        CompletableFuture[] futures = new CompletableFuture[nextWrappers.size()];
        for (int i = 0; i < nextWrappers.size(); i++) {
            int finalI = i;
            futures[i] = CompletableFuture.runAsync(() -> nextWrappers.get(finalI)
                    .work(executorService, WorkerWrapper.this, remainTime - costTime, forParamUseWrappers),
                    executorService);
        }
        try {
            // 等待全部支路任务在时限务返回
            CompletableFuture.allOf(futures).get(remainTime - costTime, TimeUnit.MILLISECONDS);
        } catch (Exception e) {
        }
    }
    ```

    

* 需要注意的是，不是只有当前任务正常执行后才需要触发后续任务的执行。前置依赖任务超时、异常、当前任务执行前已超时、当前任务被短路等情形下，当前任务将不会执行，在为当前任务执行快速失败后，仍将执行后续任务的触发，此时后续任务同样不会真正被执行，只会执行他们的快速失败方法，这主要是为当前任务和后续任务设置默认值，并以默认值执行回调，保证任务组执行的完整。

    这里同样需要`CAS`并发写入控制，理由同`workerDoJob`。
    
    ```java
    private boolean fastFail(int expect, Exception e) {
        // 试图将它从expect状态,改成Error
        // 同样需要`CAS`并发写入控制，理由同上。
        if (!compareAndSetState(expect, ERROR)) {
            return false;
        }
        // 计算结果仍未默认值空值
        if (checkIsNullResult()) {
            if (e == null) {
                workResult = defaultResult();
            } else {
                workResult = defaultExResult(e);
            }
        }
        callback.result(false, param, workResult);
        return true;
    }
    ```
    
    

## `Async`

* 任务组启动的入口为`Async`的`beginWork`方法，将源头任务交由线程池执行，开启整个任务组的执行。`beginWork`方法与`beginNext`方法较为类似，但最大不同点在于启动任务组的线程会作为超时控制线程，当设置的超时时间到达后，为所有未执行或者正在执行的`WorkerWrapper`执行`fastFail`快速失败，设置默认值及进行回调，同时未被执行任务将不会被执行，正在执行的任务也不会被强制中断，效果类似线程池的`stop`方法。

    ```java
    public static boolean beginWork(long timeout, ExecutorService executorService, List<WorkerWrapper> workerWrappers) throws ExecutionException, InterruptedException {
        // 将任务组起始任务交由线程池异步执行
        CompletableFuture[] futures = new CompletableFuture[workerWrappers.size()];
        for (int i = 0; i < workerWrappers.size(); i++) {
            WorkerWrapper wrapper = workerWrappers.get(i);
            futures[i] = CompletableFuture.runAsync(() -> wrapper.work(executorService, timeout, forParamUseWrappers), executorService);
        }
        try {
            // 启动任务组的线程会作为超时控制线程，等待超时时间到达
            CompletableFuture.allOf(futures).get(timeout, TimeUnit.MILLISECONDS);
            return true;
        } catch (TimeoutException e) {
            Set<WorkerWrapper> set = new HashSet<>();
            totalWorkers(workerWrappers, set);
            // 为所有未执行或者正在执行的`WorkerWrapper`执行`fastFail`快速失败
            // 设置默认值及进行回调
            for (WorkerWrapper wrapper : set) {
                wrapper.stopNow();
            }
            return false;
        }
    }
    ```

    

## 思考

* 计算任务短路部分，判断只针对后续任务数等于1的情形成立（如`a->b->c`），如果后续任务数大于1，则该任务将被执行。可能作者认为一个任务被多个后续任务依赖，具有较高的重要性，所以只要该任务未被执行就默认将其执行，忽略短路特性。

    我认为即使当前任务被多个后续任务依赖，多个后续任务是存在都被触发的情形，所以是可以被短路跳过的。可以修改检查是否需要短路的方法，`DFS`方式遍历以当前节点为起点的依赖图，如果出现未被触发的任务，证明当前任务由执行的必要，否则当前任务可以跳过。

    ```java
    private boolean checkNextWrapperResultDFS() {
        // 当前节点状态
        if (getState() == INIT) {
            return true;
        }
        // 无后续节点且当前已被执行
        if (nextWrappers == null) {
            return false;
        }
        // 查看是否存在未被执行后续依赖图节点
        for (WorkerWrapper nextWrapper : nextWrappers) {
            if (nextWrapper.checkNextWrapperResultDFS()) {
                return true;
            }
        }
        return false;
    }
    ```

    

* 为什么任务组超时后只是跳过尚未执行任务，正在执行的任务仍然可以继续执行。我们可以通过在外部发起`interrupt`调用，线程内部使用`isInterrupted`检查中断状态，实现强制中断任务。但是长时间的任务一般是请求接口，强制中断的意义不大，同时`isInterrupted`额外增加复杂度。

* 线程池复用线程导致的`ThradLocal`数据污染问题。使用`TransmittableThreadLocal`，当从父线程提交任务到线程池时，`TransmittableThreadLocal`会捕获在父线程中设置的变量副本，并确保这些副本在子线程中可用，即使这些子线程可能是由线程池中的现有线程执行的。

* 状态并发读取问题：但计算任务完成后先设置状态为完成完成，再设置返回值。先设置标志位后设置结果方式将带来并发读写问题。

    ```java
    private WorkResult<V> workerDoJob() {
        // 执行耗时操作
        V resultValue = worker.action(param, forParamUseWrappers);
        // 标志位：当前节点已完成
        if (!compareAndSetState(WORKING, FINISH)) {
            return workResult;
        }
        // 写入结果
        workResult.setResultState(ResultState.SUCCESS)
        workResult.setResult(resultValue);
        // 任务回调
        callback.result(true, param, workResult)
    }
    ```

    例如`must(a,b)->c`任务中，`a,b`必须同时完成才会触发`c`，当`a`完成并进入`c`的触发判断时，`b`刚执行完`worker.action`并设置节点状态为已完成，此时`c`读取`b`状态发现其已完成，则`c`满足触发条件，进入`action`方法，并读取`b`的结果，此时`b`尚未将结果写入，导致`c`读取错误值。

    可以调整顺序，先设置结果，再执行回调，最后设置状态。修改后上述情形下，`c`将不满足触发条件，`c`最终由`b`触发。

    ```java
    private WorkResult<V> workerDoJob() {
        // 执行耗时操作
        V resultValue = worker.action(param, forParamUseWrappers);
        // 写入结果
        workResult.setResultState(ResultState.SUCCESS);
        workResult.setResult(resultValue);
        // 任务回调
        callback.result(true, param, workResult)
        // 标志位：当前节点已完成
        if (!compareAndSetState(WORKING, FINISH)) {
            return workResult;
        }
    }
    ```

    
