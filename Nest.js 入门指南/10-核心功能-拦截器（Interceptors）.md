# 10-核心功能-拦截器（Interceptors）

## 拦截器的基本概念和作用

拦截器（Interceptors）是 Nest.js 中一个强大的功能，它基于面向切面编程（AOP）的概念。想象一下，当一个请求进入你的应用程序时，它就像一个信件要经过多个处理环节。拦截器就像是这个过程中的检查站，可以在请求到达目的地之前和之后进行各种处理。

### 工作原理

让我们通过一个简单的例子来理解拦截器的工作原理：

```ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const method = request.method;
    const url = request.url;
    
    console.log(`开始请求: ${method} ${url}`);
    const now = Date.now();

    return next
      .handle()
      .pipe(
        tap(() => {
          console.log(`结束请求: ${method} ${url} - 耗时: ${Date.now() - now}ms`);
        }),
      );
  }
}
```

这个例子展示了一个日志拦截器的实现。让我为你解析其中的关键部分：

1. `implements NestInterceptor`：表明这个类实现了拦截器接口
2. `intercept` 方法：拦截器的核心方法，接收两个参数：
   - `context`：执行上下文，包含请求和响应信息
   - `next`：调用链中的下一个处理器

### 拦截器的生命周期

拦截器在请求-响应周期中的工作过程是这样的：

1. 请求进入应用程序
2. 经过中间件（Middleware）
3. 到达守卫（Guards）
4. 进入拦截器的 pre-controller 阶段
5. 到达管道（Pipes）
6. 控制器处理请求
7. 进入拦截器的 post-controller 阶段
8. 发送响应

### 与其他功能的区别

让我们来看看拦截器与其他 Nest.js 功能的主要区别：

```ts
// 中间件 - 处理原始请求
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: Function) {
    console.log('中间件处理请求');
    next();
  }
}

// 守卫 - 权限验证
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    return true; // 进行权限验证
  }
}

// 拦截器 - 转换响应数据
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({ data, statusCode: 200 }))
    );
  }
}
```

主要区别在于：

- 中间件：主要处理底层的 HTTP 请求和响应
- 守卫：专注于权限和认证逻辑
- 拦截器：可以转换请求和响应数据，添加额外的逻辑

### 实际应用案例

让我们来看一个更实用的例子 - 缓存拦截器：

```ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable, of } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  private cache = new Map();

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const cacheKey = request.url;

    // 检查缓存中是否有数据
    if (this.cache.has(cacheKey)) {
      console.log(`从缓存返回数据: ${cacheKey}`);
      return of(this.cache.get(cacheKey));
    }

    // 如果没有缓存，则处理请求并缓存结果
    return next.handle().pipe(
      tap(data => {
        console.log(`缓存数据: ${cacheKey}`);
        this.cache.set(cacheKey, data);
      }),
    );
  }
}
```

使用这个拦截器：

```ts
@Controller('products')
@UseInterceptors(CacheInterceptor)
export class ProductsController {
  @Get()
  findAll() {
    // 这个方法的结果会被缓存
    return this.productsService.findAll();
  }
}
```

这个例子展示了拦截器如何：

1. 在请求到达控制器之前检查缓存
2. 如果有缓存，直接返回缓存数据
3. 如果没有缓存，则处理请求并缓存结果

你能看到拦截器的强大之处了吗？它可以在不修改控制器代码的情况下，为应用添加额外的功能。

## 拦截器的实现方式

### 拦截器的基本结构

让我们首先了解拦截器的标准实现模式：

```ts
@Injectable()
export class MyInterceptor implements NestInterceptor {
  // 拦截器的核心方法
  intercept(
    context: ExecutionContext, 
    next: CallHandler
  ): Observable<any> {
    // 在请求处理之前执行的逻辑
    const pre = this.beforeRequest(context);

    // 处理请求并返回响应流
    return next.handle().pipe(
      // 在响应返回之后执行的逻辑
      map(data => this.afterRequest(data))
    );
  }

  // 请求前处理
  private beforeRequest(context: ExecutionContext) {
    const request = context.switchToHttp().getRequest();
    // 可以进行一些预处理操作
    console.log('请求上下文:', request.method, request.url);
  }

  // 响应后处理
  private afterRequest(data: any) {
    // 可以对响应数据进行转换
    return { 
      data, 
      timestamp: new Date().toISOString() 
    };
  }
}
```

