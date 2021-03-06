# Java Dynamic Load Jar

JAVA类加载机制
- 全盘负责：一个ClassLoader装载一个类时，除非显示地使用另一个ClassLoader，该类所依赖及引用的类也由这个ClassLoader载入
- 双亲委派：子类加载器如果没有加载过该目标类，就先委托父类加载器加载该目标类，只有在父类加载器找不到字节码文件的情况下才从自己的类路径中查找并装载目标类
   - 避免重复加载
   - 安全

## Options

- OSGi
   - Pandora
   - SOFAArk
- ClassLoader
- Java Agent Instruments
- Dynamic compiling and loading
- Java embed script lang, such as Groovy

## OSGi

从一开始发布OSGI，引以为傲的两大特点：模块化(Bundle)与热部署（动态化）。

使用Bundle做模块化管理，OSGi做的很成熟。但是热部署并没有想象的那么好用。

### Bundle

OSGi的核心组件Bundle，与java中jar包的差别就是元数据配置。

### OSGI组件的热部署

一般Java的热部署（在不重启JVM的情况下替换class文件）只能适用于方法体的修改。

如果是更大的类结构的修改则需要自定义类加载器。

即使你采用OSGi，但也不代表你的应用就具备了hot deployment的能力。

在hot deployment上，完美的结果就是当更新完成后，新的执行请求就在新的代码逻辑上正确的执行，就像没发生过更新这回事样，但实际要做到这样的效果，远没这么容易，即使是基于OSGi也同样如此。

仅仅借助OSGi的动态化机制，其实是不足以实现真正的热部署的，这里的一个原因是通常代码里是带状态信息的，或者说一些全局变量信息，而OSGi的替换其实主要是通过创建新对象实例，然后替换引用的方式来实现，这也就意味着对于有状态信息的，得自己处理好状态的保存以及还原，否则是会有问题的，我而要做到这一点系统变得超级复杂。

另外还有个更麻烦的是，如果应用是OSGi和非OSGi混用，又要做动态化，那就得让非OSGi拿到的只是一个OSGi里对象的一个假的引用，以便随时替换，这个改造起来就更麻烦了。

并且要做到真正的完全动态是不行的，也就意味着动态化这特性基本就是个玩具，如果是为了隔离可以自己做一个简单的ClassLoader隔离机制。

[reference](https://blog.csdn.net/huakai_sun/article/details/78112493)

## Pandora

- 最初，HSF 1.X为了解决与应用的jar冲突问题，使用OSGi来做隔离
   - 当时淘系大部分的应用都运行在JBoss中，.sar 作为JBoss支持的一种部署格式
- HSF 2.X起，隔离的功能被独立地交付给Pandora
   - 这时候的“隔离”不再是“HSF与应用的隔离”，而是“中间件与应用的隔离”以及“中间件之间的隔离”
      - 中间件包括：HSF、Notify、MetaQ、Diamond、Tair等
      - 这些中间件被称为Pandora Plugin Module，每个中间件有独立的 ModuleClassLoader
   - Pandora容器废弃了OSGi框架，只引入了它的隔离机制，重新实现ClassLoader
- PandoraBoot则将SpringBoot和Pandora进行了整合
   - convention over configuration

## SOFAArk

## ClassLoader

jar变化后，使用新的`ClassLoader`统一加载，原有的unreacheable classes and ClassLoader will be GC'ed, with uncertain when.

风险：
- java.lang.ClassNotFoundException, NoClassDefFoundError, NoSuchMethodError, ClassFormatError, ClassCastException, AbstractMethodError
- Germ区变大隐患

## Java Agent Instruments

不支持增加方法，不支持method signature change，不支持增加类。

即，只支持方法体的修改：refineClass

## Dynamic compiling and loading

## Java embed script lang, such as Groovy

## DDDplus Plugin Hot Reloading

### 目标

以jar包为单位管理扩展业务，扩展业务包，此处称为Plugin Jar。

Plugin有2种：
- `Pattern` + `Extension`
- `Partner` + `Extension`

### Checklist before release

- [X] useSpring = false case
- [X] stress test to detect memory leakage: off heap memory/metaspace
- [X] 先静态加载plugin，然后reload，确保生效
- [X] Spring lazy load
