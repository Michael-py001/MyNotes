# 02-基础入门-Nest.js的核心架构和设计理念

Nest.js 的架构是为了帮助开发者 **组织代码** 和 **管理依赖**，它主要基于 **模块化设计**、**依赖注入** 和 **面向对象编程**。这使得 Nest.js 在构建大规模的应用时，代码更加清晰、易于维护。

## 模块化设计

模块化设计是 Nest.js 最重要的核心概念之一。每个功能都可以封装在一个独立的模块中，使得代码更加组织有序、便于扩展。

- **模块（Modules）**：一个模块是由一组相关的控制器（Controllers）和提供者（Providers，如服务等）组成。模块通过 `@Module` 装饰器来定义。

**案例**：让我们创建一个简单的 **用户模块（User Module）**，包含用户控制器和用户服务。

1. **创建用户模块**： 在 Nest.js 中，使用 CLI 可以轻松生成模块、控制器和服务：

```bash
nest g module user
nest g controller user
nest g service user
```

这将生成如下结构：

```css
src/
  user/
    user.module.ts
    user.controller.ts
    user.service.ts
```

2. **编写 `user.module.ts`**： 模块文件用于组织控制器和服务。它会告诉 Nest.js 这个模块包含了哪些控制器和提供者（服务）。

   ```ts
   import { Module } from '@nestjs/common';
   import { UserController } from './user.controller';
   import { UserService } from './user.service';
   
   @Module({
     controllers: [UserController],
     providers: [UserService],
   })
   export class UserModule {}
   
   ```

   这里我们声明了 `UserController` 和 `UserService` 属于 `UserModule`。

3. **编写 `user.controller.ts`**： 控制器负责处理客户端发来的 HTTP 请求，并将请求转发给服务层。

   ```ts
   import { Controller, Get } from '@nestjs/common';
   import { UserService } from './user.service';
   
   @Controller('users')
   export class UserController {
     constructor(private readonly userService: UserService) {}
   
     @Get()
     findAll(): string {
       return this.userService.findAll();
     }
   }
   ```

   这里的 `@Controller('users')` 表示这个控制器将处理 `/users` 路径的请求。`@Get()` 处理 GET 请求。

4. **编写 `user.service.ts`**： 服务（Service）负责处理核心的业务逻辑，它通常不直接与 HTTP 请求交互，而是由控制器调用。

   ```ts
   import { Injectable } from '@nestjs/common';
   
   @Injectable()
   export class UserService {
     findAll(): string {
       return 'This action returns all users';
     }
   }
   ```

   这里的 `findAll()` 方法返回了一个简单的字符串信息，表示获取所有用户。

5. **在 `app.module.ts` 中注册模块**： 我们需要将 `UserModule` 注册到应用的根模块中，使应用知道这个模块的存在。

   ```ts
   import { Module } from '@nestjs/common';
   import { AppController } from './app.controller';
   import { AppService } from './app.service';
   import { UserModule } from './user/user.module';
   
   @Module({
     imports: [UserModule],
     controllers: [AppController],
     providers: [AppService],
   })
   export class AppModule {}
   ```

6. **运行应用**： 启动应用后，访问 `http://localhost:3000/users`，你应该会看到 `This action returns all users` 的响应。

## 依赖注入（DI）

Nest.js 的依赖注入（Dependency Injection）机制允许开发者轻松管理不同组件之间的依赖关系，避免手动创建和管理对象实例。依赖注入通过 **构造函数注入** 进行管理，即需要的依赖通过构造函数传递给组件。

### `@Injectable()` 装饰器的作用

- **标记提供者**：`@Injectable()` 装饰器将一个类标记为提供者，使其可以被 NestJS 的依赖注入系统管理。
- **依赖注入**：被标记为 `@Injectable()` 的类可以被注入到其他类的构造函数中，作为依赖项使用。

### 案例

在上面的例子中，`UserController` 依赖于 `UserService`。通过构造函数注入，Nest.js 自动将 `UserService` 提供给 `UserController`。

```ts
export class UserController {
  constructor(private readonly userService: UserService) {}
}
```

- `private readonly` 表示 `userService` 是 `UserController` 的一个私有属性，并且一旦赋值后不可更改。

- 当 `UserController` 被创建时，Nest.js 自动注入 `UserService` 的实例，而无需手动管理。

### 面向对象编程（OOP）

Nest.js 提倡使用 **面向对象编程（OOP）**，例如模块化、类、装饰器等特性，使得代码更加可读、可维护。

- **模块化封装**：每个模块负责一组相关的功能，增强了代码的结构性。
- **类与装饰器**：Nest.js 通过装饰器如 `@Controller`、`@Injectable` 等，为类增加特定的功能。

## 装饰器是什么?

装饰器是一种特殊的 TypeScript 语法，用于为类或方法添加额外的元数据。比如 `@Controller` 表明这个类是一个控制器，`@Get()` 表示这个方法处理 GET 请求。

## 总结

- **模块化**：Nest.js 通过模块化设计，将不同的功能封装在模块中，方便扩展和管理。
- **依赖注入**：通过依赖注入，Nest.js 可以自动管理组件之间的依赖关系，简化了对象创建和管理的工作。
- **OOP**：面向对象编程的使用使得代码更加有组织性，并且利用 TypeScript 的特性增强了代码的可维护性。