### 深入理解拦截器的关键要素

#### 1. 装饰器和依赖注入

- `@Injectable()` 使拦截器成为一个可注入的提供者

- 可以在构造函数中注入其他服务

```ts
@Injectable()
export class ComplexInterceptor implements NestInterceptor {
  constructor(
    private readonly logService: LogService,
    private readonly configService: ConfigService
  ) {}

  intercept(context: ExecutionContext, next: CallHandler) {
    // 可以使用注入的服务
    this.logService.log('拦截器开始工作');
    
    return next.handle().pipe(
      tap(() => {
        const config = this.configService.get('interceptorSettings');
        // 使用配置信息
      })
    );
  }
}
```

#### 2. RxJS 操作符的高级应用

拦截器最强大的地方在于使用 RxJS 操作符进行响应处理

```ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler, HttpException, HttpStatus } from '@nestjs/common';
import { Observable, of, throwError, timer } from 'rxjs';
import { 
  map,           // 转换数据
  catchError,    // 捕获错误
  timeout,       // 设置超时
  retry,         // 重试
  mergeMap,      // 合并映射
  switchMap,     // 切换映射
  tap,           // 执行副作用
  delay,         // 延迟
  retryWhen,     // 高级重试策略
  concatMap,     // 顺序映射
  exhaustMap     // 忽略并发请求
} from 'rxjs/operators';

@Injectable()
export class AdvancedRxJSInterceptor implements NestInterceptor {
  // 高级错误处理和重试策略
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      // 1. 数据转换
      map(data => {
        // 对响应数据进行转换
        return {
          code: 200,
          data: data,
          timestamp: Date.now()
        };
      }),

      // 2. 错误捕获和处理
      catchError(error => {
        // 统一错误响应
        return throwError(() => new HttpException({
          code: error.status || 500,
          message: error.message || '系统未知错误',
          timestamp: Date.now()
        }, error.status || HttpStatus.INTERNAL_SERVER_ERROR));
      }),

      // 3. 请求超时处理
      timeout(5000),

      // 4. 重试机制
      retryWhen(errors => 
        errors.pipe(
          // 对特定类型错误进行重试
          mergeMap((error, index) => {
            // 指数退避策略
            const retryAttempt = index + 1;
            const waitTime = Math.pow(2, retryAttempt) * 1000;

            // 只对网络错误进行重试
            if (error.status >= 500 && retryAttempt <= 3) {
              return timer(waitTime);
            }
            return throwError(error);
          })
        )
      ),

      // 5. 高级数据流操作
      tap({
        next: (data) => {
          // 记录成功响应
          console.log('响应成功', data);
        },
        error: (err) => {
          // 记录错误日志
          console.error('响应失败', err);
        }
      }),

      // 6. 并发控制示例
      exhaustMap(data => {
        // 如果有多个并发请求，只处理第一个，忽略后续请求
        return of(data).pipe(
          delay(1000) // 模拟耗时操作
        );
      })
    );
  }
}

// 实际使用示例
@Controller('example')
export class ExampleController {
  constructor(
    private readonly exampleService: ExampleService
  ) {}

  @Get()
  @UseInterceptors(AdvancedRxJSInterceptor)
  async findAll() {
    return this.exampleService.findAll();
  }
}
```

#### RxJS 操作符详细解析

##### 1. `map`

- 用于转换数据流中的每一个元素
- 可以对响应数据进行结构重组、添加额外信息

##### 2. `catchError`

- 捕获并处理observable流中的错误
- 可以返回一个新的observable或抛出错误

##### 3. `timeout`

- 设置observable的超时时间
- 超过指定时间未完成则抛出错误

##### 4. `retryWhen`

- 高级重试策略
- 可以根据错误类型和重试次数动态决定是否重试
- 实现了指数退避算法，避免瞬时重试造成服务压力

##### 5. `tap`

