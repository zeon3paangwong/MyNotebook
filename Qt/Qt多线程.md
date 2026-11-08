QSignaleMapper():
connect(btn, &QPushButton::clicked,mapper, qOverload<>(&QSignalMapper::map));
connect(mapper, &QSignalMapper::mappedInt,this, &MyWidget::onButtonClicked);
qOverload<>(&QSignalMapper::map)确认QSignalMapper::map参数。

QThread子类化 vs moveToThread
QThread子类化后重写run方法，在主线程start后只执行一次。
moveToThread将对象的槽函数放到一个线程中去执行，开启run后进行事件循环。


线程池
将任务放到一个线程池里，线程池通过好几个线程，有任务进来就使用有空闲的线程，线程满了就把任务加到任务队列
需要继承QRunnable,重写run方法

并行概念
把一个大任务分成多个小任务，最后进行汇总。
并行库
QtConCurrent::run->QFuture;
需要继承QFutureWatcher重写run方法
并行计算
QtConCurrent::mappedReduced(列表，单任务处理函数，汇总函数*)

线程同步
当多个线程同时修改公共资源时，可能谁造成数据不一致，所以需要协同多个线程对公共资源的访问，此过程称为线程同步。
原理：变量值装入寄存器，计算寄存器的值，将寄存器中的值写回内存。

对象树模型
删除父节点，对应的子节点同时也会删除，自动进行内存的释放。




