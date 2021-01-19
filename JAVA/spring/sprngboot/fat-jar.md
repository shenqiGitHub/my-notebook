# SpringBoot 系列-FatJar 启动原理

之前有写过一篇文章来介绍 JAR 文件和 MENIFEST.MF 文件，详见：[聊一聊 JAR 文件和 MANIFEST.MF](https://juejin.im/post/6844903876877893640)，在这篇文章中介绍了 JAR 文件的内部结构。本篇将继续延续前面的节奏，来介绍下，在 SpringBoot 中，是如何将一个 FatJar 运行起来的。

## FatJar 解压之后的文件目录

从 [Spring 官网](https://start.spring.io/) 或者通过 Idea 创建一个新的 SpringBoot 工程，方便起见，建议什么依赖都不加，默认带入的空的 SpringBoot 工程即可。

通过 maven 命令进行打包，打包成功之后得到的构建产物截图如下：



![img]()



在前面的文章中有提到，jar 包是zip 包的一种变种，因此也可以通过 unzip 来解压

```
unzip -q guides-for-jarlaunch-0.0.1-SNAPSHOT.jar -d mock
复制代码
```

解压的 mock 目录，使用 tree 指令，看到整个解压之后的 FatJar 的目录结构如下（部分省略）：

```
.
├── BOOT-INF
│   ├── classes
│   │   ├── application.properties  # 用户-配置文件
│   │   └── com
│   │       └── glmapper
│   │           └── bridge
│   │               └── boot
│   │                   └── BootStrap.class  # 用户-启动类
│   └── lib
│       ├── jakarta.annotation-api-1.3.5.jar
│       ├── jul-to-slf4j-1.7.28.jar
│       ├── log4j-xxx.jar # 表示 log4j 相关的依赖简写
│       ├── logback-xxx.jar # 表示 logback 相关的依赖简写
│       ├── slf4j-api-1.7.28.jar
│       ├── snakeyaml-1.25.jar
│       ├── spring-xxx.jar   # 表示 spring 相关的依赖简写
├── META-INF
│   ├── MANIFEST.MF
│   └── maven
│       └── com.glmapper.bridge.boot
│           └── guides-for-jarlaunch
│               ├── pom.properties
│               └── pom.xml
└── org
    └── springframework
        └── boot
            └── loader
                ├── ExecutableArchiveLauncher.class
                ├── JarLauncher.class
                ├── LaunchedURLClassLoader$UseFastConnectionExceptionsEnumeration.class
                ├── LaunchedURLClassLoader.class
                ├── Launcher.class
                ├── MainMethodRunner.class
                ├── PropertiesLauncher$1.class
                ├── PropertiesLauncher$ArchiveEntryFilter.class
                ├── PropertiesLauncher$PrefixMatchingArchiveFilter.class
                ├── PropertiesLauncher.class
                ├── WarLauncher.class
                ├── archive
                │   ├── # 省略
                ├── data
                │   ├── # 省略
                ├── jar
                │   ├── # 省略
                └── util
                    └── SystemPropertyUtils.class

复制代码
```

简单来看，FatJar 解压之后包括三个文件夹：

```
├── BOOT-INF # 存放的是业务相关的，包括业务开发的类和配置文件，以及依赖的jar
│   ├── classes
│   └── lib
├── META-INF # 包括 MANIFEST.MF 描述文件和 maven 的构建信息
│   ├── MANIFEST.MF
│   └── maven
└── org # SpringBoot 相关的类
    └── springframework
复制代码
```

我们平时在 debug SpringBoot 工程的启动流程时，一般都是从 SpringApplication#run 方法开始

```
@SpringBootApplication
public class BootStrap {
    public static void main(String[] args) {
        // 入口
        SpringApplication.run(BootStrap.class,args);
    }
}
复制代码
```

对于 java 程序来说，我们知道启动入口必须有 main 函数，这里看起来是符合条件的，但是有一点就是，通过 java 指令执行一个带有 main 函数的类时，是不需要有 -jar 参数的，比如新建一个 BootStrap.java 文件，内容为：

```
public class BootStrap {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
复制代码
```

通过 javac 编译此文件：

```
javac BootStrap.java
复制代码
```

然后就可以得到编译之后的 .class 文件 BootStrap.class ，此时可以通过 java 指令直接执行：

```
java BootStrap  # 输出 Hello World
复制代码
```

那么对于 java -jar 呢？这个其实在 [java 的官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html) 中是有明确描述的：

- -jar filename

> Executes a program encapsulated in a JAR file. The filename argument is the name of a JAR file with a manifest that contains a line in the form Main-Class:classname that defines the class with the public static void main(String[] args) method that serves as your application's starting point.

> When you use the -jar option, the specified JAR file is the source of all user classes, and other class path settings are ignored.

简单说就是，java -jar 命令引导的具体启动类必须配置在 MANIFEST.MF 资源的 Main-Class 属性中。

那回过头再去看下之前打包好、解压之后的文件目录，找到 /META-INF/MANIFEST.MF 文件，看下元数据：

```
Manifest-Version: 1.0
Implementation-Title: guides-for-jarlaunch
Implementation-Version: 0.0.1-SNAPSHOT
Start-Class: com.glmapper.bridge.boot.BootStrap
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Build-Jdk-Spec: 1.8
Spring-Boot-Version: 2.2.0.RELEASE
Created-By: Maven Archiver 3.4.0
# Main-Class 在这里，指向的是 JarLauncher
Main-Class: org.springframework.boot.loader.JarLauncher
复制代码
```

org.springframework.boot.loader.JarLauncher 类存放在 org/springframework/boot/loader 下面：

```
└── boot
    └── loader
        ├── ExecutableArchiveLauncher.class
        ├── JarLauncher.class  # JarLauncher
        ├── # 省略
复制代码
```

这样就基本理清楚了， FatJar 中，org.springframework.boot.loader 下面的类负责引导启动 SpringBoot 工程，作为入口，BOOT-INF 中存放业务代码和依赖，META-INF 下存在元数据描述。

## JarLaunch - FatJar 的启动器

在分析 JarLaunch 之前，这里插一下，org.springframework.boot.loader 下的这些类是如何被打包在 FatJar 里面的

### spring-boot-maven-plugin 打包 spring-boot-loader 过程

因为在新建的空的 SpringBoot 工程中并没有任何地方显示的引入或者编写相关的类。实际上，对于每个新建的 SpringBoot 工程，可以在其 pom.xml 文件中看到如下插件：

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
复制代码
```

这个是 SpringBoot 官方提供的用于打包 FatJar 的插件，org.springframework.boot.loader 下的类其实就是通过这个插件打进去的；

下面是此插件将 loader 相关类打入 FatJar 的一个执行流程：

> org.springframework.boot.maven#execute-> org.springframework.boot.maven#repackage -> org.springframework.boot.loader.tools.Repackager#repackage-> org.springframework.boot.loader.tools.Repackager#writeLoaderClasses-> org.springframework.boot.loader.tools.JarWriter#writeLoaderClasses

最终的执行方法就是下面这个方法，通过注释可以看出，该方法的作用就是将 spring-boot-loader 的classes 写入到 FatJar 中。

```
/**
 * Write the required spring-boot-loader classes to the JAR.
 * @throws IOException if the classes cannot be written
 */
@Override
public void writeLoaderClasses() throws IOException {
	writeLoaderClasses(NESTED_LOADER_JAR);
}
复制代码
```

### JarLaunch 基本原理

基于前面的分析，这里考虑一个问题，能否直接通过 java BootStrap 来直接运行 SpringBoot 工程呢？这样在不需要 -jar 参数和 JarLaunch 引导的情况下，直接使用最原始的 java 指令理论上是不是也可以，因为有 main 方法。

#### 通过 `java BootStrap` 方式启动

BootStrap 类的如下：

```
@SpringBootApplication
public class BootStrap {
    public static void main(String[] args) {
        SpringApplication.run(BootStrap.class,args);
    }
}
复制代码
```

编译之后，执行 `java com.glmapper.bridge.boot.BootStrap`，然后抛出异常了：

```
Exception in thread "main" java.lang.NoClassDefFoundError: org/springframework/boot/SpringApplication
        at com.glmapper.bridge.boot.BootStrap.main(BootStrap.java:13)
Caused by: java.lang.ClassNotFoundException: org.springframework.boot.SpringApplication
        at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:338)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
        ... 1 more
复制代码
```

从异常堆栈来看，是因为找不到 SpringApplication 这个类；这里其实还是比较好理解的，BootStrap 类中引入了 SpringApplication，但是这个类是在 BOOT-INF/lib 下的，而 java 指令在启动时也没有指定 class path 。

> 这里不再赘述，通过 -classpath + -Xbootclasspath 的方式尝试了下，貌似也不行，如果有通过 java 指令直接运行成功的，欢迎留言沟通。

#### 通过 `java JarLaunch 启动`

再通过 `java org.springframework.boot.loader.JarLauncher` 方式启动，可以看到是可以的。



![img]()



那这里基本可以猜到，JarLauncher 方式启动时，一定会通过某种方式将所需要依赖的 JAR 文件作为 BootStrap 的依赖引入进来。下面就来简单分析下 JarLauncher 启动时，作为启动引导类，它做了哪些事情。

#### 基本原理分析

JarLaunch 类的定义如下：

```
public class JarLauncher extends ExecutableArchiveLauncher {
    // BOOT-INF/classes/
    static final String BOOT_INF_CLASSES = "BOOT-INF/classes/";
    // BOOT-INF/lib/
    static final String BOOT_INF_LIB = "BOOT-INF/lib/";
    // 空构造函数
    public JarLauncher() {
    }
    // 带有指定 Archive 的构造函数
    protected JarLauncher(Archive archive) {
    	super(archive);
    }
    // 是否是可嵌套的对象
    @Override
    protected boolean isNestedArchive(Archive.Entry entry) {
    	if (entry.isDirectory()) {
    		return entry.getName().equals(BOOT_INF_CLASSES);
    	}
    	return entry.getName().startsWith(BOOT_INF_LIB);
    }
    
    // main 函数
    public static void main(String[] args) throws Exception {
    	new JarLauncher().launch(args);
    }

}
复制代码
```

通过代码，我们很明显可以看到几个关键的信息点：

- `BOOT_INF_CLASSES` 和 `BOOT_INF_LIB` 两个常量对应的是前面解压之后的两个文件目录
- JarLaunch 中包含一个 main 函数，作为启动入口

但是单从 main 来看，只是构造了一个 JarLaunch 对象，然后执行其 launch 方法，并没有我们期望看到的构建所需依赖的地方。实际上这部分是在 JarLaunch 的父类 ExecutableArchiveLauncher 的构造函数中来完成的。

```
public ExecutableArchiveLauncher() {
    try {
        // 构建 archive 
    	this.archive = createArchive();
    }
    catch (Exception ex) {
    	throw new IllegalStateException(ex);
    }
}

// 构建 Archive
protected final Archive createArchive() throws Exception {
    ProtectionDomain protectionDomain = getClass().getProtectionDomain();
    CodeSource codeSource = protectionDomain.getCodeSource();
    URI location = (codeSource != null) ? codeSource.getLocation().toURI() : null;
    // 这里就是拿到当前的 classpath 
    // /Users/xxx/Documents/test/glmapper-springboot-study-guides/guides-for-jarlaunch/target/mock/
    String path = (location != null) ? location.getSchemeSpecificPart() : null;
    if (path == null) {
    	throw new IllegalStateException("Unable to determine code source archive");
    }
    File root = new File(path);
    if (!root.exists()) {
    	throw new IllegalStateException("Unable to determine code source archive from " + root);
    }
    // 构建 Archive 
    return (root.isDirectory() ? new ExplodedArchive(root) : new JarFileArchive(root));
}
复制代码
```

> PS: 关于 Archive 的概念这里由于篇幅有限，不再展开说明。

通过上面构建了一个 Archive ，然后继续执行 launch 方法：

```
protected void launch(String[] args) throws Exception {
    // 注册协议，利用了 java.net.URLStreamHandler 的扩展机制，SpringBoot
    // 扩展出了一种可以解析 jar in jar 的协议
    JarFile.registerUrlProtocolHandler();
    // 通过 classpath 来构建一个 ClassLoader
    ClassLoader classLoader = createClassLoader(getClassPathArchives());
    // launch 
    launch(args, getMainClass(), classLoader);
}
复制代码
```

下面值需要关注下 getMainClass() 方法即可，这里就是获取 MENIFEST.MF 中指定的 Start-Class ，实际上就是我们的工程里面的 BootStrap 类：

```
@Override
protected String getMainClass() throws Exception {
    // 从 archive 中拿到 Manifest
    Manifest manifest = this.archive.getManifest();
    String mainClass = null;
    if (manifest != null) {
        // 获取 Start-Class
    	mainClass = manifest.getMainAttributes().getValue("Start-Class");
    }
    if (mainClass == null) {
    	throw new IllegalStateException(
    			"No 'Start-Class' manifest entry specified in " + this);
    }
    // 返回 mainClass
    return mainClass;
}
复制代码
```

最终是通过构建了一个 MainMethodRunner 实例对象，然后通过反射的方式调用了 BootStrap 类中的 main 方法：



![img]()



## 小结

本文主要从 JarLaunch 的角度分析了下 SpringBoot 的启动方式，对常规 java 方式和 java -jar 等启动方式进行了简单的演示；同时简单阐述了下 JarLaunch 启动的基本工作原理。对于其中 构建 Archive 、自定义协议 Handler 等未做深入探究，后面也会针对相关点再做单独分析。