- 执行副作用，不改变数据流
- 常用于日志记录、性能监控

##### 6. `exhaustMap`

- 处理并发请求的特殊操作符
- 忽略后续的请求，直到当前请求完成

#### 拦截器的绑定方式

**方法级别绑定**

```ts
@Controller('users')
export class UserController {
  @Get()
  @UseInterceptors(LoggingInterceptor)
  findAll() {
    // 仅对该方法生效
  }
}
```

**控制器级别绑定**

```ts
@Controller('users')
@UseInterceptors(LoggingInterceptor)
export class UserController {
  // 对控制器所有方法生效
}
```

**全局绑定**

```ts
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalInterceptors(new LoggingInterceptor());
}
```

#### 实战案例：性能监控拦截器

我们来实现一个完整的性能监控拦截器：

```ts
@Injectable()
export class PerformanceInterceptor implements NestInterceptor {
  private readonly logger = new Logger(PerformanceInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    const request = context.switchToHttp().getRequest();

    return next.handle().pipe(
      tap({
        next: (data) => {
          const duration = Date.now() - start;
          this.logger.log(`
            路径: ${request.path}
            方法: ${request.method}
            耗时: ${duration}ms
            响应数据大小: ${JSON.stringify(data).length} 字节
          `);
        },
        error: (error) => {
          const duration = Date.now() - start;
          this.logger.error(`
            请求异常
            路径: ${request.path}
            方法: ${request.method}
            耗时: ${duration}ms
            错误信息: ${error.message}
          `);
        }
      })
    );
  }
}
```

## 拦截器的作用范围和绑定方式

引入的依赖和模块：

```ts
import { 
  Injectable, 
  NestInterceptor, 
  ExecutionContext, 
  CallHandler, 
  UseInterceptors, 
  Controller, 
  Get, 
  Module 
} from '@nestjs/common';
import { 
  Observable 
} from 'rxjs';
import { 
  map 
} from 'rxjs/operators';
```

### 方法级别绑定

```ts
@Controller('users')
export class UserController {
  @Get()
  @UseInterceptors(MethodLevelInterceptor)
  findAll() {
    // 仅对该方法生效
  }
}

@Injectable()
export class MethodLevelInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        message: '方法级别拦截器处理',
        data
      }))
    );
  }
}
```

### 控制器级别绑定

```ts
@Controller('users')
@UseInterceptors(ControllerLevelInterceptor)
export class UserController {
  @Get()
  findAll() {
    // 控制器所有方法都会应用此拦截器
  }

  @Get(':id')
  findOne() {
    // 同样会应用拦截器
  }
}

@Injectable()
export class ControllerLevelInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('控制器级别拦截器');
    return next.handle();
  }
}
```

### 全局绑定

#### 在主应用中全局绑定

```ts
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { GlobalInterceptor } from './global.interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // 全局绑定拦截器
  app.useGlobalInterceptors(new GlobalInterceptor());
  
  await app.listen(3000);
}
bootstrap();
```

#### 通过模块全局绑定

```ts
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { GlobalInterceptor } from './global.interceptor';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: GlobalInterceptor
    }
  ]
})
export class AppModule {}

@Injectable()
export class GlobalInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('全局拦截器生效');
    return next.handle().pipe(
      map(data => ({
        statusCode: 200,
        globalTimestamp: Date.now(),
        data
      }))
    );
  }
}
```

#### 多拦截器执行顺序

```ts
@Controller('users')
@UseInterceptors(FirstInterceptor, SecondInterceptor)
export class UserController {
  @Get()
  @UseInterceptors(ThirdInterceptor)
  findAll() {
    // 多拦截器执行顺序演示
  }
}

// 拦截器执行顺序：
// 1. FirstInterceptor (全局/控制器级别)
// 2. SecondInterceptor (全局/控制器级别)
// 3. ThirdInterceptor (方法级别)
```

### . 作用域的实践建议

1. **方法级别**：适用于特定业务逻辑
2. **控制器级别**：适用于同类资源的通用处理
3. **全局级别**：适用于全应用的横切关注点

### 关键点总结

