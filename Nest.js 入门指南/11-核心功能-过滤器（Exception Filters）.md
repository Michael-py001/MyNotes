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


## 异常过滤器基础概念

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

## 异常过滤器的作用范围和多重异常处理

```ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  BadRequestException,
  UnauthorizedException,
} from '@nestjs/common';
import { Request, Response } from 'express';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';
```

让我们通过一个完整的示例来理解异常过滤器的不同作用范围：

```ts
// filters/business-exception.filter.ts
@Catch(Error)
export class BusinessExceptionFilter implements ExceptionFilter {
  catch(exception: Error, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    // 判断异常类型并处理
    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = exception.message;

    if (exception instanceof HttpException) {
      status = exception.getStatus();
    } else if (exception instanceof TypeError) {
      status = HttpStatus.BAD_REQUEST;
      message = '参数类型错误';
    }

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: message,
      type: exception.constructor.name
    });
  }
}
```

现在，让我们创建一个示例控制器，展示不同作用范围的异常过滤器使用方法：

```ts
// user/user.controller.ts
import { Controller, Get, Post, Body, UseFilters, ParseIntPipe } from '@nestjs/common';
import { BusinessExceptionFilter } from '../filters/business-exception.filter';

// 1. 控制器级别的异常过滤器
@Controller('users')
@UseFilters(BusinessExceptionFilter)
export class UserController {
  
  // 2. 方法级别的异常过滤器
  @Post()
  @UseFilters(BusinessExceptionFilter)
  async createUser(@Body() createUserDto: any) {
    if (!createUserDto.email) {
      throw new BadRequestException('邮箱不能为空');
    }
    // 其他业务逻辑...
  }

  @Get(':id')
  async findOne(@Param('id', ParseIntPipe) id: number) {
    if (id <= 0) {
      throw new BadRequestException('ID必须大于0');
    }
    // 其他业务逻辑...
  }
}
```

接下来，我们实现一个全局异常过滤器，它将捕获应用程序中所有未处理的异常：

```ts
// filters/global-exception.filter.ts
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(GlobalExceptionFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = '服务器内部错误';
    let errorResponse: any = {};

    if (exception instanceof HttpException) {
      status = exception.getStatus();
      errorResponse = exception.getResponse();
      message = errorResponse.message || exception.message;
    }

    // 记录错误日志
    this.logger.error(
      `${request.method} ${request.url}`,
      exception instanceof Error ? exception.stack : '',
      'GlobalExceptionFilter'
    );

    // 构建标准化的错误响应
    const responseBody = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message: message,
      error: status >= 500 ? '服务器错误' : '客户端错误',
      details: process.env.NODE_ENV === 'development' ? errorResponse : undefined
    };

    response.status(status).json(responseBody);
  }
}
```

在主应用程序中注册全局异常过滤器：

```ts
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { GlobalExceptionFilter } from './filters/global-exception.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // 注册全局异常过滤器
  app.useGlobalFilters(new GlobalExceptionFilter());
  
  await app.listen(3000);
}
bootstrap();
```

这个完整的示例展示了异常过滤器的三个作用范围：

1. 全局范围：捕获整个应用程序的异常
2. 控制器范围：处理特定控制器中的所有异常
3. 方法范围：只处理特定路由方法的异常

异常过滤器的执行顺序是从小范围到大范围：方法级 -> 控制器级 -> 全局级。这意味着如果方法级的过滤器处理了异常，控制器级和全局级的过滤器就不会被触发。

## 内置异常与 HTTP 异常处理

首先，让我们看看需要用到的依赖包：

```ts
import {
  HttpException,
  HttpStatus,
  Injectable,
  NotFoundException,
  BadRequestException,
  ForbiddenException,
  ConflictException,
  UnauthorizedException,
  InternalServerErrorException,
} from '@nestjs/common';
```

让我们先了解 Nest.js 的内置 HttpException 类：

```ts
// examples/http-exception-demo.ts
export class HttpExceptionDemo {
  // 基础 HttpException 使用
  example1() {
    // 最基本的异常抛出方式
    throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);

    // 自定义响应对象
    throw new HttpException({
      status: HttpStatus.FORBIDDEN,
      error: 'This is a custom message',
      details: { reason: 'Permission denied' }
    }, HttpStatus.FORBIDDEN);
  }

  // 使用内置的异常类
  example2() {
    // 404 未找到资源
    throw new NotFoundException('User not found');
    
    // 400 错误请求
    throw new BadRequestException('Invalid parameters');
    
    // 401 未授权
    throw new UnauthorizedException('Please login first');
    
    // 403 禁止访问
    throw new ForbiddenException('No permission');
    
    // 409 冲突
    throw new ConflictException('Username already exists');
  }
}
```

