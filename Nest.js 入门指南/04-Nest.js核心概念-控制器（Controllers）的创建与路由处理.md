# 04-Nest.js核心概念-控制器（Controllers）的创建与路由处理

## 什么是控制器？

**控制器（Controller）** 是 Nest.js 负责处理 **HTTP 请求** 的核心组件。控制器的作用是：

- **接收客户端发来的请求**（GET、POST、PUT、DELETE 等）。
- **调用服务（Service）** 处理业务逻辑。
- **返回响应**（比如 JSON、HTML、文本等）给客户端。

Nest.js 控制器是通过 **装饰器（Decorators）** 来定义路由的。常见的装饰器包括 `@Controller()`、`@Get()`、`@Post()` 等。

## 创建控制器

在 Nest.js 中，我们可以通过 Nest CLI 快速创建控制器。假设我们需要创建一个 `UserController` 来处理用户相关的请求，CLI 命令如下：

在 Nest.js 中，我们可以通过 Nest CLI 快速创建控制器。假设我们需要创建一个 `UserController` 来处理用户相关的请求，CLI 命令如下：

```bash
nest g controller user
```

这将生成一个 `user.controller.ts` 文件，文件结构如下：

```ts
import { Controller, Get } from '@nestjs/common';

@Controller('users')
export class UserController {
  @Get()
  findAll(): string {
    return 'This action returns all users';
  }
}
```

- `@Controller('users')`：将这个控制器与 `/users` 路径关联，意味着这个控制器将处理所有 `/users` 路径的请求。

- `@Get()`：定义一个 GET 请求的处理器。每当客户端发起一个 GET 请求到 `/users`，这个方法 `findAll()` 会被调用。

## 路由处理

Nest.js 中，**路由** 是通过控制器内的方法和相应的 HTTP 方法装饰器（如 `@Get()`、`@Post()` 等）来处理的。

### GET 请求

`@Get()` 用于处理客户端发来的 GET 请求。它可以与控制器的某个方法绑定，以响应该方法返回的结果。

例如，定义一个获取所有用户的 GET 请求：

```ts
@Controller('users')
export class UserController {
  @Get()
  findAll(): string {
    return 'This action returns all users';
  }
}
```

访问 `http://localhost:3000/users` 时，将返回 `This action returns all users`。

### POST 请求

`@Post()` 用于处理 POST 请求，通常用于创建新资源。

例如，定义一个创建用户的 POST 请求：

```ts
@Controller('users')
export class UserController {
  @Post()
  create(): string {
    return 'This action creates a new user';
  }
}
```

当客户端向 `/users` 发送 POST 请求时，将返回 `This action creates a new user`。

### 路由参数

有时，我们需要通过 URL 路径获取一些参数。Nest.js 提供了 `@Param()` 装饰器，用于从请求路径中提取参数。

例如，获取特定用户的 ID：

```ts
@Controller('users')
export class UserController {
  @Get(':id')
  findOne(@Param('id') id: string): string {
    return `This action returns a user with id ${id}`;
  }
}
```

当你访问 `http://localhost:3000/users/123` 时，将返回 `This action returns a user with id 123`。

### 请求体（Body）

对于 POST 请求，我们通常需要接收请求体中的数据。Nest.js 提供了 `@Body()` 装饰器，用来获取请求体中的数据。

例如，处理创建用户的 POST 请求：

```ts
import { Body, Controller, Post } from '@nestjs/common';

@Controller('users')
export class UserController {
  @Post()
  create(@Body() createUserDto: any): string {
    return `This action creates a new user with the following data: ${JSON.stringify(createUserDto)}`;
  }
}
```

当客户端向 `/users` 发送 POST 请求，并包含如下 JSON 数据时：

```json
{
  "name": "John",
  "age": 30
}
```

将返回 `This action creates a new user with the following data: {"name":"John","age":30}`。

## 路由处理的示例

我们通过一个完整的示例来展示控制器的创建和路由处理：

1. **控制器的定义**：

   ```ts
   import { Controller, Get, Post, Body, Param } from '@nestjs/common';
   
   @Controller('users')
   export class UserController {
     private users = [];
   
     @Get()
     findAll(): any[] {
       return this.users;
     }
   
     @Post()
     create(@Body() user: any): string {
       this.users.push(user);
       return `User with name ${user.name} has been created`;
     }
   
     @Get(':id')
     findOne(@Param('id') id: number): any {
       return this.users[id];
     }
   }
   ```

2. **功能解释**：

   - `findAll()`：处理 GET 请求，返回所有用户的数据。

   - `create()`：处理 POST 请求，接收用户数据并添加到 `users` 数组中。

   - `findOne()`：处理 GET 请求，接收用户 ID 并返回对应的用户。

3. **模拟请求**：

   - 访问 `http://localhost:3000/users` 返回所有用户。

   - 向 `/users` 发送 POST 请求，包含用户数据，将新用户添加到用户列表。

   - 访问 `http://localhost:3000/users/0` 将返回 ID 为 0 的用户数据。