- 拦截器支持多个绑定级别
- 可以灵活组合使用
- 遵循"责任单一"原则
- 注意执行顺序和性能开销

## 常见应用场景和最佳实践

引入使用到的依赖：

```ts
import { 
  Injectable, 
  NestInterceptor, 
  ExecutionContext, 
  CallHandler,
  HttpException,
  HttpStatus,
  Logger,
  CacheInterceptor,
  HttpAdapterHost
} from '@nestjs/common';
import { 
  Observable, 
  throwError, 
  of 
} from 'rxjs';
import { 
  tap, 
  catchError, 
  map, 
  timeout,
  timeoutWith
} from 'rxjs/operators';
import { 
  Request, 
  Response 
} from 'express';
```

### 日志记录拦截器

这个拦截器可以记录所有请求的详细信息，包括执行时间、请求参数和响应结果。

```ts
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest<Request>();
    const { method, url, body, headers } = request;
    const startTime = Date.now();
    const requestId = headers['x-request-id'] || this.generateRequestId();

    // 请求开始时记录
    this.logger.log({
      message: 'Request started',
      requestId,
      method,
      url,
      body,
      timestamp: new Date().toISOString()
    });

    return next.handle().pipe(
      tap({
        next: (response) => {
          // 请求成功时记录
          const duration = Date.now() - startTime;
          this.logger.log({
            message: 'Request completed successfully',
            requestId,
            method,
            url,
            duration,
            response,
            timestamp: new Date().toISOString()
          });
        },
        error: (error) => {
          // 请求失败时记录
          const duration = Date.now() - startTime;
          this.logger.error({
            message: 'Request failed',
            requestId,
            method,
            url,
            duration,
            error: {
              message: error.message,
              stack: error.stack
            },
            timestamp: new Date().toISOString()
          });
        }
      })
    );
  }

  private generateRequestId(): string {
    return `req_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

### 响应转换拦截器

统一处理响应格式，确保 API 返回统一的数据结构。

```ts
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const response = context.switchToHttp().getResponse<Response>();
    const request = context.switchToHttp().getRequest<Request>();

    return next.handle().pipe(
      map(data => ({
        code: response.statusCode,
        message: 'Success',
        data,
        path: request.url,
        timestamp: new Date().toISOString(),
        metadata: {
          userAgent: request.headers['user-agent'],
          ip: request.ip
        }
      })),
      catchError(error => {
        return throwError(() => ({
          code: error.status || HttpStatus.INTERNAL_SERVER_ERROR,
          message: error.message || 'Internal server error',
          path: request.url,
          timestamp: new Date().toISOString(),
          errors: error.response?.errors || []
        }));
      })
    );
  }
}
```

### 性能监控拦截器

监控 API 性能和响应时间，可以与监控系统集成。

```ts
@Injectable()
export class PerformanceInterceptor implements NestInterceptor {
  private readonly logger = new Logger(PerformanceInterceptor.name);
  
  constructor(
    private readonly performanceThreshold: number = 1000 // 默认阈值1秒
  ) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const startTime = process.hrtime();
    const request = context.switchToHttp().getRequest<Request>();

    return next.handle().pipe(
      tap(() => {
        const [seconds, nanoseconds] = process.hrtime(startTime);
        const duration = seconds * 1000 + nanoseconds / 1000000; // 转换为毫秒

        // 记录性能指标
        const performanceMetrics = {
          endpoint: request.url,
          method: request.method,
          duration,
          timestamp: new Date().toISOString(),
          slow: duration > this.performanceThreshold
        };

        if (duration > this.performanceThreshold) {
          this.logger.warn({
            message: 'Slow API performance detected',
            ...performanceMetrics
          });
        }

        // 这里可以将性能指标发送到监控系统
        this.sendToMonitoringSystem(performanceMetrics);
      })
    );
  }

  private sendToMonitoringSystem(metrics: any) {
    // 实现发送指标到监控系统的逻辑
    // 例如: Prometheus, Grafana, ELK等
  }
}
```

### 高级缓存拦截器

实现一个带有过期时间和自动更新的缓存拦截器。

