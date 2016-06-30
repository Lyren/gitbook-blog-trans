# The Clean Architecture
## Getting Started

众所周知，开发一款高质量的软件是一个艰难而复杂的过程。这不仅仅意味着软件要满足产品需求，更要求软件架构的健壮性，可维护行和可测试性，并且要能够灵活的应对需求的增加和变更。这就是 **The Clean Architecture** 出现的原因，它可能是一个很好的方法去开发任何一款软件应用。  
**Clean Architecture** 代表了类似以下生产系统的一组实践：  

![clean architecture](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture1.png)  

* 独立于框架  
* 独立于UI  
* 独立于数据仓库  
* 独立于任何外部组件  

如图所示，相比于4个圆圈你更应该关注**依赖规则**。源码的依赖性只能是由外向内，外圈的业务相对内圈来说是完全屏蔽的。但是圆圈的个数是视实际状况而定的，并不是说和示意图一样只能是4个。

一下是对上图出现的词汇的解释:

* _Entities_: 应用的业务对象
* _Use Case_: 这些Uses Cases用来协调与Entities交互的数据流。（也称之为Interactors）
* _Interface Adapters_: 这组适配器将数据从最方便的格式转化为Use Case 和 Entities。（Presenters & Controllers属于这一层）
* _Frameworks and Drivers_: UI,tools,frameworks,etc

更加详细的介绍[文章](http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) & [视频](http://vimeo.com/43612849) 。

## Android Architecture
它的目标是通过将业务规则与外界隔离来实现关注分离，以到达在不依赖任何外部元素的情况下进行测试。

要做到这一点，我的建议是把项目分为各具目的的3层，并且每一层分别实现自己的功能。（单一职责）

需要注意的是，每一层都是用自己的数据模型，这样才能到达独立性（你能够在代码中看到为了实现数据转换而存在一些数据映射器。这意味着，如果你不想在整个应用中交叉使用你的数据模型，那么你将付出一些代价）。
通过下图，你可以看到它的大致样子:

![](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_android.png)

**注意:** 为了使这个例子更加清晰，我没有使用任何的第三方库（除了Gson，mockito，robolectric和espresso）。但是，请你不要犹豫使用ORMs和第三方依赖注入框架或者任意一个你熟悉的工具后者类库，这会让你的生活更加容易（请记住，重复造轮子是不明智的）。

### Presentation Layer
与视图和动画相关的逻辑在这一层发生。这一层的实现，你不仅可以使用**MVP**，你还可以使用类似MVC和MVVM的任何一种模式。在这里我不会细讲MVP，但是有一点你需要记住，**Fragment和Activity仅仅是view**，在其中不会有除了UI逻辑和渲染逻辑之外的任何逻辑。
在这一层中的**Presenter**是由Android UI线程外的执行工作的Interators组成，并且通过回调来返回数据，最后view负责将数据展示出来。

![](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_mvp.png)

[**Effective Android UI**](https://github.com/pedrovgs/EffectiveAndroidUI/)

### Domain Layer
**业务逻辑层：**所有的业务逻辑都在这儿发生。关于Android工程，你可以看到所有的Interactors都在这儿实现。

**这一层是一个纯java模块，没有任何的Android依赖**。所有的外部组件都是通过接口访问这儿的业务对象。

![](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_domain.png)

### Data Layer
应用所需的所有数据都是通过该层的一个实现 [**Repository Pattern**](http://martinfowler.com/eaaCatalog/repository.html)  的Repository的实现类来提供。
例如，当需要通过id来获取一个User对象，如果该对象存在于缓存中的话就从缓存中获取，否则就从远程服务器请求然后将返回的User对象放入缓存。
所有这一切背后的想法是**数据源对客户端来说是透明的**。客户端只需要获取到数据，但是根本不需要关心数据是来自哪里(内存，磁盘,服务器等)

![](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_data.png)

### Error Handling
异常处理一直是一个值得讨论的话题，你们要是有好的解决方式我们可以在这探讨。在这里我是采用回调的策略来处理异常的。比如说我的回调里定义了两个函数**OnResponse（）**和**onError（）**，然后在数据仓库层中发生了异常。在**onError（）**中携带回一个封装了异常的包装类**ErrorBundle**。这种方式伴随着一些问题，因为在异常传递到Presentiation层之前，会一直存在一条回调链。这其实是会影响代码的可读性。

### Testing

关于测试，基于不同的层我选择了不同的测试方案：

* **Presentation Layer：**用Android Instrumentation 和 Espresso来进行集成和功能测试。
* **Domain Layer：** 用JUnit 和 mockito进行单元测试。
* **Data Layer：** 用Robolectric，JUnit和mockito进行集成和单元测试。

### Source Code
Talk is cheap,show me the code .[Here is the Github link.](https://github.com/android10/Android-CleanArchitecture)

### Conclusion
我完全赞同Uncle Bob的说法：“Architecture is About Intent,not Frameworks”。我们每天都会面对各种各样的挑战，我们也会有很多种方式去解决它们。当你在使用上述的方法来解决你的问题时，你会发现你的应用程序将会是这样的：

* **易于维护**  
* **易于测试**  
* **高内聚，低耦合**  
* **...**

### Further Reading
1. [Architecting Android..the evolution](http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/)
2. [Tasting Dagger2 on Android](http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
3. [The Mayans Lost Guide to RxJava on Android](https://speakerdeck.com/android10/the-mayans-lost-guide-to-rxjava-on-android)
4. [It is about philosophy:Culture of a good programmer](https://speakerdeck.com/android10/it-is-about-philosophy-culture-of-a-good-programmer)

### Links and Resource
1. [The clean architecture by Uncle Bob](http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html)
2. [Architecture is About Intent,not Frameworks](http://www.infoq.com/news/2013/07/architecture_intent_frameworks)
3. [Model View Presenter](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter)
4. [Repository Pattern by Martin Fowler](http://martinfowler.com/eaaCatalog/repository.html)
5. [Android Design Patters Presentation](http://www.slideshare.net/PedroVicenteGmezSnch/)