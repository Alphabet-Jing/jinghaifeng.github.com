#Android 开发分析工具

分析性能的工具可分为两种类型：

* host
* on-device

##Host Tools

1. Systrace
	
	Systrace功能十分强大，可以展示系统某一时间点各种事件及持续事件。可通过该工具捕捉数据获得跟踪文件，并在浏览器中分析结果。
	
	可通过Android Studio 或 命令行打开：
	>$SDK_ROOT/platform-tools/systrace/systrace.py.
	
2. AllocationTracker

	用于追踪开启到结束之间的内存分配。对于检查某一时段的内存分配，做出对应优化有作用。
	
	可在ddms和Android Studio 打开。
	
3. TraceView

	方法分析器，可以运行在trace mode(追踪每一次方法的调用)或者sample mode(特定事件间隔内的方法调用)。Trace mode能够很好的帮助追踪间隔期间内，应用所执行的完整代码路径，并标明不同方法的消耗时间。
	
	