```ts
@Injectable()
export class AdvancedCacheInterceptor implements NestInterceptor {
  private cache = new Map<string, {
    data: any;
    expireAt: number;
    updating: boolean;
  }>();

  constructor(
    private readonly ttl: number = 60000, // 缓存时间，默认1分钟
    private readonly staleWhileRevalidate: number = 30000 // 后台更新时间，默认30秒
  ) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest<Request>();
    const cacheKey = this.generateCacheKey(request);

    // 检查缓存是否存在且有效
    const cachedData = this.cache.get(cacheKey);
    const now = Date.now();

    if (cachedData) {
      // 如果缓存未过期，直接返回
      if (cachedData.expireAt > now) {
        return of(cachedData.data);
      }

      // 如果缓存已过期但在后台更新时间内，返回旧数据并触发更新
      if (!cachedData.updating && 
          cachedData.expireAt + this.staleWhileRevalidate > now) {
        this.updateCacheInBackground(cacheKey, next);
        return of(cachedData.data);
      }
    }

    // 获取新数据并缓存
    return this.fetchAndCacheData(cacheKey, next);
  }

  private generateCacheKey(request: Request): string {
    return `${request.method}:${request.url}:${JSON.stringify(request.query)}`;
  }

  private fetchAndCacheData(
    cacheKey: string, 
    next: CallHandler
  ): Observable<any> {
    return next.handle().pipe(
      tap(data => {
        this.cache.set(cacheKey, {
          data,
          expireAt: Date.now() + this.ttl,
          updating: false
        });
      })
    );
  }

  private updateCacheInBackground(
    cacheKey: string, 
    next: CallHandler
  ): void {
    const cachedData = this.cache.get(cacheKey);
    if (cachedData) {
      cachedData.updating = true;
      this.fetchAndCacheData(cacheKey, next).subscribe();
    }
  }
}
```

### 最佳实践-1.模块化和可配置性

```ts
// 创建配置接口
interface InterceptorConfig {
  logging?: boolean;
  performance?: {
    enabled: boolean;
    threshold: number;
  };
  cache?: {
    enabled: boolean;
    ttl: number;
  };
}

// 创建可配置的拦截器工厂
@Injectable()
export class ConfigurableInterceptorFactory {
  static create(config: InterceptorConfig): NestInterceptor {
    return {
      intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        const interceptors: NestInterceptor[] = [];

        if (config.logging) {
          interceptors.push(new LoggingInterceptor());
        }

        if (config.performance?.enabled) {
          interceptors.push(
            new PerformanceInterceptor(config.performance.threshold)
          );
        }

        if (config.cache?.enabled) {
          interceptors.push(
            new AdvancedCacheInterceptor(config.cache.ttl)
          );
        }

        // 串联所有拦截器
        return interceptors.reduce(
          (acc, interceptor) => 
            interceptor.intercept(context, { handle: () => acc }),
          next.handle()
        );
      }
    };
  }
}
```

### 最佳实践-2.错误处理最佳实践

```ts
@Injectable()
export class ErrorHandlingInterceptor implements NestInterceptor {
  constructor(private readonly httpAdapterHost: HttpAdapterHost) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000), // 添加超时处理
      timeoutWith(5000, throwError(() => new HttpException(
        'Request timeout',
        HttpStatus.REQUEST_TIMEOUT
      ))),
      catchError(error => {
        // 统一错误处理逻辑
        const { httpAdapter } = this.httpAdapterHost;
        const ctx = context.switchToHttp();
        const response = ctx.getResponse();
        const request = ctx.getRequest();

        const errorResponse = {
          statusCode: error?.status || HttpStatus.INTERNAL_SERVER_ERROR,
          message: error?.message || 'Internal server error',
          timestamp: new Date().toISOString(),
          path: request.url,
          details: process.env.NODE_ENV === 'development' ? error?.stack : undefined
        };

        httpAdapter.reply(response, errorResponse, errorResponse.statusCode);
        return throwError(() => error);
      })
    );
  }
}
```

#### 全局注册

