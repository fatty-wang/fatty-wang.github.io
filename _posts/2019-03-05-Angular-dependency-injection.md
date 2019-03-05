---
layout: post
title: "Angular中的依赖注入 "
date: 2019-03-05
author: "WangX"
catalog: true
header-style: text
tags:
    - Angular
    - DI
---

## 依赖注入相关概念

>    In software engineering, dependency injection is a technique whereby one object (or static method) supplies the dependencies of another object. A dependency is an object that can be used (a service). An injection is the passing of a dependency to a dependent object (a client) that would use it. The service is made part of the client's state.Passing the service to the client, rather than allowing a client to build or find the service, is the fundamental requirement of the pattern.                      ----wikipedia

* 依赖注入意在解耦：使得客户端的代码不必跟着它依赖对象的改变而改变。
* 依赖注入是控制反转（概念更加宽泛的技术）的一种形式。客户端将提供其依赖的职责委托给外部代码（注入器）。客户端不允许直接调用注入代码，是注入代码来构建服务，并注入给客户端。也就是说，客户端代码不需要了解注入代码是如何构建服务，甚至不知道哪个真实的服务注入代码正在使用。客户端只需要知道它依赖的那些服务的固有接口，从而调用。这样就把使用和构造的责任分离开来。
* 依赖注入涉及4个角色：
1. 被使用的**服务**对象
2. 依赖的服务的**客户端**对象
3. 定义客户端怎样使用服务的**接口**
4. 构造服务并把服务注入客户端的**注入器**


## Angular中的依赖注入

#### 概述

!["DI框架构成"](/img/angular2/di-in-angular2-5.svg "DI框架构成")
* Angular中的依赖注入基本上由三大部分构成：
1. Injector: 向用户公开api以创建依赖项实例的注入器对象。
2. Provider: 依赖提供商告诉注入器如何创建一个依赖的的实例。依赖提供商获取token并将其映射到创建对象的工厂函数。
3. Dependency: 依赖就是将要被创建的可注入对象。

#### 可注入服务(Dependency)

>服务在angular中是一个广义的概念，它包括应用所需的任何值、函数或特性。狭义的服务是一个明确定义了用途的类。
* @Injectable() 装饰器把类标记为可供注入的服务。
* @Injectable() 装饰器具有一个名叫 providedIn 的元数据选项，在那里可以指定把被装饰类的提供商放到 root 注入器中，或某个特定 NgModule 的注入器中。
* 当 Angular 创建一个构造函数中有参数的类时，它会查找有关这些参数的类型，和供注入使用的元数据，以便找到正确的服务。 如果 Angular 无法找到参数信息，它就会抛出一个错误。 **只有当类具有某种装饰器时，Angular 才能找到参数信息。** @Injectable() 装饰器是所有服务类的标准装饰器。

#### 注入器(Injector)

* 注入器负责创建服务实例，并把它们注入到客户端的类中。很少需要自己创建 Angular 的注入器。Angular 会在执行应用时为你创建注入器，第一个注入器是根注入器，创建于启动过程中。
* Provider会告诉注入器如何创建该服务。 要想让注入器能够创建服务（或提供其它类型的依赖），必须使用某个Provider配置好注入器。
* 可以通过制定带有依赖类型的**构造函数**参数来要求 Angular 在组件的构造函数中注入依赖项。
* **在某个注入器的范围内，服务是单例的**。也就是说，在指定的注入器中最多只有某个服务的最多一个实例。
* 当 Angular 创建一个在 @Component() 中指定了 providers 的组件实例时，它也会为该实例创建一个新的子注入器。 类似的，当在运行期间加载一个新的 NgModule 时，Angular 也可以为它创建一个拥有自己的提供商的注入器。子模块和组件注入器彼此独立，并且会为所提供的服务分别创建自己的实例。当 Angular 销毁 NgModule 或组件实例时，也会销毁这些注入器以及注入器中的那些服务实例。
* 注入器是可继承的，这意味着如果指定的注入器无法解析某个依赖，它就会请求父注入器来解析它。 组件可以从它自己的注入器来获取服务、从其祖先组件的注入器中获取、从其父 NgModule 的注入器中获取，或从 root 注入器中获取。
* 当使用提供商配置注入器时，就会把提供商和一个DI令牌关联起来。注入器维护一个内部**令牌-提供商的映射表**，当请求一个依赖项时就会引用它。令牌就是这个映射表的键。 
* 当组件或服务声明某个依赖项时，该类的构造函数会以参数的形式接收那个依赖项。通过给这个参数加上@Optional()注解，可以告诉Angular，该依赖是可选的。