现在，让我们创建一些自定义异常类来处理特定的业务场景：

```ts
// exceptions/business.exception.ts
export class BusinessException extends HttpException {
  constructor(
    message: string,
    status: HttpStatus = HttpStatus.BAD_REQUEST,
    private readonly errorCode?: string
  ) {
    super(
      {
        message,
        errorCode,
        timestamp: new Date().toISOString(),
        type: 'BusinessException'
      },
      status
    );
  }
}

// exceptions/validation.exception.ts
export class ValidationException extends BadRequestException {
  constructor(
    public validationErrors: Record<string, string[]>
  ) {
    super({
      message: 'Validation failed',
      errors: validationErrors,
      timestamp: new Date().toISOString()
    });
  }
}

// exceptions/database.exception.ts
export class DatabaseException extends InternalServerErrorException {
  constructor(
    operation: string,
    error: Error
  ) {
    super({
      message: `Database ${operation} operation failed`,
      details: process.env.NODE_ENV === 'development' ? error.message : undefined,
      timestamp: new Date().toISOString()
    });
  }
}
```

让我们在实际业务场景中使用这些异常：

```ts
// services/user.service.ts
@Injectable()
export class UserService {
  async createUser(userData: CreateUserDto) {
    try {
      // 验证用户数据
      if (!this.isValidEmail(userData.email)) {
        throw new ValidationException({
          email: ['Invalid email format']
        });
      }

      // 检查用户名是否已存在
      const existingUser = await this.userRepository.findByUsername(userData.username);
      if (existingUser) {
        throw new BusinessException(
          'Username already taken',
          HttpStatus.CONFLICT,
          'USER_EXISTS'
        );
      }

      // 尝试创建用户
      const result = await this.userRepository.create(userData);
      if (!result) {
        throw new DatabaseException(
          'create',
          new Error('Failed to create user record')
        );
      }

      return result;
    } catch (error) {
      // 处理其他未预期的错误
      if (!(error instanceof HttpException)) {
        throw new InternalServerErrorException(
          'An unexpected error occurred while creating user'
        );
      }
      throw error;
    }
  }
}
```

为了统一处理这些异常，我们可以创建一个增强的异常过滤器：

```ts
// filters/enhanced-exception.filter.ts
@Catch()
export class EnhancedExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(EnhancedExceptionFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    // 准备默认的错误响应结构
    let errorResponse = {
      statusCode: HttpStatus.INTERNAL_SERVER_ERROR,
      message: 'Internal server error',
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
    };

    // 根据不同的异常类型定制响应
    if (exception instanceof HttpException) {
      const status = exception.getStatus();
      const response = exception.getResponse() as any;

      errorResponse = {
        ...errorResponse,
        statusCode: status,
        message: response.message || exception.message,
        errorCode: response.errorCode,
        errors: response.errors,
        details: response.details,
      };
    } else if (exception instanceof Error) {
      errorResponse.message = exception.message;
    }

    // 记录错误日志
    this.logger.error(
      `${request.method} ${request.url}`,
      exception instanceof Error ? exception.stack : 'Unknown error',
      'EnhancedExceptionFilter'
    );

    // 根据环境移除敏感信息
    if (process.env.NODE_ENV === 'production') {
      delete errorResponse.details;
      if (errorResponse.statusCode >= 500) {
        errorResponse.message = 'Internal server error';
      }
    }

    response.status(errorResponse.statusCode).json(errorResponse);
  }
}
```

最后，让我们把这个过滤器应用到实际的控制器中：

```ts
// controllers/user.controller.ts
@Controller('users')
@UseFilters(EnhancedExceptionFilter)
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  async createUser(@Body() createUserDto: CreateUserDto) {
    try {
      return await this.userService.createUser(createUserDto);
    } catch (error) {
      // 异常会被 EnhancedExceptionFilter 捕获并处理
      throw error;
    }
  }

  @Get(':id')
  async getUser(@Param('id') id: string) {
    const user = await this.userService.findById(id);
    if (!user) {
      throw new NotFoundException(`User with id ${id} not found`);
    }
    return user;
  }
}
```

这个章节我们学习了：

1. Nest.js 内置的 HttpException 类及其子类
2. 如何创建和使用自定义异常类
3. 如何根据不同的业务场景抛出适当的异常
4. 如何创建统一的异常处理过滤器
5. 如何在生产环境中安全地处理异常信息
