# 03-Nest.js核心概念-模块（Modules）的定义与使用

## 什么是模块？

在 Nest.js 中，模块（Module）是**组织代码**的基本方式。每个模块封装了一组相关的功能，比如**控制器（Controllers）、服务（Services）、中间件（Middleware）、提供者（Providers）**等。模块的核心作用是帮助开发者管理应用的不同部分，使得代码更加模块化、易于维护和扩展。

模块是通过 `@Module()` 装饰器定义的，这个装饰器接受一个 **元数据对象**，用来组织控制器、服务、导入的模块等内容。

##  模块的定义

每个模块都是一个类，使用 `@Module()` 装饰器来装饰。我们来看一个简单的模块定义：

每个模块都是一个类，使用 `@Module()` 装饰器来装饰。我们来看一个简单的模块定义：

```js
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { UserService } from './user.service';

@Module({
  controllers: [UserController],  // 声明此模块包含的控制器
  providers: [UserService],       // 声明提供者（服务）
})
export class UserModule {}
```

- **controllers**：这是一个数组，用来声明哪些控制器属于这个模块。控制器是处理 HTTP 请求的类。

- **providers**：这是一个数组，用来声明哪些服务或提供者属于这个模块。服务是处理业务逻辑的类。

## 使用模块

为了在整个应用中使用模块，我们需要将模块导入到应用的根模块中。根模块是 Nest.js 应用的入口点，通常是 `app.module.ts` 文件。

我们可以通过 `imports` 属性，**将其他模块导入到根模块中**。例如：

```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UserModule } from './user/user.module';  // 导入 UserModule

@Module({
  imports: [UserModule],  // 在应用中使用 UserModule
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

在这个例子中，`AppModule` 是应用的根模块，它导入了 `UserModule`，因此 `UserModule` 的所有功能都可以在应用中使用。

##  模块的依赖注入

Nest.js 模块可以导入其他模块，建立模块之间的依赖关系。通过导入其他模块，一个模块可以共享其服务、控制器或其他功能。

例如，如果我们有一个 `AuthModule`，它包含身份验证逻辑，而其他模块（比如 `UserModule`）需要使用身份验证功能，我们可以在 `UserModule` 中导入 `AuthModule`：

```ts
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import { AuthModule } from '../auth/auth.module';  // 导入 AuthModule

@Module({
  imports: [AuthModule],  // 使用 AuthModule 的功能
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}
```

