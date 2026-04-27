---
title: "Clean架构在Android多模块项目中的依赖管理"
date: 2026-04-28 08:00:00
categories: ["Android开发","Clean架构","多模块项目","依赖管理"]
tags: ["Clean架构","Android多模块项目","依赖管理","实体层","用例层","接口适配器层","框架与驱动层","单向依赖","版本管理"]
---

# 🧹Clean架构在Android多模块项目中的依赖管理详细教程

想象一下，你正在建造一座大型的商业大厦，大厦由不同的功能区域组成，如商场、写字楼、酒店等。为了让各个区域能够独立运作又相互协作，就需要合理地规划它们之间的连接和依赖关系。在Android多模块项目中，Clean架构就像是这座大厦的设计蓝图，而依赖管理则是确保各个模块之间高效协作的关键“桥梁”。在这篇教程中，我们将从头到尾学习如何在Android多模块项目中运用Clean架构进行依赖管理。

## 1. 理解Clean架构和多模块项目
### 1.1 Clean架构概述
Clean架构是一种软件设计模式，它的核心思想是将项目划分为不同的层次，每个层次有明确的职责，并且依赖关系是单向的，从外层指向内层。就好比一座城堡，最外层是城墙，负责抵御外界的攻击；中间层是士兵的营房，负责执行具体的任务；最内层是国王的宫殿，存储着最重要的信息。在Clean架构中，最内层是实体层（Entities），包含业务的核心数据和规则；往外依次是用例层（Use Cases）、接口适配器层（Interface Adapters）和框架与驱动层（Frameworks & Drivers）。

### 1.2 Android多模块项目
Android多模块项目就像是一个大型的拼图，每个模块都是拼图的一部分，它们可以独立开发、测试和维护。常见的模块类型有功能模块、基础模块等。例如，一个电商App可能有商品展示模块、购物车模块、用户登录模块等。

## 2. 创建Android多模块项目
### 2.1 初始化项目
打开Android Studio，选择 `Start a new Android Studio project`，按照向导创建一个新的项目。

### 2.2 添加模块
在项目上右键，选择 `New` -> `Module`，可以选择不同类型的模块，如 `Android Library` 或 `Java Library`。这里我们以创建一个 `Android Library` 模块为例，输入模块名称，点击 `Finish`。

### 2.3 项目结构
创建完成后，项目结构会像这样：
```plaintext
MyApp
├── app
└── feature-module
```
`app` 是主应用模块，`feature-module` 是我们新创建的功能模块。

## 3. 运用Clean架构划分模块
### 3.1 实体层模块
创建一个新的 `Java Library` 模块，命名为 `entities`。这个模块只包含业务的核心数据类和规则。例如：
```java
// entities模块中的User类
package com.example.entities;

// 用户实体类，包含用户的基本信息
public class User {
    private String name;
    private int age;

    // 构造函数，用于初始化用户信息
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 获取用户姓名的方法
    public String getName() {
        return name;
    }

    // 获取用户年龄的方法
    public int getAge() {
        return age;
    }
}
```

### 3.2 用例层模块
创建一个新的 `Java Library` 模块，命名为 `use-cases`。这个模块依赖于 `entities` 模块，负责处理业务逻辑。例如：
```java
// use-cases模块中的UserUseCase类
package com.example.usecases;

import com.example.entities.User;

// 用户用例类，负责处理与用户相关的业务逻辑
public class UserUseCase {
    // 根据用户年龄判断是否成年
    public boolean isAdult(User user) {
        return user.getAge() >= 18;
    }
}
```
在 `use-cases` 模块的 `build.gradle` 文件中添加对 `entities` 模块的依赖：
```groovy
dependencies {
    implementation project(':entities')
}
```

### 3.3 接口适配器层模块
创建一个新的 `Android Library` 模块，命名为 `interface-adapters`。这个模块依赖于 `use-cases` 模块，负责将业务逻辑与用户界面或外部系统进行交互。例如：
```java
// interface-adapters模块中的UserPresenter类
package com.example.interfaceadapters;

import com.example.entities.User;
import com.example.usecases.UserUseCase;

// 用户展示器类，负责将业务逻辑与用户界面进行交互
public class UserPresenter {
    private UserUseCase userUseCase;

    // 构造函数，初始化用户用例
    public UserPresenter(UserUseCase userUseCase) {
        this.userUseCase = userUseCase;
    }

    // 判断用户是否成年并返回相应的提示信息
    public String showUserAdultStatus(User user) {
        if (userUseCase.isAdult(user)) {
            return user.getName() + " is an adult.";
        } else {
            return user.getName() + " is not an adult.";
        }
    }
}
```
在 `interface-adapters` 模块的 `build.gradle` 文件中添加对 `use-cases` 模块的依赖：
```groovy
dependencies {
    implementation project(':use-cases')
}
```

### 3.4 框架与驱动层模块
主应用模块 `app` 可以看作是框架与驱动层模块，它依赖于 `interface-adapters` 模块，负责启动应用和展示用户界面。在 `app` 模块的 `build.gradle` 文件中添加对 `interface-adapters` 模块的依赖：
```groovy
dependencies {
    implementation project(':interface-adapters')
}
```

## 4. 依赖管理的注意事项
### 4.1 单向依赖
Clean架构要求依赖关系是单向的，即外层模块可以依赖内层模块，但内层模块不能依赖外层模块。例如，`app` 模块可以依赖 `interface-adapters` 模块，但 `interface-adapters` 模块不能依赖 `app` 模块。如果违反了这个规则，就会破坏架构的独立性和可维护性。

### 4.2 版本管理
在添加依赖时，要注意依赖库的版本管理。可以使用 `ext` 变量在项目的 `build.gradle` 文件中统一管理依赖库的版本。例如：
```groovy
// 项目根目录下的build.gradle文件
buildscript {
    ext {
        // 定义依赖库的版本
        appCompatVersion = '1.3.1'
    }
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.0.4'
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
然后在各个模块的 `build.gradle` 文件中使用这些变量：
```groovy
// app模块的build.gradle文件
dependencies {
    implementation "androidx.appcompat:appcompat:$appCompatVersion"
}
```

### 4.3 常见错误及解决办法
- **依赖冲突**：当不同模块依赖同一个库的不同版本时，会出现依赖冲突。可以使用 `resolutionStrategy` 来解决冲突，例如：
```groovy
configurations.all {
    resolutionStrategy {
        force 'com.example:library:1.0.0'
    }
}
```
- **找不到依赖模块**：检查模块名称和路径是否正确，以及 `settings.gradle` 文件中是否包含了该模块。

## 小结
在这篇教程中，我们学习了Clean架构和Android多模块项目的基本概念，通过实际操作创建了一个运用Clean架构的Android多模块项目，并进行了依赖管理。核心要点包括：
- **Clean架构**：将项目划分为不同层次，单向依赖，提高项目的可维护性和可测试性。
- **多模块项目**：将项目拆分为多个独立的模块，方便开发和维护。
- **依赖管理**：遵循单向依赖原则，统一管理依赖库的版本，解决依赖冲突。

补充资源链接：
- [Clean Architecture官方文档](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Android多模块开发教程](https://developer.android.com/topic/libraries/architecture/multi-module)

## 下一步建议
- 深入学习Clean架构的高级应用，如依赖注入框架（Dagger、Hilt）的使用。
- 学习如何在多模块项目中进行单元测试和集成测试。
- 实践更多的多模块项目，提高自己的开发能力。可以参考一些开源的多模块项目，如 [Google Android Architecture Samples](https://github.com/android/architecture-samples)。