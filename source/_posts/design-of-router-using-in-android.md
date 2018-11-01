---
title: 考拉Android客户端路由总线设计
date: 2017-07-12 09:42:59
tags: [Android,Router,Priority,Regular Expression, Annotation, Annotation Processor]
categories: Android
---

# 前言

当前，Android路由框架已经有很多了，如雨后春笋般出现，大概是因为去年提出了Android组件化的概念。当一个产品的业务规模上升到一定程度，或者是跨团队开发时，团队/模块间的合作问题就会暴露出来。如何保持团队间业务的往来？如何互不影响或干涉对方的开发进度？如何调用业务方的功能？组件化给上述问题提供了一个答案。组件化所要解决的核心问题是解耦，路由正是为了解决模块间的解耦而出现的。本文阐述了考拉Android端的路由设计方案，尽管与市面上的方案大同小异，但更多的倾向于与考拉业务进行一定程度的结合。

## 传统的页面跳转

页面跳转主要分为三种，App页面间跳转、H5跳转回App页面以及App跳转至H5。

### App页面间跳转

App页面间的跳转，对于新手来说一般会在跳转的页面使用如下代码：

```
Intent intent = new Intent(this, MainActivity.class);
intent.putExtra("dataKey", "dataValue");
startActivity(intent);
```

对于有一定经验的程序员，会在跳转的类生成自己的跳转方法：

```
public class OrderManagerActivity extends BaseActivity {
	public static void launch(Context context, int startTab) {
		Intent i = new Intent(context, OrderManagerActivity.class);
		i.putExtra(INTENT_IN_INT_START_TAB, startTab);
		context.startActivity(i);
	}
}
```

无论使用哪种方式，本质都是生成一个`Intent`，然后再通过`Context.startActivity(Intent)/Activity.startActivityForResult(Intent, int)`实现页面跳转。这种方式的不足之处是当包含多个模块，但模块间没有相互依赖时，这时候的跳转会变得相当困难。如果已知其他模块的类名以及对应的路径，可以通过`Intent.setComponent(Component)`方法启动其他模块的页面，但往往模块的类名是有可能变化的，一旦业务方把模块换个名字，这种隐藏的Bug对于开发的内心来说是崩溃的。另一方面，这种重复的模板代码，每次至少写两行才能实现页面跳转，代码存在冗余。

### H5-App页面跳转

对于考拉这种电商应用，活动页面具有时效性和即时性，这两种特性在任何时候都需要得到保障。运营随时有可能更改活动页面，也有可能要求点击某个链接就能跳转到一个App页面。传统的做法是对`WebViewClient.shouldOverrideUrlLoading(WebView, String)`进行拦截，判断url是否有对应的App页面可以跳转，然后取出url中的params封装成一个`Intent`传递并启动App页面。

感受一下在考拉App工程中曾经出现过的下面这段代码：