如果你希望拦截器在整个应用程序中生效，可以在 main.ts 文件中全局注册它：

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ErrorHandlingInterceptor } from './path/to/interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 全局注册拦截器
  app.useGlobalInterceptors(new ErrorHandlingInterceptor(app.get(HttpAdapterHost)));

  await app.listen(3000);
}
bootstrap();
```

#### 模块级注册

如果你只希望在特定模块中使用拦截器，可以在模块的 providers 数组中注册它：

```ts
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { ErrorHandlingInterceptor } from './path/to/interceptor';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: ErrorHandlingInterceptor,
    },
  ],
})
export class SomeModule {}
```

#### 控制器级注册

如果你只希望在特定控制器中使用拦截器，可以在控制器的装饰器中使用 @UseInterceptors：

```ts
import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { ErrorHandlingInterceptor } from './path/to/interceptor';

@Controller('some-route')
@UseInterceptors(ErrorHandlingInterceptor)
export class SomeController {
  @Get()
  someMethod() {
    // Your method logic
  }
}
```

## 高级特性和技巧

引入依赖：

```ts
import { 
  Injectable, 
  NestInterceptor, 
  ExecutionContext, 
  CallHandler,
  Inject,
  forwardRef,
  Scope,
  Optional,
  HttpException,
  HttpStatus
} from '@nestjs/common';
import { 
  Observable, 
  from, 
  throwError,
  of,
  defer
} from 'rxjs';
import { 
  map, 
  catchError, 
  mergeMap,
  tap,
  retryWhen,
  delay,
  take
} from 'rxjs/operators';
import { Reflector } from '@nestjs/core';
import { 
  REQUEST,
  ModuleRef 
} from '@nestjs/core';
```

### 异步拦截器实现

异步拦截器允许我们在拦截器中处理异步操作，比如从数据库获取数据或调用外部服务。

```ts
@Injectable()
export class AsyncOperationInterceptor implements NestInterceptor {
  constructor(
    private readonly moduleRef: ModuleRef,
    @Inject(REQUEST) private readonly request: Request
  ) {}

  async intercept(
    context: ExecutionContext, 
    next: CallHandler
  ): Promise<Observable<any>> {
    // 异步获取用户信息
    const user = await this.loadUserData();
    
    // 异步加载配置
    const config = await this.loadConfiguration();

    // 使用 defer 来处理异步操作
    return defer(async () => {
      // 前置异步操作
      await this.beforeRequest(user, config);

      return next.handle().pipe(
        mergeMap(async (data) => {
          // 后置异步操作
          const enrichedData = await this.enrichResponse(data, user);
          return enrichedData;
        })
      );
    });
  }

  private async loadUserData(): Promise<any> {
    const userService = await this.moduleRef.resolve('UserService');
    return userService.findByRequest(this.request);
  }

  private async loadConfiguration(): Promise<any> {
    const configService = await this.moduleRef.resolve('ConfigService');
    return configService.getConfig();
  }

  private async beforeRequest(user: any, config: any): Promise<void> {
    // 执行请求前的异步操作
    await this.validateUserPermissions(user, config);
  }

  private async enrichResponse(data: any, user: any): Promise<any> {
    // 扩充响应数据
    return {
      ...data,
      userContext: user,
      timestamp: new Date()
    };
  }

  private async validateUserPermissions(user: any, config: any): Promise<void> {
    if (!user.permissions.includes(config.requiredPermission)) {
      throw new HttpException('Permission denied', HttpStatus.FORBIDDEN);
    }
  }
}
```

### 依赖注入高级用法

我们可以在拦截器中使用 Nest.js 的高级依赖注入特性。

```ts
@Injectable({ scope: Scope.REQUEST })
export class AdvancedDIInterceptor implements NestInterceptor {
  constructor(
    private readonly reflector: Reflector,
    @Optional() private readonly logger?: LoggerService,
    @Inject(forwardRef(() => UserService))
    private readonly userService?: UserService
  ) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // 使用反射获取元数据
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const requiredPermissions = this.reflector.get<string[]>(
      'permissions', 
      context.getClass()
    );

