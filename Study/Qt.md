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

RTTI:
Runtime Tyep Identification
元对象：
对于继承QObject类，同时使用Q_OBJECT宏对象的时候，在实例化一个对象的时候，Qt会生成一个对应的元对象QMetaObject。
`QMetaObject::newInstance()` 是 Qt 的**反射式对象创建**：不用 `new 类名`，而是**靠类名字符串或元对象，在运行时把对象造出来**。它和 `invokeMethod` 是同一套机制的“构造版本”——一个负责调函数，一个负责调构造函数。
1. 必须继承 `QObject`
2. 类声明里必须有 `Q_OBJECT`
3. **构造函数必须加 `Q_INVOKABLE`**（moc 才会把它写进元数据）
Q_INBOKABEL关键字：
作用是把一个普通的成员函数“注册”到 Qt 的元对象系统中，使其能够被**反射调用**（通过 `QMetaObject::invokeMethod` 按函数名字符串调用），也能被 QML、脚本引擎、属性系统等识别和使用。
`QMetaObject::invokeMethod`作用：
1、调用`Q_INBOKABEL` 修饰的函数
2、跨线程异步调用其他类的槽函数。
3、延迟调用该函数。

Sql注入：
黑客输入带"的字符，可能会使数据库执行奇怪的sql，Qt为了防止这种，进行了sql语句的预编译，需要执行固定模板的sql语句
