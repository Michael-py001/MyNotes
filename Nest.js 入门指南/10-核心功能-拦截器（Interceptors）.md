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

## 高级特性和技巧

## 与其他功能的集成使用