```
public static Intent startActivityByUrl(Context context, String url, boolean fromWeb, boolean outer) {
    if (StringUtils.isNotBlank(url) && url.startsWith(StringConstants.REDIRECT_URL)) {  
        try {
            String realUrl = Uri.parse(url).getQueryParameter("target");
            if (StringUtils.isNotBlank(realUrl)) {
                url = URLDecoder.decode(realUrl, "UTF-8");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    Intent intent = null;
    try {
        Uri uri = Uri.parse(url);
        String host = uri.getHost();
        List<String> pathSegments = uri.getPathSegments();
        String path = uri.getPath();
        int segmentsLength = (pathSegments == null ? 0 : pathSegments.size());

        if (!host.contains(StringConstants.KAO_LA)) {
            return null;
        }

        if((StringUtils.isBlank(path))){
            do something...
            return intent;
        }

        if (segmentsLength == 2 && path.startsWith(StringConstants.JUMP_TO_GOODS_DETAIL)) {
            do something...
        } else if (path.startsWith(StringConstants.JUMP_TO_SPRING_ACTIVITY_TAB)) {  
            do something...
        } else if (path.startsWith(StringConstants.JUMP_TO_SPRING_ACTIVITY_DETAIL) && segmentsLength == 3) { 
            do something...
        } else if (path.startsWith(StringConstants.START_CART) && segmentsLength == 1) { 
            do something...
        } else if (path.startsWith(StringConstants.JUMP_TO_COUPON_DETAIL)
                || (path.startsWith(StringConstants.JUMP_TO_COUPON) && segmentsLength == 2)) {
            do something...
        } else if (canOpenMainPage(host, uri.getPath())) { 
            do something...
        } else if (path.startsWith(StringConstants.START_ORDER)) { 
            if (!UserInfo.isLogin(context)) {
                do something...
            } else {
                do something...
            }
        } else if (path.startsWith(StringConstants.START_SAVE)) { 
            do something...
        } else if (path.startsWith(StringConstants.JUMP_TO_NEW_DISCOVERY)) {  
            do something...
        } else if (path.startsWith(StringConstants.JUMP_TO_NEW_DISCOVERY_2) && segmentsLength == 3) { 
            do something...
        } else if (path.startsWith(StringConstants.START_BRAND_INTRODUCE)
                || path.startsWith(StringConstants.START_BRAND_INTRODUCE2)) {  
            do something...
        } else if (path.startsWith(StringConstants.START_BRAND_DETAIL) && segmentsLength == 2) {  
            do something...
        } else if (path.startsWith(StringConstants.JUMP_TO_ORDER_DETAIL)) {  
            if (!UserInfo.isLogin(context) && outer) {
                do something...
            } else {
                do something...
            }
        } else if (path.startsWith("/cps/user/certify.html")) {   
            do something...
        } else if (path.startsWith(StringConstants.IDENTIFY)) { 
            do something...
        } else if (path.startsWith("/album/share.html")) {  
            do something...
        } else if (path.startsWith("/album/tag/share.html")) {  
            do something...
        } else if (path.startsWith("/live/roomDetail.html")) {   
            do something...
        } else if (path.startsWith(StringConstants.JUMP_TO_ORDER_COMMENT)) { 
            if (!UserInfo.isLogin(context) && outer) {
                do something...
            } else {
                do something...
            }
        } else if (openOrderDetail(url, path)) {
            if (!UserInfo.isLogin(context) && outer) {
                do something...
            } else {
                do something...
            }
        } else if (path.startsWith(StringConstants.JUMP_TO_SINGLE_COMMENT)) {  
            do something...
        } else if (path.startsWith("/member/activity/vip_help.html")) {
            do something...
        } else if (path.startsWith("/goods/search.html")) {
            do something...
        } else if(path.startsWith("/afterSale/progress.html")){  
            do something...
        } else if(path.startsWith("/afterSale/apply.html")){  
            do something...
        } else if(path.startsWith("/order/track.html")) { 
            do something...
        }
    } catch (Exception e) {
        e.printStackTrace();
    }

    if (intent != null && !(context instanceof Activity)) {
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    }
    return intent;
}

```

这段代码整整260行，看到代码时我的内心是崩溃的。这种做法的弊端在于：

* **判断不合理**。上述代码仅判断了HOST是否包含`StringConstants.KAO_LA`，然后根据PATH区分跳转到哪个页面，PATH也只判断了起始部分，当URL越来越多的时候很有可能造成误判。
* **耦合性太强**。已知拦截的所有页面的引用都必须能够拿到，否则无法跳转；
* **代码混乱**。PATH非常多，从众多的PATH中匹配多个已知的App页面，想必要判断匹配规则就要写很多函数解决；
* **拦截过程不透明**。开发者很难在URL拦截的过程中加入自己的业务逻辑，如打点、启动Activity前添加特定的Flag等；
* **没有优先级概念，也无法降级处理**。同一个URL，只要第一个匹配到App页面，就只能打开这个页面，无法通过调整优先级跳转到别的页面或者使用H5打开。

### App页面-H5跳转

这种情况不必多说，启动一个WebViewActivity即可。

## 页面路由的意义

路由最先被应用于网络中，路由的定义是通过互联的网络把信息从**源地址**传输到**目的地址**的活动。页面跳转也是相当于从**源页面**跳转到**目标页面**的过程，每个页面可以定义为一个统一资源标识符（URI），在网络当中能够被别人访问，也可以访问已经被定义了的页面。路由常见的使用场景有以下几种：

