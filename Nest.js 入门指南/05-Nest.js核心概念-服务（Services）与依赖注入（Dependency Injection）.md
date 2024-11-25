# 05-Nest.js核心概念-服务（Services）与依赖注入（Dependency Injection）

## 什么是服务（Service）？

**服务（Service）** 是 Nest.js 中用来**处理业务逻辑**的核心部分。控制器通常负责处理 HTTP 请求，而服务则负责具体的业务逻辑。将业务逻辑封装在服务中可以使代码更易于维护、测试和复用。

在 Nest.js 中，服务是通过类来定义的，通常被标记为 `@Injectable()`，这表明这个类可以被 Nest.js 的 **依赖注入系统** 使用。

## 创建服务

你可以通过 Nest CLI 创建一个服务。假设我们要为用户相关的操作创建一个 `UserService`，可以通过以下命令生成：

```bash
nest g service user
```

这将生成一个 `user.service.ts` 文件，默认的内容如下：

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  getAllUsers(): string[] {
    return ['John', 'Jane', 'Doe'];
  }
}
```

## 服务与控制器的配合

服务通常与控制器配合使用，控制器负责处理请求，而服务负责执行具体的业务逻辑。我们来看如何在控制器中使用服务：

```ts
import { Controller, Get } from '@nestjs/common';
import { UserService } from './user.service';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  getAllUsers(): string[] {
    return this.userService.getAllUsers();
  }
}
```

- **依赖注入（Dependency Injection）**：`UserController` 通过 **构造函数** 注入了 `UserService`。`constructor(private readonly userService: UserService)` 表示 `UserService` 的实例会自动被注入到控制器中。

- **`getAllUsers()`**：控制器的方法通过调用 `UserService` 中的 `getAllUsers()` 方法来处理业务逻辑。

## 依赖注入（Dependency Injection）

**依赖注入** 是 Nest.js 的核心特性之一。通过依赖注入，类可以轻松地获取所需的依赖，而不需要自己创建实例。依赖注入使得模块之间的解耦变得简单，并且有助于提高代码的可测试性。

依赖注入在 Nest.js 中是通过构造函数实现的。只要在构造函数中声明要注入的依赖，Nest.js 就会自动注入相应的服务实例。

**依赖注入的基本流程**：

1. **定义服务**：通过 `@Injectable()` 声明一个服务类。
2. **提供服务**：在模块中，通过 `providers` 属性声明该服务。
3. **注入服务**：在需要使用服务的类（如控制器）中，通过构造函数注入该服务。

## 服务的依赖注入示例

假设我们有一个 `AuthService` 来处理用户的认证操作。我们希望在 `UserService` 中使用 `AuthService`。可以像下面这样进行依赖注入：

1. **创建 AuthService**：

   ```js
   import { Injectable } from '@nestjs/common';
   
   @Injectable()
   export class AuthService {
     validateUser(userId: string): boolean {
       // 模拟验证用户的业务逻辑
       return userId === '123';
     }
   }
   ```

2. **在 UserService 中注入 AuthService**：

   ```ts
   import { Injectable } from '@nestjs/common';
   import { AuthService } from '../auth/auth.service';
   
   @Injectable()
   export class UserService {
     constructor(private readonly authService: AuthService) {}
   
     getUserData(userId: string): string {
       if (this.authService.validateUser(userId)) {
         return `User data for userId: ${userId}`;
       } else {
         return 'Invalid user';
       }
     }
   }
   ```

3. **在 UserModule 中提供 AuthService**：

   ```ts
   import { Module } from '@nestjs/common';
   import { UserController } from './user.controller';
   import { UserService } from './user.service';
   import { AuthService } from '../auth/auth.service';  // 导入 AuthService
   
   @Module({
     controllers: [UserController],
     providers: [UserService, AuthService],  // 将 AuthService 注册到 providers
   })
   export class UserModule {}
   ```

4. **控制器使用服务**：

   ```ts
   import { Controller, Get, Param } from '@nestjs/common';
   import { UserService } from './user.service';
   
   @Controller('users')
   export class UserController {
     constructor(private readonly userService: UserService) {}
   
     @Get(':id')
     getUserData(@Param('id') id: string): string {
       return this.userService.getUserData(id);
     }
   }
   ```

   通过这个示例，当访问 `http://localhost:3000/users/123` 时，会返回 `User data for userId: 123`，而访问其他用户 ID 时会返回 `Invalid user`。

## 依赖注入的范围

在 Nest.js 中，服务的生命周期和作用范围（Scope）是通过模块来管理的：

- **单例服务**：默认情况下，服务是 **单例模式** 的，这意味着在整个应用中只有一个实例。
- **请求作用域**：你可以通过设置服务为 **请求作用域**（Request Scope）来让每个请求都创建一个新的服务实例。
- **瞬时作用域**：瞬时服务（Transient Scope）会在每次依赖注入时创建一个新的实例。

你可以通过 `@Injectable({ scope: Scope.REQUEST })` 或 `@Injectable({ scope: Scope.TRANSIENT })` 来控制服务的作用范围。

`@Injectable({ scope: Scope.REQUEST })` 是 NestJS 中用于定义服务的作用域（scope）的装饰器。默认情况下，NestJS 中的服务是单例的，即在整个应用程序生命周期内只有一个实例。然而，有时你可能需要每个请求都有一个新的服务实例，这时就可以使用 `Scope.REQUEST`。

- 使用请求作用域（`Scope.REQUEST`）的场景包括但不限于：

  - 处理请求特定的数据

  - 避免状态共享

  - 日志记录和跟踪

  - 多租户应用

- 使用 `Scope.TRANSIENT` 的场景包括但不限于：
  - 瞬态依赖：需要依赖于一些临时对象，每次使用时都需要一个新的实例。
  - 多实例并行处理：在并行处理任务时，每个任务需要一个独立的服务实例。
  - 动态创建实例：需要动态创建服务实例，并且这些实例之间不应共享状态。
