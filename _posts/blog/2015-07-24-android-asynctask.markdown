---
layout: post
keywords: blog
description: blog
title: "AsyncTask 小记"
categories: [Android]
tags: [Android]
---
{% include JB/setup %}

#AsyncTask 小记

**AsyncTask ** 主要作用情景为——执行耗时任务完成后更新UI，以免阻塞 **MainThread**——比如：网络请求（登陆，同步等）、数据库操作，复杂运算。可看作是 **Handler** + **Thread** 的某种简略方式（从源码中能明显看出），但功能上不可能完全取代。

###内部机制
1. 在内部会实例化一个静态的 `private static InternalHandler sHandler;`继承自**Handler**。同时自定义 **AsyncTaskResult** 对象，每次子线程需要与主线程传递消息时，就会根据 **AsyncTaskResult** 附带的消息类型不同，进行不同的操作。

2. 在内部的调度问题。虽然可以建立多个 **AsyncTask** ，但其内部 **Handler** 和 **ThreadPoolExecutor** 都是静态的，在进程范围内共享，并非相互独立。

在 **Android** 3.0 版本前后，**AsyncTask** 做了较大改变：

 * 3.0之前默认为并行操作：

		private static final int CORE_POOL_SIZE = 5;	//5个核心县城
		private static final int MAXIMUM_POOL_SIZE = 128;	//最大线程数128
		private static final int KEEP_ALIVE = 10;

		public static final Executor THREAD_POOL_EXECUTOR = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory); //建立线程池处理

 * 3.0之后默认为串行操作：
		
		public final AsyncTask<Params, Progress, Result> execute(Params... params) {
		return executeOnExecutor(sDefaultExecutor, params); // 默认执行的是sDefaultExecutor
    		}
		
		private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR; //sDefaultExecutor 又是 SerialExecutor 的静态实例
		
		private static class SerialExecutor implements Executor {
        	final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        	Runnable mActive;

        	public synchronized void execute(final Runnable r) {
            	mTasks.offer(new Runnable() {
                	public void run() {
                    	try {
                        	r.run(); // 可看出当一个任务执行完成后，再执行下一个，并不会并行。
                    	} finally {
                        	scheduleNext();
                    	}
                	}
            	});
            	if (mActive == null) {
                	scheduleNext();
            	}
        	}

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
           		}
        	}
		}

 修改为并发也较为简单，在执行 `public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
                                                                       Params... params)` 此方法时，**Executor** 采用 `AsyncTask.THREAD_POOL_EXECUTOR`作为默认线程池。

###注意点
 * 由于 Handler 需要和主线程交互，而 sHandler 又内置于 AsyncTask 中，所以 AsyncTask 的创建必须在主线程。
 * AsyncTaskResult 的 doInBackgroud 方法执行异步任务，运行于子线程，其他方法运行于主线程。
 * 不要手动去调用 AsyncTask 的 onPreExecute ， doInBackground ， publishProgress ... 方法，这些任务由 Android 系统自动调用。
 * 一个任务 AsyncTask 任务只能被执行一次。
 * 运行中可以随时调用 cancel 方法取消任务，如果被成功调用 isCancelled 会返回 true, 并且不会执行 onPostExecute 方法，取而代之的是 onCancelled 方法。如果，这个任务已经执行了，这调用cancel并不会把task真正的结束，而是继续执行，不过最后毁掉的方法改为 onCancelled 。
 * 若我们在onPreExcute 显示进度条 ，在 onPostExcute 关闭进度条，正常情况没有任何问题。一旦当 Activity 的 onConfiguration 被调用时，如果引起 Activity 重新启动的同时 AsyncTask 正在进行网络请求，异步的操作后会导致 Runtime 报错。解决思路可以借助第三方接口，将结果数据传递给它，它再传递到已注册的 Activity 处，这就u有效的隔离了 Activity 与 AsyncTask 的数据交互。具体实现可以使用 EventBus，otta，或者自己实现均可。