* App接收到一个通知，点击通知打开App的某个页面（OuterStartActivity）
* 浏览器App中点击某个链接打开App的某个页面（OuterStartActivity）
* App的H5活动页面打开一个链接，可能是H5跳转，也可能是跳转到某一个native页面（WebViewActivity）
* 打开页面需要某些条件，先验证完条件，再去打开那个页面（需要登录）
* App内的跳转，可以减少手动构建Intent的成本，同时可以统一携带部分参数到下一个页面（打点）

除此之外，使用路由可以避免上述弊端，能够降低开发者页面跳转的成本。

# 考拉路由总线

## 路由框架

 ![Alt pic](http://nos.netease.com/knowledge/0d94b9af-bb78-4a7c-9322-e23bcfec551c?imageView&thumbnail=980x0) 

[点击查看大图](http://nos.netease.com/knowledge/0d94b9af-bb78-4a7c-9322-e23bcfec551c)

考拉路由框架主要分为三个模块：**路由收集**、**路由初始化**以及**页面路由**。路由收集阶段，定义了基于Activity类的注解，通过`Android Processing Tool`(以下简称“APT”)收集路由信息并生成路由表类；路由初始化阶段，根据生成的路由表信息注入路由字典；页面路由阶段，则通过路由字典查找路由信息，并根据查找结果定制不同的路由策略略。

## 路由设计思路

总的来说，考拉路由设计追求的是功能模块的解耦，能够实现基本路由功能，以及开发者使用上足够简单。考拉路由的前两个阶段对于路由使用者几乎是无成本的，只需要在使用路由的页面定义一个类注解`@Router`即可，页面路由的使用也相当简单，后面会详细介绍。

### 功能设计

路由在一定程度上和网络请求是类似的，可以分为请求、处理以及响应三个阶段。这三个阶段对使用者来说既可以是透明的，也可以在路由过程中进行拦截处理。考拉路由框架目前支持的功能有：

1. 支持基本Activity的启动，以及startActivityForResult回调；✅
2. 支持不同协议执行不同跳转；（kaola://\*、http(s)://\*、native://\*等）✅
3. 支持多个SCHEME/HOST/PATH跳转至同一个页面；（(pre.)*.kaola.com(.hk)）✅
4. 支持路由的正则匹配；✅
5. 支持Activity启动使用不同的Flag；✅
6. 支持路由的优先级配置；✅
7. 支持对路由的动态拦截、监听以及降级；✅

以上功能保证了考拉业务模块间的解耦，也能够满足目前产品和运营的需求。

### 接口设计

 ![Alt pic](http://nos.netease.com/knowledge/130829f3-7355-44ee-af48-b2a7e8ffbbae?imageView&thumbnail=980x0) 

[点击查看大图](http://nos.netease.com/knowledge/130829f3-7355-44ee-af48-b2a7e8ffbbae)

一个好的模块或框架，需要事先设计好接口，预留足够的权限供调用者支配，才能满足各种各样的需求。考拉路由框架在设计过程中使用了常见的设计模式，如Builder模式、Factory模式、Wrapper模式等，并遵循了一些设计原则。（最近在看第二遍Effective Java，对以下原则深有体会，推荐看一下）

* **针对接口编程，而不是针对实现编程**

这条规则排在最前面的原因是，针对接口编程，不管是对开发者还是对使用者，真的是**百利而无一害**。在路由版本迭代的过程中，底层对接口无论实现怎样的修改，也不会影响到上层调用。对于业务来说，路由的使用是无感知的。

考拉路由框架在设计过程中并未完全遵循这条原则，下一个版本的迭代会尽量按照这条原则来实现。但在路由过程中的关键步骤都预留了接口，具体有：

**RouterRequestCallback**

```
public interface RouterRequestCallback {
    void onFound(RouterRequest request, RouterResponse response);

    boolean onLost(RouterRequest request);
}
```

路由表中是否能够匹配到路由信息的回调，如果能够匹配，则回调`onFound()`，如果不能够匹配，则返回`onLost()`。`onLost()`的结果由开发来定义，如果返回的结果是`true`，则认为开发者处理了这次路由不匹配的结果，最终返回`RouterResult`的结果是成功路由。

**RouterHandler**

```
public interface RouterHandler extends RouterStarter {
	RouterResponse findResponse(RouterRequest request); 
}
```

路由处理与启动接口，根据给定的路由请求，查找路由信息，根据路由响应结果，分发给相应的启动器执行后续页面跳转。这个接口的设计不太合理，功能上不完善，后续会重新设计这个接口，让调用方有权限干预查找路由的过程。

**RouterResultCallback**

```
public interface RouterResultCallback {
    boolean beforeRoute(Context context, Intent intent);

    void doRoute(Context context, Intent intent, Object extra);

    void errorRoute(Context context, Intent intent, String errorCode, Object extra);
}
```

匹配到路由信息后，真正执行路由过程的回调。`beforeRoute()`这个方法是在真正路由之前的回调，如果开发者返回`true`，则认为这条路由信息已被调用者拦截，不会再回调后面的`doRoute()`以及执行路由。在路由过程中发生的任何异常，都会回调`errorRoute()`方法，这时候路由中断。

**ResponseInvoker**

```
public interface ResponseInvoker {
    void invoke(Context context, Intent intent, Object... args);
}
```

路由执行者。如果开发需要执行路由前进行一些全局操作，例如添加额外的信息传入到下一个Activity，则可以自己实现这个接口。路由框架提供默认的实现：`ActivityInvoker`。开发也可以继承`ActivityInvoker`，重写`invoke()`方法，先实现自己的业务逻辑，再调用`super.invoke()`方法。

**OnActivityResultListener**

```
public interface OnActivityResultListener {
    void onActivityResult(int requestCode, int resultCode, Intent data);
}
```

特别强调一下这个Listener。本来这个回调的作用是方便调用者在执行`startActivityForResult`的时候可以通过回调来告知结果，但由于**不保留活动**的限制，离开页面以后这个监听器是无法被系统保存(`saveInstanceState`)的，因此不推荐在`Activity/Fragment`中使用回调，而是在非Activity组件/模块里使用，如`View/Window/Dialog`。这个过程已经由`core`包里的`CoreBaseActivity`实现，开发使用的时候，可以直接调用`CoreBaseActivity.startActivityForResult(intent, requestCode, onActivityResultListener)`，也可以通过`KaolaRouter.with(context).url(url).startForResult(requestCode, onActivityResultListener)`调用。例如，要启动订单管理页并回调：

```
KaolaRouter.with(context)
        .url(url)
        .data("orderId", "replace url param key.")
        .startForResult(1, new OnActivityResultListener() {
            @Override
            public void onActivityResult(int requestCode, int resultCode, Intent data) {
                DebugLog.e(requestCode + " " + resultCode + " " + data.toString());
            }
        });
```

**RouterResult**

```
public interface RouterResult {

    boolean isSuccess();

    RouterRequest getRouterRequest();

    RouterResponse getRouterResponse();
}
```

告知路由的结果，路由结果可以被干预，例如`RouterRequestCallback.onLost()`，返回`true`的时候，路由也是成功的。这个接口不管路由的成功或失败都会返回。

**不随意暴露不必要的API**

*“要区别设计良好的模块与设计不好的模块，最重要的因素在于，这个模块对于外部的其他模块而⾔言，是否隐藏其内部数据和其他实现细节。设计良好的模块会隐藏所有的实现细节，把它的API与它的实现清晰地隔离开来。然后，模块之间只通过它们的API进行通信，一个模块不需要知道其他模块的内部工作情况。这被称为封装(encapsulation)。”*（摘自Effective Java, P58）

举个例子，考拉路由框架对路由调用的入参做了限制，一旦入参，则不能再做修改，调用者无需知道路由框架对使用这些参数怎么实现调用者想要的功能。实现上，由`RouterRequestWrapper`继承自`RouterRequestBuilder`，后者通过Builder模式给用户构造相关的参数，前者通过Wrapper模式装饰`RouterRequestBuilder`中的所有变量，并在`RouterRequestWrapper`类中提供所有参数的get函数，供路由框架使用。

**单一职责**

无论是类还是方法，均需要遵循单一职责原则。一个类实现一个功能，一个方法做一件事。例如，`KaolaRouterHandler`是考拉路由的处理器，实现了`RouterHandler`接口，实现路由的查找与转发；`RouterRequestBuilder`用于收集路由请求所需参数；`RouterResponseFactory`用于生成路由响应的结果。

**提供默认实现**

针对接口编程的好处是随时可以替换实现，考拉路由框架在路由过程中的所有监听、拦截以及路由过程都提供了默认的实现。使用者即可以不关心底层的实现逻辑，也可以根据需要替换相关的实现。

## 考拉路由实现原理

考拉路由框架基于注解收集路由信息，通过APT实现路由表的动态生成，类似于ButterKnife的做法，在运行时导入路由表信息，并通过正则表达式查找路由，根据路由结果实现最终的页面跳转。

### 收集路由信息

首先定义一个注解`@Router`，注解里包含了路由协议、路由主机、路由路径以及路由优先级。

```
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.CLASS) 
public @interface Router {
    /**
     * URI协议，已经提供默认值，默认实现了四种协议：https、http、kaola、native
     */
    String scheme() default "(https|http|kaola|native)://";

    /**
     * URI主机，已经提供默认值
     */
    String host() default "(pre\\.)?(\\w+\\.)?kaola\\.com(\\.hk)?";

    /**
     * URI路径，选填，如果使用默认值，则只支持本地路由，不支持url拦截
     */
    String value() default "";

    /**
     * 路由优先级，默认为0。
     */
    int priority() default 0;
}
```

对于需要使用路由的页面，只需要在类的声明处加上这个注解，标明这个页面对应的路由路径即可。例如：

```
@Router("/app/myQuestion.html") 
public class MyQuestionAndAnswerActivity extends BaseActivity { 
	……
}
```

那么通过APT生成的标记这个页面的url则是一个正则表达式：

```
(https|http|kaola|native)://(pre\.)?(\w+\.)?kaola\.com(\.hk)?/app/myQuestion\.html
```
路由表则是由多条这样的正则表达式构成。

### 生成路由表

路由表的生成需要使用APT工具以及*Square*公司开源的[javapoet](https://github.com/square/javapoet)类库，目的是根据我们定义的`Router`注解让机器帮我们“写代码”，生成一个Map类型的路由表，其中key根据`Router`注解的信息生成对应的正则表达式，value是这个注解对应的类的信息集合。首先定义一个`RouterProcessor`，继承自`AbstractProcessor`，

```
public class RouterProcessor extends AbstractProcessor {
    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
		// 初始化相关环境信息
        mFiler = processingEnv.getFiler();
        elementUtil = processingEnv.getElementUtils();
        typeUtil = processingEnv.getTypeUtils();
        Log.setLogger(processingEnv.getMessager());
    }

    @Override
    public Set<String> getSupportedAnnotationTypes() {
        Set<String> supportAnnotationTypes = new HashSet<>();
        // 获取需要处理的注解类型，目前只处理Router注解
        supportAnnotationTypes.add(Router.class.getCanonicalName());
        return supportAnnotationTypes;
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        // 收集与Router相关的所有类信息，解析并生成路由表
        Set<? extends Element> routeElements = roundEnv.getElementsAnnotatedWith(Router.class);
        try {
            return parseRoutes(routeElements);
        } catch (Exception e) {
            Log.e(e.getMessage(), e);
            return false;
        }
    }
}
```

上述的三个方法属于AbstractProcessor的方法，`public abstract boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv)`是抽象方法，需要子类实现。

```
private boolean parseRoutes(Set<? extends Element> routeElements) throws IOException {
    if (null == routeElements || routeElements.size() == 0) {
        return false;
    }
    // 获取Activity类的类型，后面用于判断是否是其子类
    TypeElement typeActivity = elementUtil.getTypeElement(ACTIVITY);
	// 获取路由Builder类的标准类名
    ClassName routeBuilderCn = ClassName.get(RouteBuilder.class);
	// 构建Map<String, Route>集合
    String routerConstClassName = RouterProvider.ROUTER_CONST_NAME;
    TypeSpec.Builder typeSpec = TypeSpec.classBuilder(routerConstClassName).addJavadoc(WARNING_TIPS).addModifiers(PUBLIC);
    /**
     * Map<String, Route>
     */
    ParameterizedTypeName inputMapTypeName =
            ParameterizedTypeName.get(ClassName.get(Map.class), ClassName.get(String.class),
                    ClassName.get(Route.class));
    ParameterSpec groupParamSpec = ParameterSpec.builder(inputMapTypeName, ROUTER_MAP_NAME).build();
    MethodSpec.Builder loadIntoMethodOfGroupBuilder = MethodSpec.methodBuilder(METHOD_LOAD_INTO)
            .addAnnotation(Override.class)
            .addModifiers(PUBLIC)
            .addParameter(groupParamSpec);
	// 将路由信息放入Map<String, Route>集合中
    for (Element element : routeElements) {
        TypeMirror tm = element.asType();
        Router route = element.getAnnotation(Router.class);
		// 获取当前Activity的标准类名
        if (typeUtil.isSubtype(tm, typeActivity.asType())) {
            ClassName activityCn = ClassName.get((TypeElement) element);
            String key = "key" + element.getSimpleName().toString();
            String routeString = RouteBuilder.assembleRouteUri(route.scheme(), route.host(), route.value());
            if (null == routeString) {
                //String keyValue = RouteBuilder.generateUriFromClazz(Activity.class);
                loadIntoMethodOfGroupBuilder.addStatement("String $N= $T.generateUriFromClazz($T.class)", key,
                        routeBuilderCn, activityCn);
            } else {
                //String keyValue = "(" + route.value() + ")|(" + RouteBuilder.generateUriFromClazz(Activity.class) + ")";
                loadIntoMethodOfGroupBuilder.addStatement(
                        "String $N=$S + $S + $S+$T.generateUriFromClazz($T.class)+$S", key, "(", routeString, ")|(",
                        routeBuilderCn, activityCn, ")");
            }

            /**
             * routerMap.put(url, RouteBuilder.build(String url, int priority, Class<?> destination));
             */
            loadIntoMethodOfGroupBuilder.addStatement("$N.put($N, $T.build($N, $N, $T.class))", ROUTER_MAP_NAME,
                    key, routeBuilderCn, key, String.valueOf(route.priority()), activityCn);

            typeSpec.addField(generateRouteConsts(element));
        }
    }

    // Generate RouterConst.java
    JavaFile.builder(RouterProvider.OUTPUT_DIRECTORY, typeSpec.build()).build().writeTo(mFiler);

    // Generate RouterGenerator
    JavaFile.builder(RouterProvider.OUTPUT_DIRECTORY, TypeSpec.classBuilder(RouterProvider.ROUTER_GENERATOR_NAME)
            .addJavadoc(WARNING_TIPS)
            .addSuperinterface(ClassName.get(RouterProvider.class))
            .addModifiers(PUBLIC)
            .addMethod(loadIntoMethodOfGroupBuilder.build())
            .build()).build().writeTo(mFiler);

    return true;
}
```

最终生成的路由表如下：

```
/**
 * DO NOT EDIT THIS FILE!!! IT WAS GENERATED BY KAOLA PROCESSOR. */
public class RouterGenerator implements RouterProvider {
  @Override
  public void loadRouter(Map<String, Route> routerMap) {
    String keyActivityDetailActivity="(" + "(https|http|kaola|native)://(pre\\.)?(\\w+\\.)?kaola\\.com(\\.hk)?/activity/spring/\\w+" + ")|("+RouteBuilder.generateUriFromClazz(ActivityDetailActivity.class)+")";
    routerMap.put(keyActivityDetailActivity, RouteBuilder.build(keyActivityDetailActivity, 0, ActivityDetailActivity.class));
    String keyLabelDetailActivity="(" + "(https|http|kaola|native)://(pre\\.)?(\\w+\\.)?kaola\\.com(\\.hk)?/album/tag/share\\.html" + ")|("+RouteBuilder.generateUriFromClazz(LabelDetailActivity.class)+")";
    routerMap.put(keyLabelDetailActivity, RouteBuilder.build(keyLabelDetailActivity, 0, LabelDetailActivity.class));
    String keyMyQuestionAndAnswerActivity="(" + "(https|http|kaola|native)://(pre\\.)?(\\w+\\.)?kaola\\.com(\\.hk)?/app/myQuestion.html" + ")|("+RouteBuilder.generateUriFromClazz(MyQuestionAndAnswerActivity.class)+")";
    routerMap.put(keyMyQuestionAndAnswerActivity, RouteBuilder.build(keyMyQuestionAndAnswerActivity, 0, MyQuestionAndAnswerActivity.class));
	……
}
```

其中，`RouteBuilder.generateUriFromClazz(Class)`的实现如下，目的是生成一条默认的与标准类名相关的native跳转规则。

```
public static final String SCHEME_NATIVE = "native://";
public static String generateUriFromClazz(Class<?> destination) {
    String rawUri = SCHEME_NATIVE + destination.getCanonicalName();
    return rawUri.replaceAll("\\.", "\\\\.");
}
```

可以看到，路由集合的key是一条正则表达式，包括了url拦截规则以及自定义的包含标准类名的native跳转规则。例如，`keyMyQuestionAndAnswerActivity`最终生成的key是

	((https|http|kaola|native)://(pre\\.)?(\\w+\\.)?kaola\\.com(\\.hk)?/app/
	myQuestion.html)|(native://com.kaola.modules.answer.myAnswer.
	MyQuestionAndAnswerActivity)

这样，调用者不仅可以通过默认的拦截规则
`(https|http|kaola|native)://(pre\\.)?(\\w+\\.)?kaola\\.com(\\.hk)?/app/myQuestion.html)`
跳转到对应的页面，也可以通过
`(native://com.kaola.modules.answer.myAnswer.MyQuestionAndAnswerActivity)`。
这样的好处是模块间的跳转也可以使用，不需要依赖引用类。而native跳转会专门生成一个类`RouterConst`来记录，如下：

```
/**
 * DO NOT EDIT THIS FILE!!! IT WAS GENERATED BY KAOLA PROCESSOR. */
public class RouterConst {
  public static final String ROUTE_TO_ActivityDetailActivity = "native://com.kaola.modules.activity.ActivityDetailActivity";
  public static final String ROUTE_TO_LabelDetailActivity = "native://com.kaola.modules.albums.label.LabelDetailActivity";
  public static final String ROUTE_TO_MyQuestionAndAnswerActivity = "native://com.kaola.modules.answer.myAnswer.MyQuestionAndAnswerActivity";
  public static final String ROUTE_TO_CertificatedNameActivity = "native://com.kaola.modules.auth.activity.CertificatedNameActivity";
  public static final String ROUTE_TO_CPSCertificationActivity = "native://com.kaola.modules.auth.activity.CPSCertificationActivity";
  public static final String ROUTE_TO_BrandDetailActivity = "native://com.kaola.modules.brands.branddetail.ui.BrandDetailActivity";
  public static final String ROUTE_TO_CartContainerActivity = "native://com.kaola.modules.cart.CartContainerActivity";
  public static final String ROUTE_TO_SingleCommentShowActivity = "native://com.kaola.modules.comment.detail.SingleCommentShowActivity";
  public static final String ROUTE_TO_CouponGoodsActivity = "native://com.kaola.modules.coupon.activity.CouponGoodsActivity";
  public static final String ROUTE_TO_CustomerAssistantActivity = "native://com.kaola.modules.customer.CustomerAssistantActivity";
  ……
}
```

### 初始化路由

路由初始化在Application的过程中以同步的方式进行。通过获取`RouterGenerator`的类直接生成实例，并将路由信息保存在`sRouterMap`变量中。

```
public static void init() {
    try {
        sRouterMap = new HashMap<>();
        ((RouterProvider) (Class.forName(ROUTER_CLASS_NAME).getConstructor().newInstance())).loadRouter(sRouterMap);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

### 页面路由

给定一个url以及上下文环境，即可使用路由。调用方式如下：

```
KaolaRouter.with(context).url(url).start();
```

页面路由分为路由请求生成，路由查找以及路由结果执行这几个步骤。路由请求目前较为简单，仅是封装了一个RouterRequest接口

```
public interface RouterRequest {
	Uri getUriRequest(); 
}
```

路由的查找过程相对复杂，除了遍历路由初始化以后导入内存的路由表，还需要判断各种各样的前置条件。具体的条件判断代码中有相关注释。

```
@Override
public RouterResponse findResponse(RouterRequest request) {
    if (null == sRouterMap) {
        return null;
        //throw new IllegalStateException(
        //        String.format("Router has not been initialized, please call %s.init() first.",
        //                KaolaRouter.class.getSimpleName()));
    }
    if (mRouterRequestWrapper.getDestinationClass() != null) {
        RouterResponse response = RouterResponseFactory.buildRouterResponse(null, mRouterRequestWrapper);
        reportFoundRequestCallback(request, response);
        return response;
    }
    Uri uri = request.getUriRequest();
    String requestUrl = uri.toString();
    if (!TextUtils.isEmpty(requestUrl)) {
        for (Map.Entry<String, Route> entry : sRouterMap.entrySet()) {
            if (RouterUtils.matchUrl(requestUrl, entry.getKey())) {
                Route routerModel = entry.getValue();
                if (null != routerModel) {
                    RouterResponse response =
                            RouterResponseFactory.buildRouterResponse(routerModel, mRouterRequestWrapper);
                    reportFoundRequestCallback(request, response);
                    return response;
                }
            }
        }
    }
    return null;
}

@Override
public RouterResult start() {
	// 判断Context引用是否还存在
    WeakReference<Context> objectWeakReference = mContextWeakReference;

    if (null == objectWeakReference) {
        reportRouterResultError(null, null, RouterError.ROUTER_CONTEXT_REFERENCE_NULL, null);
        return getRouterResult(false, mRouterRequestWrapper, null);
    }
    Context context = objectWeakReference.get();
    if (context == null) {
        reportRouterResultError(null, null, RouterError.ROUTER_CONTEXT_NULL, null);
        return getRouterResult(false, mRouterRequestWrapper, null);
    }
	// 判断路由请求是否有效
    if (!checkRequest(context)) {
        return getRouterResult(false, mRouterRequestWrapper, null);
    }
	// 遍历查找路路由结果
    RouterResponse response = findResponse(mRouterRequestWrapper);
	// 判断路由结果，执行路由结果为空时的拦截
    if (null == response) {
        boolean handledByCallback = reportLostRequestCallback(mRouterRequestWrapper);
        if (!handledByCallback) {
            reportRouterResultError(context, null, RouterError.ROUTER_RESPONSE_NULL,
                    mRouterRequestWrapper.getRouterRequest());
        }
        return getRouterResult(handledByCallback, mRouterRequestWrapper, null);
    }
	// 获取路由结果执行的接口
    ResponseInvoker responseInvoker = getResponseInvoker(context, response);

    if (responseInvoker == null) {
        return getRouterResult(false, mRouterRequestWrapper, response);
    }

    Intent intent;
    try {
        intent = RouterUtils.generateResponseIntent(context, response, mRouterRequestWrapper);
    } catch (Exception e) {
        reportRouterResultError(context, null, RouterError.ROUTER_GENERATE_INTENT_ERROR, e);
        return getRouterResult(false, mRouterRequestWrapper, response);
    }
	// 生成相应的Intent
    if (null == intent) {
        reportRouterResultError(context, null, RouterError.ROUTER_GENERATE_INTENT_NULL, response);
        return getRouterResult(false, mRouterRequestWrapper, response);
    }
	// 获取路由结果回调接口，如果为空，则使用默认提供的实现
    RouterResultCallback routerResultCallback = getRouterResultCallback();

    // 由使用者处理
    if (routerResultCallback.beforeRoute(context, intent)) {
        return getRouterResult(true, mRouterRequestWrapper, response);
    }
    try {
        responseInvoker.invoke(context, intent, mRouterRequestWrapper.getRequestCode(),
                mRouterRequestWrapper.getOnActivityResultListener());
        routerResultCallback.doRoute(context, intent, null);
        return getRouterResult(true, mRouterRequestWrapper, response);
    } catch (Exception e) {
        reportRouterResultError(context, intent, RouterError.ROUTER_INVOKER_ERROR, e);
        return getRouterResult(false, mRouterRequestWrapper, response);
    }
}
```

最终会调用`ResponseInvoker.invoke()`方法执行路由。

# 待开发

1. 职责链模式，参考OkHttp
2. 集成Fragment
3. 支持异步
4. 路由缓存
5. 路由智能优先级（调用过的，放最前面）
6. 集成权限管理
7. 考虑需要登录的情况，统一处理

# 总结

考拉路由框架与其他路由框架相比，目前功能较简单，目的也仅是支持页面跳转。为了达到对开发者友好、使用简单的目的，本文在设计路由框架的过程中使用了一些简单的设计模式，使得整个系统的可扩展性较强，也能够充分的满足考拉的业务需求。

# 参考

1. <https://github.com/alibaba/ARouter>
2. <http://www.jianshu.com/p/79e9a54e85b2>
3. <https://joyrun.github.io/2016/08/01/ActivityRouter/>
4. <http://www.jianshu.com/p/8a3eeeaf01e8>
5. <http://www.jianshu.com/p/f582c3893bed>