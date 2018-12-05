---
title: 移动端工程架构与后端工程架构的思想摩擦之旅
categories: Architecture
date: 2018-11-30 10:11:57
tags: [Architecture, Android, BackEnd, Design, Modularization]
---

<blockquote class="blockquote-center">记资源投放后端工程的架构调整与优化</blockquote>

# 架构思考

一直以来对软件工程架构有着极大的兴趣，无论是之前负责的移动端Android工程，亦或是现在转到后端开发后维护的资源投放工程。可以说一个团队中并非每个开发都能够深入掌握架构知识，但需要每个人能够拥有软件架构的意识。架构是对工程整体结构与组件的抽象描述，是软件工程的基础骨架。架构在工程层面不分领域，且思想是通用的。引用维基百科对于软件架构的定义[^1]：

> 软件体系结构是构建计算机软件实践的基础。与建筑师设定建筑项目的设计原则和目标，作为绘图员画图的基础一样，软件架构师或者系统架构师陈述软件架构以作为满足不同客户需求的实际系统设计方案的基础。从和目的、主题、材料和结构的联系上来说，软件架构可以和建筑物的架构相比拟。一个软件架构师需要有广泛的软件理论知识和相应的经验来实施和管理软件产品的高级设计。软件架构师定义和设计软件的模块化，模块之间的交互，用户界面风格，对外接口方法，创新的设计特性，以及高层事物的对象操作、逻辑和流程。

架构的合理设计可以解决面对复杂系统时可能面临的很多问题，例如：

* 业务边界与模块职责划分问题
* 代码权限控制问题（数据库不应直接被业务方调用）
* 代码重复，逻辑分支多，坏味道多的问题
* 由于考虑不周，可能存在隐藏bug
* 修改一个逻辑需要修改N个地方代码逻辑

从实际的实践来看，的确如此。以前在移动端做的架构设计流程，在后端重新得到了实践。

## 移动端架构思考

尚未接触到强大的Spring容器之前，我一直探索着在移动端有一种能够在编译期暴露服务声明，运行时自动注入实现类的做法；接触到Spring以后，得以理解这其实就是IoC容器的概念。Android的组件化思想，以及网上发布的各类组件化的技术文章，给了我很多值得借鉴的思路。客户端的代码一般是以module来组织的。一个module，既可以配置成为一个独立发布的库，也可以编译成一个单独的apk。组件化的概念正是利用了module这一特点，将一个大工程中的业务拆分成一个个module，各个module间的业务相对独立，组件间通过各自暴露的业务接口实现通信。基于此思想的移动端架构模型可以使用下图来表示。

