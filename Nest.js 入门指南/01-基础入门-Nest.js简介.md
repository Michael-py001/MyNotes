# 01-基础入门-Nest.js简介

## 什么是Nest.js

Nest.js 是一个 **构建服务器端应用** 的框架，基于 **Node.js**，并采用 **TypeScript** 编写。它是受 **Angular** 的启发而设计的，所以如果你熟悉 Angular，Nest.js 的架构和使用方法会很容易上手。

- **Node.js 是什么？** Node.js 是一个使用 **JavaScript** 进行服务器端编程的运行时环境。传统上，JavaScript主要用于浏览器端，但是Node.js允许我们在服务器上使用JavaScript。
- **Nest.js 与其他框架的区别**：
  - 相比于 **Express.js** 或 **Koa.js** 这样的轻量级框架，Nest.js 提供了更强的结构化和模块化支持。
  - Nest.js 使用 **TypeScript**，它是 JavaScript 的超集，提供了类型系统和现代 JavaScript 特性，帮助开发者编写更加健壮、可维护的代码。
  - **面向对象**、**函数式** 和 **响应式** 编程的支持，使得开发者可以根据不同的需求采用不同的编程范式。

## Nest.js 的设计理念

Nest.js 的核心设计理念是帮助开发者 **组织代码** 和 **管理依赖**，从而构建易于维护和扩展的应用程序。其架构采用模块化的设计，每个功能可以封装在独立的模块中，方便重用和管理。

- 核心设计思想：
  1. **模块化**：将不同的功能封装为独立的模块，确保代码更具组织性和可扩展性。
  2. **依赖注入（Dependency Injection, DI）**：通过 DI，Nest.js 可以自动管理服务之间的依赖关系，开发者只需要声明哪些服务需要哪些依赖即可。
  3. **MVC（Model-View-Controller）架构**：通过控制器（Controller）处理请求，服务（Service）负责业务逻辑，模块（Module）管理不同功能的组织。

## 实例讲解

为了更好地理解 Nest.js 的概念，让我们来看一个具体的例子。

1. **传统的 Node.js 应用**：假设我们用最常见的 Express.js 来实现一个简单的 HTTP 服务器。

   ```js
   const express = require('express');
   const app = express();
   
   app.get('/', (req, res) => {
     res.send('Hello, World!');
   });
   
   app.listen(3000, () => {
     console.log('Server is running on http://localhost:3000');
   });
   
   ```

   这个代码非常简单，但当我们需要处理更多的路由、更多的逻辑时，代码很容易变得混乱和难以维护。

2. **使用 Nest.js 实现相同功能**：
   首先，我们通过 Nest.js 的 CLI 创建一个新项目：

   ```bash
   nest new hello-world
   cd hello-world
   npm run start
   ```

   默认的项目已经为你创建了一个基本的结构，并包含一个 “Hello World” 控制器。

   项目结构看起来像这样：

   ```css
   src/
     app.controller.ts
     app.module.ts
     app.service.ts
     main.ts
   ```

3. **代码解释**：

   - **app.controller.ts** 是控制器，用于处理 HTTP 请求。

     ```typescript
     import { Controller, Get } from '@nestjs/common';
     import { AppService } from './app.service';
     
     @Controller()
     export class AppController {
       constructor(private readonly appService: AppService) {}
     
       @Get()
       getHello(): string {
         return this.appService.getHello();
       }
     }
     ```

   - **app.service.ts** 是服务，负责业务逻辑。在这里，业务逻辑只是简单地返回一个字符串。

     ```ts
     import { Injectable } from '@nestjs/common';
     
     @Injectable()
     export class AppService {
       getHello(): string {
         return 'Hello, World!';
       }
     }
     ```

   - **app.module.ts** 是模块，它管理项目中的所有控制器和服务，将它们组合在一起。

     ```ts
     import { Module } from '@nestjs/common';
     import { AppController } from './app.controller';
     import { AppService } from './app.service';
     
     @Module({
       imports: [],
       controllers: [AppController],
       providers: [AppService],
     })
     export class AppModule {}
     ```

   - **main.ts** 是应用的入口点，用于启动应用程序。

     ```ts
     import { NestFactory } from '@nestjs/core';
     import { AppModule } from './app.module';
     
     async function bootstrap() {
       const app = await NestFactory.create(AppModule);
       await app.listen(3000);
     }
     bootstrap();
     ```

     当你启动这个项目后，打开浏览器访问 `http://localhost:3000`，你会看到 **"Hello, World!"**。
     ![image-20240918164955748](https://s2.loli.net/2024/09/18/GVw4Ux1Xc6kesE3.png)

## 总结

- **Nest.js 主要优势**：

  - **结构化**：项目从一开始就有一个清晰的结构，每个文件和功能都分离得很清楚。

  - **模块化设计**：每个功能模块（如控制器、服务等）都是独立的，方便未来的扩展。

  - **依赖注入**：通过依赖注入机制，Nest.js 让服务之间的关系管理更加简单，特别是在大型项目中非常有用。

- **与传统 Node.js 框架对比**：

  - 如果使用 Express.js，开发者通常需要手动组织项目结构、处理依赖，而 Nest.js 提供了自动化的依赖注入和良好的项目架构。

  - Nest.js 的学习曲线相比 Express 稍陡一些，但它的模块化和扩展性在大型项目中非常有价值。