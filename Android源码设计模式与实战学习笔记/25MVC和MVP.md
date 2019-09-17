# Android源码设计模式解析与实战

## 二十五、MVC

1. **起源与历史** 

   * 全称：Model-View-Controller

   * 1970由TrygveReenskaug在Smalltalk-80系统上首次提出、最开始不叫MVC，叫MVCE(Editor)，以Model为中心。

   * MVC是一种框架模式而非设计模式。

   * GOF把MVC看做3种设计模式：观察者模式、策略模式与组合模式的合体，核心在观察者模式，即基于发布/订阅者模型的框架。

   **框架模式和设计模式区别：**

   框架：是对代码的重用，面向于一系列相同行为代码的重用。是大智慧，用来对软件设计进行分工

   设计：是对设计的重用，面向于一系列相同结构代码的重用。是小技巧，对具体问题提出解决方案，以提高代码复用率，降低耦合度。

   架构：介于框架和设计之间。

   **开发领域3种重用**

   * 内部重用：同一应用中能公共使用的抽象块；
   * 代码重用：将通用模块组合成库或工具集，以便在多个应用和领域都能使用；
   * 应用框架的重用：专用领域提供通用的或现成的基础结构，以获得最高级别的重用性。

2. **优缺点**

   优点：易于理解，技术含量不高，易于维护和修改，耦合度不高，表现层与业务层分离

   缺点：无明确定义，完全理解不容易，给调试带来一定困难，更多文件，对于小规模项目会带来更大的工作量和复杂性。

3. **MVC在Android中的实现**

   如：View层一般采用XML文件进行界面描述，Model对应于本地的数据文件或网络获取的书具体，Controller由Activity承担。

   一般Activity中获取数据及界面元素，并将两者进行绑定，但其逻辑不能过于复杂，Activity响应时间是5秒，超过这个时间可能被回收。

   MVC更适合于规模比较大的项目，如Android的UI系统框架，我们使用的时候实际上是由framework给我们搭建好并提供给我们的。

   实例：

   //设置视图层

   ListView lv = new ListView(this);

   //获取数据

   String [] data = getResource().getStringArray(R.array.data);

   //数据绑定

   ArrayAdapter adapter = new ArrayAdapter(this,R.layout.activity_mvc_item,data);

   lv.setAdapter(adapter);

## 二十六、MVP应用架构模式

1. **MVP模式介绍**

   * MVC演化版本，全称Model View Presenter。

   * 分离显示层（UI界面）和逻辑层（数据），他们之间通过接口进行通信，降低耦合。

   * 理想化的MVP模式可以实现同一份逻辑代码搭配不同的显示界面，因为他们之间并不依赖于具体，而是依赖于抽象。
   * 并不是一个标准的模式，有多种实现方式。我们可以根据自己的需求和自己认为对的方式去修正MVP实现方式，它可以随着Presenter的复杂程度变化。只要保证我们是**通过Presenter将View和Model解耦和、降低类型复杂度、各个模块可以独立测试、独立变化**，这就是正确的方向。
   * 在Android中，大多数人可能会把Activity、Fragment作为View看待，可以看成是粗粒度的View，也可将他们看成Presenter。

2. **优缺点**

   优点：解除View与Model的耦合，良好的可扩展性、可测试性，保证系统的整洁性、灵活性。

   缺点：对于简单的应用稍显麻烦，各种各样的接口与概念，使得整个应用充斥着零散的接口。

3. **MVP模式的三个角色**

   * Presenter 交互中间人

     View和Model沟通的桥梁，它从Model层检索数据后返回给View层，使得View与Model没有耦合，将业务逻辑从View角色上抽离出来。

   * View 用户界面

     通常指Activity、Fragment或者某个View控件，它含有一个Presenter成员变量。

     通常View实现一个逻辑接口，将View上的操作通过转交给Presenter实现，Presenter调用View逻辑接口将结果返回给View元素。

   * Model 数据的存取

     对于结构化的App来说，Model角色主要提供数据的存取功能，可以说Model是封装了数据库DAO或者网络获取数据的角色或兼具二者。Presenter通过Model层存储、获取数据。

4. **与MVC、MVVM的区别**

   * 与MVC，MVP中的View不能直接访问Model。而MVC则可以
   * MVVM（VM指ViewModel），View与Model进行双向绑定（data-binding），两者之间有一方变化则会反应到另一方上。有点类似ListView与Adapter，Adapter就是ViewModel的角色。

5. **MVP的实现**

   略

6. **MVP与Activity、Fragment的生命周期**

   问题引入：Presenter执行一些耗时操作时，当Presenter持有了Activity的强引用，如果在请求结束之前Activity被销毁了，由于网络请求还没有返回，导致Presenter一直持有Activity对象，使得Activity对象无法被回收，此时就发生了内存泄漏。

   解决方案：通过若引用和Activity、Fragment的生命周期来解决。建立Presenter的抽象BasePresenter