![Android组件化架构设计](https://haitao.nos.netease.com/ca4a1c6f-6c3a-40f7-a09f-46dddde7d0f7_1305_937.png)
[点击查看原图](https://haitao.nos.netease.com/ca4a1c6f-6c3a-40f7-a09f-46dddde7d0f7_1305_937.png)

该架构模型由5个部分组成，分别是Toolkit/ToolkitSDK module、基础组件库/基础组件库module、基础服务接口/业务服务接口module、服务调度中心module以及业务module。

* Toolkit/Toolkit SDK

Toolkit是工具类及与工具类相关的SDK的集合。工具类属于工程架构里最基础的模块，提供了通用的方法与工具类服务（工具类服务是指可以被抽象成一个独立的与业务无关的基础服务，如缓存、数据库操作等）。工具类通常作为最底层的module，被其他所有模块引用。

* 基础组件库/基础组件SDK

基础组件库是基础组件及相关SDK的集合。基础组件库提供与业务相关的基础组件，是构建一个移动端应用所需要的通用组件的集合。它与工具类的区别在于基础组件库可能会包含少量业务逻辑代码，是无法拆分给其他应用使用的；另一方面，基础组件库是基础服务接口的实现，是不对业务层暴露的，避免了业务层与基础SDK打交道，有利于整体替换底层基础框架的实现（例如Volley替换为OkHttp、Fresco替换为Glide）。

* 基础服务接口/业务服务接口

基础服务接口声明了一组通用的基础服务，业务层通过基础服务接口获取基础服务，如网络请求、图片加载等。业务服务接口声明了一组该模块提供给其他模块的服务，业务之间的通信也是通过服务接口来完成的。例如首页模块需要获取购物车的商品数量，首先通过服务调度中心获取购物车的服务接口，再通过服务接口调用购物车获取商品数量的接口方法即可。

* 服务调度中心

服务调度中心，是一个接口收集与管理的容器。服务调度中心将所有基础服务接口与业务接口收集起来，通过一定的方式与它们的实现类进行绑定。所有的业务都需要通过服务调度中心才能够获取到服务。服务的注册与发现和Spring容器的IoC思想是类似的。

* 业务层

业务层是每个业务的具体实现的集合。业务层的业务之间是没有直接引用关系的，业务层提供了业务服务接口中暴露的服务的具体实现。业务之间的通信需要通过服务调度中心获取其他业务的服务接口。

### 移动端架构小结

通过接口服务架构模型，模块之间是高度解耦的。业务负责人唯一需要维护的公共部分便是这个模块在业务服务接口中暴露的服务。对于业务服务的接口功能增改变得非常方便，业务实现的逻辑更改、代码优化等，只要不改变服务接口的签名，就不需要其他业务方改动任何代码即可完成，由此团队的开发效率是非常高的。

## 后端架构思考

对于后端工程来说，架构的设计与实现必定是与工程的业务难度及复杂程度相关的，如果只是很简单的业务模型，就没有必要弄得太过复杂，避免得不偿失。本人只接触了几个月后端知识，对于后端的架构体系与演进过程处于不断地学习和探索中。投放系统是我接触到的第一个完整的后端工程，其中Web工程采用传统的MVC架构[^2]，对我具有很大地学习和借鉴意义，项目架构如下图所示。

![投放Web工程MVC架构图](https://haitao.nos.netease.com/62129445-5378-4d34-b18e-11ee48d6fdf9_846_833.png)
[点击查看原图](https://haitao.nos.netease.com/62129445-5378-4d34-b18e-11ee48d6fdf9_846_833.png)

该架构纵向划分成展示层、控制层、服务层、对象关系映射层和数据服务层5个部分，层级间通过AOP的方式插入了业务监控、日志、权限控制、统计分析等功能。

* 展示层（View）

展示层是系统与用户打交道的地方，提供与用户交互的界面。对于用户而言，只有展示层是可见的、可操作的。展示层对于某些工程来说不是必须的，例如提供纯后台服务的工程。

* 控制层（Controller）

主要负责与Model和View打交道，但同时又保持其相对独立。Controller决定使用哪些Model，对Model执行什么操作，为视图准备哪些数据，是MVC中沟通的桥梁。在Controller层提供了http服务供展示层调用。在依赖管理中，控制层需要依赖服务层提供服务。

* 服务层（Service/Facade）

服务层是业务逻辑实现的地方，上层需要使用的功能都在服务层来实现具体的业务逻辑。服务层就是将底层的数据通过一定的条件和方式进行数据组装并提供给上层调用。服务层可以拆分为业务接口和业务实现，业务实现可以对外部隐藏。在投放工程中，控制层既依赖了业务接口，又依赖了业务实现。后面的改造我们可以看到，编译期红色线依赖是完全没有必要的。服务层需要依赖数据关系映射层与持久层的数据打交道。

* 对象关系映射层（ORM）

对象关系映射层的作用是在持久层和业务实体对象之间作一层数据实体的映射，这样在具体操作业务对象时，只需简单的操作对象的属性和方法，不需要去和复杂的SQL语句打交道。ORM使得业务不需要关心底层数据库的任何细节，包括使用的数据库类型、数据库连接与释放细节等。对象关系映射层只依赖数据服务层提供服务。

* 数据服务层（Data Server）

数据服务就是提供数据源的地方。数据服务可以提供持久化数据及缓存数据。持久，即把数据（如内存中的对象）保存到可永久保存的存储设备中（如磁盘）。持久化的主要应用是将内存中的数据存储在关系型的数据库中，当然也可以存储在磁盘文件中、XML数据文件中等等。而缓存是将信息（数据或页面）放在内存中以避免频繁的数据库存储或执行整个页面的生命周期，直到缓存的信息过期或依赖变更才再次从数据库中读取数据或重新执行页面的生命周期。数据服务层是数据源头，处于架构的最底层。

### 后端架构小结

后端工程，更加注重层级的概念，每一层的职责非常明确。展示层负责与用户进行页面交互，控制层合并业务数据并控制View的展示，服务层则是实现业务逻辑的聚集地，对象关系映射层在业务层和数据服务层之间建立通道，而数据服务层则提供数据。总体而言，投放工程的MVC架构给我的感觉是比移动端架构复杂，层级多，职责分工明确，带来的问题是层级间的交互也比较麻烦。另外服务层里承载了几乎所有的业务逻辑，层级偏重，如果没有好好地梳理业务逻辑划清边界，很容易把服务层搞成一锅粥。清晰的模块职责划分，可以帮助服务层更好地为控制层服务。

# 架构思想的摩擦

可以看到，客户端与后端有着非常相似的架构模型。

* 从代码组织的角度：以module作为层级代码组织的基本工具，分为工具库、基础组件库（中间件）、服务接口/API、服务层/业务层、视图层等。module间的依赖关系几乎是一样的。
* 从业务模型的角度：后端工程分为交易组、商品组、售后组、客服组等，对应移动端的交易链路浮层、商品详情页、售后详情页、帮助与客服页等，每个业务是由不同的组负责的，业务之间通过约定的接口相互提供服务，各种各样的业务模型聚合成了整个系统。
* 从功能服务的角度：分为业务服务接口的暴露、业务服务实现的隔离、业务服务的查找与注册。

下面从功能服务的角度，详细说明本文在思想摩擦过程中想要表达的观点。

## 业务接口的作用

业务接口，可以认为是这组业务向外暴露其功能的一套标准。标准一旦形成并发布，就需要业务方持续维护这套标准，使得标准变得完善和稳定。同时标准可以更新升级，可以通过版本来实现，提供新的功能。业务接口一般具备以下特性：

* 业务接口包含一组Java的接口集合以及与这些接口相关的POJO，通常打包成一个JAR/AAR包。
* 业务接口只提供接口功能的定义，不包含任务业务逻辑。
* 业务接口可以进行版本管理，一旦版本发布，则该版本的接口不再可变。业务需要新增功能时，只需要在原有业务接口的基础上，增加新的功能接口或方法，同时升级业务接口版本号并发布。

其他业务方需要使用该业务的功能，只需要引入该业务的JAR/AAR包，通过服务调度中心获取该服务接口即可。

## 业务逻辑的存放

业务接口仅提供了功能的定义，不包含任何业务逻辑。那么，业务逻辑（即接口的实现类）放哪里呢？不管是移动端架构还是后端架构，在工程领域，业务逻辑在任何时候都不应该对业务的使用方暴露。这样做有两个好处：

* 业务方只关心功能，不关心功能实现的过程。隐藏业务的实现逻辑可以降低业务方使用该功能的成本及复杂度。
* 业务功能的后端逻辑改动及必要的技术优化、性能优化，只要不更改接口签名，则不会影响当前的业务方使用。

一般情况下，业务的逻辑实现会放在单独的业务模块中，该业务模块仅限工程内部引用。后端传统的MVC工程架构把业务的实现逻辑放在了Service/Facade层，层级之间的类不相互引用；而基于驱动领域的设计模型把业务的实现逻辑限定在了一个领域/子域里，领域之间通过界限上下文绑定。在移动端，时下较为热门的众多组件化方案，也是将一个独立的功能模块作为单独的module，module可以独立编译为apk，也可以通过aar的方式集成到主Application中。

## 业务逻辑的隔离

业务的使用方包括工程内部的上层业务和外部服务，业务接口与业务逻辑有必要进行代码级别的隔离，这样才能避免上层业务引用到业务逻辑的代码。通常，工程的每个模块负责不同的功能，模块之间的引用关系通过依赖管理工具（如Maven或Gradle）来配置。我们可以巧用依赖管理工具的runtime compile机制来实现运行时依赖。即在编写对外接口的时候，不直接引用包含业务逻辑的module，等到编译的时候再把业务逻辑代码一起编译进来，然后在运行时通过一定的方式调用对应的业务逻辑。

* Maven通过在dependencyManagement的依赖中加入<scope>runtime<scope>标签实现[^3]。
* Gradle通过在dependencies将compile改为runtime实现[^4]。

下面的两张图简单介绍了后端工程和移动端工程基于业务逻辑隔离的工程架构思路。其中，虚线表示runtime compile依赖，实线表示正常的依赖关系。

![后端业务逻辑隔离思路](https://haitao.nos.netease.com/b91b60eb-32cc-4c81-8a61-5acba577a60a_824_423.png)
[点击查看原图](https://haitao.nos.netease.com/b91b60eb-32cc-4c81-8a61-5acba577a60a_824_423.png)

工程的启动入口几乎不包含业务代码，只包含配置文件。Controller层是一组RestfulApi的集合，给前端和客户端提供http请求服务，业务接口/API是一组dubbo接口，给其他工程业务方提供RPC调用。Controller层在编码的时候只依赖Service/Facade接口，在编译期依赖Service和Facade接口的实现。这样设计还有一个好处是对DAO层的保护，DAO层只和Service层打交道，Controller以及对外提供的dubbo接口是引用不到的，更好地保护数据安全。

![移动端业务逻辑隔离思路](https://haitao.nos.netease.com/9c1dcba3-de28-4e69-84d3-d85ee5bc5d89_672_638.png)
[点击查看原图](https://haitao.nos.netease.com/9c1dcba3-de28-4e69-84d3-d85ee5bc5d89_672_638.png)

在移动端的架构中，单Application+多module已经成为主流。每个module负责一块独立的业务，如首页、订单、购物车等，核心模块也可以拆分为独立模块，如网络引擎、图片引擎。这些独立的模块可以抽离出BusinessService/CoreService服务接口，模块间的交互只需要通过Service接口通信即可，业务对于其他的p_CoreSDK/Business的逻辑实现是不可见的。

## 业务接口的注册

有了业务接口和业务实现，还需要一种在运行时把它们“粘合”起来的工具，这一过程可以称为业务接口的注册。当业务方访问业务接口时，这个工具需要帮助我们查找到对应的业务实现。控制反转（IoC）或依赖注入（DI）的思想给我们提供了解决办法。

在后端工程中，Spring是最为常用的IoC容器之一。Spring在运行时根据配置文件或注解动态生成对象，再由变量注解通过Java反射注入到对应的实例中。因此代码中只需要通过在全局变量声明相应的注解即可完成业务接口的注册。

移动端由于有限的硬件资源，更多地把CPU时间分配给了页面渲染，保证应用体验流畅，不太可能在应用启动的过程中大量通过反射生成实例对象，因此移动端并没有出现Spring框架。尽管如此，依赖注入的思想是通用的。通常移动端只需要保证在运行时能够获取到对应的业务实现，几乎没有在运行时动态改变业务实现的需求，聪明的工程师想到了把服务的注册提前到编译期进行，这一过程可以使用JDK提供的`Annocation Processing Tool`完成（例如[Dagger2](https://github.com/google/dagger)），也可以在编译生成Class字节码以后使用ASM操作字节码注册实现（如[ARouter](https://github.com/alibaba/ARouter)）。

# 投放工程架构调整

有了前面的“理论基础”，以及跃跃欲试的心动，我们来对投放工程的架构做一次调整和优化，原则是不改变原有的业务逻辑，目的是使投放工程的业务边界和业务功能更为清晰。

## 旧工程架构

资源投放系统一共分为4个工程，分别是

* 后端前台工程：**kaola-resource-app**
* 后端离线工程：**kaola-resource-offline**
* 后端后台Web工程：**kaola-resource-web**
* 前端后台fed工程：**kaola-resource-fed**

其中，kaola-resource-app和kaola-resource-offline提供纯dubbo服务。app提供前台服务，负责给其他业务在线获取投放的素材信息；offline提供离线服务，负责给其他业务的后台系统获取投放相关的数据。kaola-resource-web和kaola-resource-fed分别是投放系统的Web后端和前端工程，资源的分配、审核、投放都在该系统完成。kaola-resource-fed属于前端工程，不在本文讨论范围内，下面介绍三个后端工程的架构体系。

### 工程依赖关系

![resource-management-classified-old](https://haitao.nos.netease.com/f3bd5ae8-3d32-4337-8001-73cfa7a633a1_1254_470.png)
[点击查看原图](https://haitao.nos.netease.com/f3bd5ae8-3d32-4337-8001-73cfa7a633a1_1254_470.png)

工程依赖关系如图所示，主要分为几类，分别是工具模块、接口模块、业务模块以及启动模块，箭头表示依赖关系。

* 工具模块

**kaola-common-cache**：主要包含Solo缓存中间件的封装，JVM缓存相关的不在该模块里。
**kaola-common-util**：工具类，包括常量、异常信息及util类。
**kaola-resource-search**->kaola-resource-generic：ndir索引服务工程。
**kaola-resource-schema（图中未列出）**：存放投放自带部分模板xml的地方，暂无其他工程引用。

* 接口模块

**kaola-resource-api**：提供在线和离线服务的dubbo接口。
**kaola-resource-generic**：通过mybatis与数据库打交道，提供数据的工程，相当于DAO层。
**kaola-resource-facade**->kaola-resource-generic：提供Web层的facade接口，相当于Service层。

* 业务实现

**kaola-resource-cache**->kaola-resource-api、kaola-resource-generic、kaola-common-cache：提供业务报警、disconf配置、JVM缓存、NCR缓存、熔断、业务监控、线程池管理等功能，且提供resource-api部分facade接口的实现、Service接口定义与实现，是resource-app工程的业务实现工程。
**kaola-resource-provider**->kaola-resource-facade、kaola-resource-api、kaola-service-util、kaola-resource-search、kaola-resource-generic：提供web工程facade接口的实现、Service接口定义与实现。
**kaola-resource-service**->kaola-resource-generic、kaola-resource-api：提供offline工程facade接口的实现、Service接口定义与实现。
**kaola-service-util**->kaola-resource-facade、kaola-common-util、kaola-resource-api：主要包含外部模板XML校验业务的工程。

* 启动模块

**kaola-resource-app**->kaola-resource-cache、kaola-common-cache、kaola-resource-api、kaola-resource-generic：app工程启动入口，SpringBoot配置。
**kaola-resource-offline**->kaola-resource-service、kaola-resource-generic、kaola-resource-api：offline工程启动入口，SpringBoot配置。
**kaola-resource-fed**：前端后台工程
**kaola-resource-web**->kaola-resource-facade、kaola-resource-provider、kaola-resource-facade、kaola-resource-api、kaola-service-util、kaola-resource-search、kaola-resource-generic、kaola-common-util：web工程启动入口，提供http服务。

### 现有工程问题

* 工程模块命名混乱

刚接触投放工程的时候完全不知道从何入手，只能根据启动工程的依赖关系找到对应的业务模块。总工程里有4个小工程，通常找一个模块的业务实现不知道去哪个模块去找。例如，从命名上来看，并不知道kaola-resource-facade是web工程的业务接口，kaola-resource-provider是web工程的业务实现，kaola-resource-service是offline工程的业务实现，对新人来说命名不够友好。

* 模块职责不清晰

由上面梳理的模块功能可以知道，kaola-resource-cache工程不仅承担了缓存管理、业务报警、熔断等基础业务，也是app工程的业务实现；kaola-resource-service既提供了业务接口，又提供了业务实现；kaola-common-util没有发挥它作为工具类应该提供的职责和义务。总体而言，模块的职责可以通过一定的修改变得更为清晰。

* 工具类、常量类管理混乱

目前几乎每个工程下面都有一个util包，且工程包含多个BusinessException/ParameterException类，带来的问题是使用时不知道用哪个类。工具类与常量类应统一为一个工具类模块，统一管理。

* 版本依赖管理

每个工程的依赖目前都是独立管理的，存在不同工程依赖中间件版本不一致的情况。主工程的pom管理机制可以优化一下，比如disconf是每个工程都要使用的，可以放在父工程使用dependencyManagement统一管理。

* 业务代码分支多

当前工程存在很多业务分支，遇到业务分支时大部分是通过if/else判断的，修改一个业务点需要修改很多地方，通常很容易遗漏，造成隐藏的bug。

## 新工程架构

考虑到现有工程架构的特点，在新工程的设计上采用平滑过渡的方式，后端主体还是分为web/offline/app三个小工程，这三个小工程分别采用传统的MVC架构。前端工程由于与后端工程关系不大，后续可以迁移到新工程里。

### 工程依赖关系调整

针对旧工程中存在的不合理的点，以及工程中存在的问题，我们对工程及工程的依赖关系进行了如下调整。

![resource-management-classified-old-vs-new](https://haitao.nos.netease.com/46ee4c0a-8abd-4ac3-8932-2dca5044ff03_1604_599.png)
[点击查看原图](https://haitao.nos.netease.com/46ee4c0a-8abd-4ac3-8932-2dca5044ff03_1604_599.png)

红色背景为删除或整合的模块，包括kaola-service-util、kaola-common-util、kaola-common-cache。
黄色背景为重命名的模块，目的是使模块功能定位更清晰，包括

* kaola-resource-provider->kaola-resource-web-provider，表示web工程的业务实现
* kaola-resource-facade->kaola-resource-web-facade，表示web工程的业务接口
* kaola-resource-generic->kaola-resource-dao，表示对象关系映射，即DAO层
* kaola-resource-service->kaola-resource-offline-provider，表示offline工程的业务实现

绿色背景为新增模块，包括kaola-resource-util、kaola-resource-offline-facade、kaola-resource-app-provider、kaola-resource-app-facade，模块的定义也比较明确。
红色的线为可以移除的依赖关系，黄色的线表示修改为runtime依赖，绿色的线表示新增依赖。

经过上述调整后，一张更为清晰的投放工程架构图便展现出来了。

![resource-management-classified-new](https://haitao.nos.netease.com/9880b70c-9810-4f82-b418-68b2f557218b_1250_520.png)
[点击查看原图](https://haitao.nos.netease.com/9880b70c-9810-4f82-b418-68b2f557218b_1250_520.png)

### 工程架构基本思想

新的工程架构，模块间的职责与依赖关系更为清晰。

从模块的职责来说，浅蓝色表示底层提供的工具类（包括各种util、缓存、索引服务等）；绿色表示数据对象关系映射服务层；黄色表示对第三方暴露的api；粉红色表示对内提供的业务接口；土黄色表示业务接口的真正实现；深蓝色则是工程的启动入口，包含各种配置文件。

从模块的依赖关系来说，启动工程在编译期仅依赖业务接口，运行时才会依赖业务实现以及与业务实现相关的底层服务，这一操作通过maven的scope标签完成，编译期编译使用的标签为`<scope>compile</scope>`，运行时编译使用的标签为`<scope>runtime</scope>`。索引、缓存、工具类作为基础服务以独立的模块存在，需要与数据库打交道的模块则直接依赖kaola-resource-dao即可。

对于当前的架构还不是最完美的，存在两个问题。

1. kaola-resource-cache提供了kaola-resource-api的实现，应将这一部分的业务逻辑移至kaola-resource-app-provider中。
2. 业务接口依赖了kaola-resource-dao，这一部分因为接口使用了dao层的Entity，需要重新规划建模，去除对dao层的依赖。

该架构的设计可以做到业务层仅和接口打交道，在业务层实现业务逻辑时，不关心底层服务的实现。业务层也没有办法直接和dao层打交道，避免数据被业务层无故修改。

### 工程模块划分

* 基础模块/服务

**kaola-resource-fed**：前端工程单独拆出，不与后端工程公用同一个git仓库
**kaola-resource-util**：工具类、异常类、常量，可以被所有子工程引用
**kaola-resource-cache**：缓存相关的模块，需要使用缓存的工程引用此module
**kaola-resource-index**：索引工程服务
**kaola-resource-dao**：数据库dao层
**kaola-resource-api**：暴露对第三方提供的所有服务api，包括online和offline接口
**kaola-resource-facade-base**：存放所有facade工程的基础接口和异常类，上述流程图没有展示出来，但确实是有必要的基础模块。

* web工程

**kaola-resource-web-facade**：web工程服务暴露，仅提供facade接口和数据实体类
**kaola-resource-web-provider**：web工程服务实现，根据需要自定义service层组件
**kaola-resource-web**：web工程启动类，compile依赖facade，runtime依赖provider

* offline工程

**kaola-resource-offline-facade**：offline工程内部服务，仅提供接口和数据实体类
**kaola-resource-offline-provider**：offline工程服务实现，及部分kaola-resource-api实现，提供dubbo provider
**kaola-resource-offline**：offline工程启动类，compile依赖facade，runtime依赖provider

* app工程

**kaola-resource-app-facade**：app工程内部服务，仅提供接口和数据实体类
**kaola-resource-app-provider**：app工程服务实现，及部分kaola-resource-api实现，提供dubbo provider
**kaola-resource-app-starter**：app工程启动类，compile依赖facade，runtime依赖provider

## 现有工程改造方案

### 工程模块改造

在上述改造原则基础上，投放工程的模块改造计划可以清晰地列出来。针对基础模块来说，需要进行如下改造：

* 新增kaola-resource-facade-base工程，存放facade的异常类
* kaola-common-cache重命名为kaola-resource-cache
* kaola-common-util重命名为kaola-resource-util
* kaola-resource-generic重命名为kaola-resource-dao
* 新增kaola-resource-facade-base工程，存放facade的异常类

剩余的三个工程，以resource-web工程为例，需要进行如下改造：

* 将kaola-resource-facade重命名为kaola-resource-web-facade，存放对web工程提供的服务接口，将BusinessException和ParameterException移除放到facade-base工程中
* kaola-resource-provider重命名为kaola-resource-web-provider，将模块中的util迁移至resource-util工程
* kaola-service-util中非业务相关的所有util全部移动到kaola-resource-util这个module中，剩余业务相关的，移动到kaola-resource-web-provider工程
* kaola-resource-web，将web对provider的依赖由compile改为runtime

offline和app工程采用类似的方案，不再赘述。

### maven依赖改造

maven依赖改造的目的，一是减少由于版本依赖不一致带来的新坑，方便第三方中间件的统一管理；二是对工程权限有所约束，不至于代码乱放，难以维护。

* 共有的jar包依赖管理
在父pom增加dependencyManagement标签，把常用的Spring、log、guava、junit、jsonserializer、disconf和网易自研的中间件等jar包统一进行版本管理。子工程需要相关组件时，仅需引入依赖，无需指定版本。

* 模块依赖权限管理
由改造后的工程架构可知，启动工程是没有办法接触到业务实现的，这需要依靠runtime时再依赖业务实现模块来实现。因此，对于每个需要依赖*-provider的工程，都需要通过<scope>runtime</scope>来依赖。

* api版本发布管理
需要对外发布的包，依赖尽量使用provided引入。在父pom增加本地api发布的版本，子工程如需依赖api或实现api的业务，需要引用父pom的api版本，api版本的升级，需要同时升级父pom和api模块工程两个地方
resource-util工程与api工程管理一致。

# TODO

* kaola-resource-api迁移所有page相关的api接口
* kaola-resource-fed迁移使用新的git工程
* kaola-resource-cache逻辑迁移
* kaola-resource-dao业务接口的依赖去除

# 架构调整总结

架构永远是聊不完的话题，特别是涉猎了知识面较广的后端以后，越发感觉需要学习的东西越来越多。本文分别从移动端和后端的架构思想谈起，从中探索共同点，结合微服务的思想，探索过程中对多端架构有了思想的摩擦，总结了一套业务接口与业务实现的规范，能够更好地为其他业务提供服务。在此基础上，将这一套规范运用在投放工程上，完成从旧架构的问题分析，到新架构的优化设计与实现的整个过程。从结果来看，模块的职责更为清晰，代码的权限与边界得到了比较好的控制，对以后业务的纵横向扩展、代码的健壮性都有了质的保障。

# 参考链接

1. <https://zh.wikipedia.org/wiki/%E8%BD%AF%E4%BB%B6%E6%9E%B6%E6%9E%84>
2. <https://zh.wikipedia.org/wiki/MVC>
3. <https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html>
4. <https://docs.gradle.org/current/userguide/dependency_types.html>

[^1]: <https://zh.wikipedia.org/wiki/%E8%BD%AF%E4%BB%B6%E6%9E%B6%E6%9E%84>
[^2]: <https://zh.wikipedia.org/wiki/MVC>
[^3]: <https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html>
[^4]: <https://docs.gradle.org/current/userguide/dependency_types.html>