```typescript
constructor(@Optional() private logger: Logger) {
  if (this.logger) {
    this.logger.log(some_message);
  }
}
```
上述代码中，必须能正确处理 null 值。如果你没有在任何地方注册过 logger 提供商，那么注入器就会把 logger 的值设置为 null。

* Angular的依赖注入系统是多级的。 实际上，应用程序中有一个与组件树结构完全相同且一一对应的注入器树。 你可以在组件树中的任何级别上重新配置注入器。
* 可以为注入器树中不同的注入器分别配置提供商。所有运行中的应用都会共享同一个内部的平台级注入器。AppModule上的注入器是全应用级注入器树的根节点，在NgModule中，指令级的注入器会遵循组件树的结构。
* 在不同位置配置provider会带来一些差异：**最终包的大小、服务的范围和服务的生命周期**。
* 在服务自身的 @Injectable() 装饰器中指定提供商时（通常在应用的根一级），CLI 生产模式构建时所用的优化工具可以执行摇树优化，它会移除没有用过的那些服务。摇树优化生成的包会更小。
* 当使用 providedIn:'root' 时，你是在配置应用的根注入器，也就是 AppModule 的注入器。 整个注入器树的真正的根是平台注入器，它是根注入器的父节点。 这可以让多个应用共享同一套平台配置。
* NgModule 级的提供商可以在 @NgModule() providers 元数据中指定，也可以在 @Injectable() 的 providedIn 选项中指定某个模块类（但根模块 AppModule 除外）。
* 如果某个模块是惰性加载的，那么请使用 @NgModule() 的 provides 选项。加载那个模块时，就会用这里的提供商来配置模块本身的注入器，而 Angular 会为该模块中创建的任何类注入相应的服务。如果你使用了 @Injectable() 中的 providedIn: MyLazyloadModule 选项，那么如果该提供商没有在别处用过，就可以在编译期间把它摇树优化掉。
* 无论对于根级注入器还是模块级注入器，服务实例的生存期都和应用或模块本身相同。
* 组件级提供商为每个组件实例配置自己的注入器。 Angular 只能把相应的服务注入到该组件实例或其下级组件实例中，而不能把这个服务实例注入到其它地方。
* 组件提供的服务具有受限的生命周期。**组件的每个新实例都会获得自己的一份服务实例。** 当销毁组件实例时，服务实例也会被同时销毁。
* 如果在AppModule的@NgModule()元数据中配置了全应用级的提供商，就会覆盖通过@Injectable()配置的那一个。可以用这种方式来为那些供多个应用使用的服务指定非默认的提供商。
* NgModule中每个组件都有它自己的注入器。通过使用@Component元数据在组件级配置某个提供商，可以把这个提供商的范围限定到该组件及其子组件。
* 注入器本质上并不属于组件，而是 DOM 中该组件实例所附着到的元素。另一个 DOM 元素上的其它组件实例则会使用另一个注入器。组件是一种特殊的指令，@Component()中的providers属性是从@Directive()中继承来的。指令也能拥有依赖，并且你也可以在它们的@Directive()元数据中配置提供商。当你使用providers属性为组件或指令配置提供商时，该提供商属于所在DOM元素的那个注入器。同一个元素上的组件与指令共享同一个注入器。
* 当一个组件申请获得一个依赖时，Angular 先尝试用该组件自己的注入器来满足它。 如果该组件的注入器没有找到对应的提供商，它就把这个申请转给它父组件的注入器来处理。 如果那个注入器也无法满足这个申请，它就继续转给它在注入器树中的父注入器。 这个申请继续往上冒泡 —— 直到 Angular 找到一个能处理此申请的注入器或者超出了组件树中的祖先位置为止。 如果超出了组件树中的祖先还未找到，Angular 就会抛出一个错误。
* **如果你在不同的层级上为同一个DI令牌注册了提供商，那么Angular所碰到的第一个注入器就会用来提供该依赖。**