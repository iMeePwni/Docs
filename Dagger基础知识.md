# Dagger 基础知识

> https://developer.android.google.cn/training/dependency-injection/dagger-basics

## 使用Dagger的优势

无需再编写冗长乏味又容易出错的样板代码。

## Dagger组件

@Component 会让Dagger生成一个容器，其中应包含满足其提供的类型所需的所有依赖项。

可以使用作用域注释将某个对象的生命周期限定为其组件的生命周期。

# 在Android应用中使用Dagger

## 最佳做法

- 如果可能，请通过@Inject进行构造函数注入，以向Dagger图中添加类型，如果没有可能，请执行以下操作：
    - 使用@Bind告知Dagger接口应采用哪种实现。
    - 使用@Provides告知Dagger如何提供您的项目不具备的类。
- 您只能在组件中声明一次模块。
- 根据注释的生命周期，为作用域命名。示例包括@ApplicationScope、@LggerUserScope 和 @ActivityScope。

## Android 中的Dagger
![依赖关系](https://developer.android.google.cn/images/training/dependency-injection/4-application-graph.png)

在Android中，你通常会创建一个位于应用类中的Dagger图,因为只要应用处于运行状态，你就希望该图的实例在内存中。这样，就将该图附加到应用生命周期。在某些情况下，你可能还希望在该图中提供应用上下文。为此，你还需要使该图位于应用类中。这种方法的一个优势在于，该图可供其他Android框架类使用。此外，它还用需您在测试中使用自定义应用类，从而简化了测试。

有关Dagger的一个注意事项是，注入的字段不能为私有字段。这些字段的公开范围必须至少为软件包私有。（字段注入只能在无法使用构造函数注入的Android框架类中使用）。

## 注入Activity

## Dagger模块

除了@Inject注释之外，还有一种方法可告知Dagger如何提供类实例，即使用Dagger模块中的信息。Dagger模块是一个带有@Module注释的类。您可以在其中使用@Provides注释定义依赖项。

```
// @Module informs Dagger that this class is a Dagger Module
@Module
class NetworkModule {

    // @Provides tell Dagger how to create instances of the type that this function
    // returns (i.e. LoginRetrofitService).
    // Function parameters are the dependencies of this type.
    @Provides
    fun provideLoginRetrofitService(): LoginRetrofitService {
        // Whenever Dagger needs to provide an instance of type LoginRetrofitService,
        // this code (the one inside the @Provides method) is run.
        return Retrofit.Builder()
                .baseUrl("https://example.com")
                .build()
                .create(LoginService::class.java)
    }
}
```

> 注意：您可以在Dagger模块中使用@Provides注释告知Dagger如何提供您的项目所不具备的类（例如，Retrofit的实例）。对于接口的实现，最佳做法是使用@Binds。

模块是一种以语义方式封装有关如何提供对象信息的方法。
@Providres方法的依赖项是该方法的参数。

如需将类型添加到Dagger图，建议的方法是使用构造函数注入（即在类的构造函数上使用@Inject注释）。有时，此方法不可行，你必须使用Dagger模块。例如，你希望Dagger使用计算结果确定如何创建对象实例时。每当必须提供该类型的实例时，Dagger就会运行@Provides方法中的代码。

## Dagger作用域

作用域可以用来在组建中指定某个类型的唯一实例。这就是将类型的作用域限定为组件生命周期的意义所在。

@Singleton是javax.inject软件包随附的唯一一个作用域注释。你可以使用它为在整个应用中重复使用的对象添加注释。

注意：使用构造函数注入（通过@Inject）时，应在类中添加作用域注释;使用Dagger模块时，应在@Provides方法中添加作用域注释。

## Dagger子组件

如需告知Dagger您希望新组件使用其他组件的一部分，方法是使用Dagger子组件。新组件必须是包含共享资源的组件的子组件。

子组件是继承并扩展父组件的对象图的组件。因此，父组件中提供的所有对象也将在子组件中提供。这样，子组件中的对象就可以依赖父组件提供的对象。

如需创建子组件的实例，你需用父组件的实例。因此父组件