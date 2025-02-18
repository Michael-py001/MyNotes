# 11-核心功能-过滤器（Exception Filters）

1. 异常过滤器基础概念（核心重要）

   - 什么是异常过滤器

   - 异常过滤器的作用和应用场景

   - 内置异常类型介绍

   - 异常层次结构

2. 异常过滤器的实现方式（核心重要）

   - 创建自定义异常过滤器

   - @Catch() 装饰器的使用

   - ExceptionFilter 接口实现

   - 异常响应的格式化处理

3. 异常过滤器的作用范围（重要）

   - 控制器级别的异常过滤器

   - 方法级别的异常过滤器

   - 全局异常过滤器

   - 多个异常过滤器的执行顺序

4. 内置异常与 HTTP 异常（重要）

   - HttpException 类的使用

   - 自定义 HTTP 异常

   - 状态码和错误消息的处理

   - 异常响应的标准化

5. 高级异常处理技巧（提升部分）

   - 异步异常处理

   - 依赖注入在异常过滤器中的应用

   - 日志记录与异常监控

   - 与验证管道的集成

让我们从第一个核心概念开始，我将为您准备一个完整的示例。

首先，我们需要的基础依赖包：

```ts
// 需要安装的依赖包
import { 
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { Request, Response } from 'express';
import { Logger } from '@nestjs/common';
```

我们先创建一个基础的自定义异常过滤器示例，这将帮助您理解核心概念：

```ts
// http-exception.filter.ts
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(HttpExceptionFilter.name);

  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    // 格式化异常响应
    const responseBody = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message: exception.message || null,
      error: exception.getResponse()
    };

    // 记录异常日志
    this.logger.error(
      `${request.method} ${request.url}`,
      exception.stack,
      'HttpExceptionFilter'
    );

    // 发送响应
    response
      .status(status)
      .json(responseBody);
  }
}
```

让我们来分析这个示例的关键部分：

1. `@Catch(HttpException)` 装饰器：
   - 指定此过滤器要处理的异常类型
   - 可以处理 HttpException 及其子类的所有异常

2. `implements ExceptionFilter`：

   - 确保我们正确实现了异常过滤器接口

   - 提供类型安全和代码提示

3. 异常处理逻辑：

   - 获取请求和响应对象

   - 格式化异常信息

   - 记录错误日志

   - 返回统一的错误响应格式

使用这个异常过滤器的方法：

```ts
// cats.controller.ts
import { Controller, Get, UseFilters } from '@nestjs/common';
import { HttpExceptionFilter } from './http-exception.filter';

@Controller('cats')
@UseFilters(HttpExceptionFilter) // 控制器级别应用
export class CatsController {
  @Get()
  findAll() {
    // 模拟抛出异常
    throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```