    return next.handle().pipe(
      tap(data => {
        // 使用可选注入的 logger
        if (this.logger) {
          this.logger.log({
            roles,
            requiredPermissions,
            data
          });
        }
      }),
      mergeMap(async (data) => {
        // 使用循环引用注入的服务
        if (this.userService) {
          const userDetails = await this.userService.enrichUserData(data);
          return { ...data, ...userDetails };
        }
        return data;
      })
    );
  }
}
```

### 高级错误处理

实现一个带有重试机制和详细错误处理的拦截器。

```ts
@Injectable()
export class AdvancedErrorHandlingInterceptor implements NestInterceptor {
  private readonly maxRetries = 3;
  private readonly initialRetryDelay = 1000;

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      retryWhen(errors => 
        errors.pipe(
          mergeMap((error, index) => {
            const retryAttempt = index + 1;

            // 根据错误类型决定是否重试
            if (this.isRetryableError(error) && retryAttempt <= this.maxRetries) {
              const delay = this.calculateRetryDelay(retryAttempt);
              return of(error).pipe(delay(delay));
            }

            return throwError(() => this.enhanceError(error));
          })
        )
      ),
      catchError(error => {
        const enhancedError = this.createDetailedError(error, context);
        return throwError(() => enhancedError);
      })
    );
  }

  private isRetryableError(error: any): boolean {
    // 判断错误是否可重试
    return (
      error.status === HttpStatus.SERVICE_UNAVAILABLE ||
      error.code === 'NETWORK_ERROR' ||
      error instanceof TimeoutError
    );
  }

  private calculateRetryDelay(attempt: number): number {
    // 实现指数退避算法
    return this.initialRetryDelay * Math.pow(2, attempt - 1);
  }

  private enhanceError(error: any): HttpException {
    const errorResponse = {
      statusCode: error.status || HttpStatus.INTERNAL_SERVER_ERROR,
      message: error.message,
      timestamp: new Date().toISOString(),
      correlationId: this.generateCorrelationId(),
      stack: process.env.NODE_ENV === 'development' ? error.stack : undefined,
      context: {
        name: error.name,
        code: error.code
      }
    };

    return new HttpException(errorResponse, errorResponse.statusCode);
  }

  private createDetailedError(error: any, context: ExecutionContext): Error {
    const request = context.switchToHttp().getRequest();
    return {
      ...error,
      request: {
        url: request.url,
        method: request.method,
        headers: request.headers
      }
    };
  }

  private generateCorrelationId(): string {
    return `err_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

### 高级状态管理

实现一个可以在请求生命周期中管理状态的拦截器。

```ts
@Injectable({ scope: Scope.REQUEST })
export class StateManagementInterceptor implements NestInterceptor {
  private readonly state = new Map<string, any>();

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const requestId = this.generateRequestId();
    
    // 初始化请求状态
    this.initializeRequestState(requestId, context);

    return next.handle().pipe(
      tap({
        next: (data) => {
          // 更新状态
          this.updateState(requestId, 'response', data);
          this.updateState(requestId, 'completed', true);
        },
        error: (error) => {
          // 错误状态处理
          this.updateState(requestId, 'error', error);
          this.updateState(requestId, 'completed', false);
        },
        finalize: () => {
          // 清理状态
          this.cleanupState(requestId);
        }
      }),
      map(data => {
        // 在响应中包含状态信息
        return {
          ...data,
          state: this.getRequestState(requestId)
        };
      })
    );
  }

  private initializeRequestState(requestId: string, context: ExecutionContext) {
    const request = context.switchToHttp().getRequest();
    this.state.set(requestId, {
      startTime: Date.now(),
      request: {
        method: request.method,
        url: request.url,
        params: request.params,
        query: request.query,
        body: request.body
      },
      completed: false,
      processing: true
    });
  }

  private updateState(requestId: string, key: string, value: any) {
    const currentState = this.state.get(requestId) || {};
    this.state.set(requestId, {
      ...currentState,
      [key]: value,
      lastUpdated: Date.now()
    });
  }

  private getRequestState(requestId: string): any {
    return this.state.get(requestId);
  }

  private cleanupState(requestId: string) {
    // 设置延迟清理，以便其他拦截器可以访问状态
    setTimeout(() => {
      this.state.delete(requestId);
    }, 5000);
  }

  private generateRequestId(): string {
    return `req_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```
