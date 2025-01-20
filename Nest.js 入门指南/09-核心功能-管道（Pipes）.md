# 09-核心功能-管道（Pipes）

## 管道的基础概念和作用（核心重要）

### 什么是管道？

管道是 Nest.js 中的一个核心概念，它的主要职责是在数据到达控制器之前对数据进行转换或验证。你可以把管道想象成一个数据的 "过滤器" 或 "处理器"，就像自来水管道中的过滤器一样，确保流过的水（数据）符合我们的要求。

让我们通过一个简单的例子来理解：

> ParseIntPipe 是 Nest.js 内置的管道之一，它的主要作用是将字符串转换为整数。

```ts
// user.controller.ts
import { Controller, Get, Param, ParseIntPipe } from '@nestjs/common';

@Controller('users')
export class UserController {
  // 方式一：直接使用内置的 ParseIntPipe
  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return `Finding user with id: ${id} (type: ${typeof id})`;
  }

  // 方式二：使用带配置的 ParseIntPipe
  @Get('v2/:id')
  findOneV2(
    @Param('id', new ParseIntPipe({ errorHttpStatusCode: 400 }))
    id: number,
  ) {
    return `Finding user with id: ${id}`;
  }

  // 方式三：处理可选参数
  @Get('search')
  search(
    @Query('page', new ParseIntPipe({ optional: true }))
    page: number = 1,
  ) {
    return `Searching users on page: ${page}`;
  }
}
```

在这个例子中，ParseIntPipe 会确保传入的 id 参数被转换为数字类型。如果转换失败（比如传入 'abc'），管道会自动抛出异常。

### 管道的执行时机

管道在请求处理流程中的执行位置非常重要。它的执行顺序是： 中间件 -> 守卫 -> 拦截器（请求） -> 管道 -> 控制器 -> 拦截器（响应）

让我们通过一个更复杂的例子来理解这个流程：

```ts
// create-user.dto.ts
export class CreateUserDto {
  @IsString()
  @MinLength(2)
  @MaxLength(20)
  name: string;

  @IsEmail()
  email: string;

  @IsInt()
  @Min(0)
  @Max(150)
  age: number;
}

// user.controller.ts
@Controller('users')
export class UserController {
  @Post()
  create(@Body(new ValidationPipe()) createUserDto: CreateUserDto) {
    // 这里的 createUserDto 已经经过验证和转换
    return `Creating user: ${createUserDto.name}`;
  }
}
```

### 管道的两个主要用途

1.  **转换**（Transformation）：

   将输入数据转换为所需的形式。例如：

   ```ts
   // transform.pipe.ts
   @Injectable()
   export class ToUppercasePipe implements PipeTransform {
     transform(value: string): string {
       return value.toUpperCase();
     }
   }
   
   // user.controller.ts
   @Controller('users')
   export class UserController {
     @Post()
     create(@Body('name', ToUppercasePipe) name: string) {
       // name 会被自动转换为大写
       return `Creating user: ${name}`;
     }
   }
   ```

2. **验证**（Validation）

   验证输入数据是否符合要求。例如：

   ```ts
   // validation.pipe.ts
   @Injectable()
   export class AgeValidationPipe implements PipeTransform<string, number> {
     transform(value: string, metadata: ArgumentMetadata): number {
       const age = parseInt(value);
       if (isNaN(age) || age < 0 || age > 150) {
         throw new BadRequestException('Invalid age value');
       }
       return age;
     }
   }
   
   // user.controller.ts
   @Controller('users')
   export class UserController {
     @Get(':age')
     findByAge(@Param('age', AgeValidationPipe) age: number) {
       return `Finding users with age: ${age}`;
     }
   }
   ```

### 管道与中间件的区别

主要区别在于：

- 中间件处理的是原始请求对象，无法访问将要执行的处理程序及其参数
- 管道主要处理参数，可以转换或验证传入控制器的参数
- 管道执行在中间件之后，但在控制器之前

实际应用示例：

```ts
// user.module.ts
@Module({
  controllers: [UserController],
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe, // 全局验证管道
    },
  ],
})
export class UserModule {}

// user.controller.ts
@Controller('users')
export class UserController {
  @Post()
  @UsePipes(new ValidationPipe({ transform: true }))
  create(@Body() createUserDto: CreateUserDto) {
    // 在这里，createUserDto 已经：
    // 1. 经过了数据验证（确保符合 DTO 中的规则）
    // 2. 经过了数据转换（比如字符串类型的数字被转换为数字类型）
    return `Creating user: ${createUserDto.name}`;
  }
}
```

在实际开发中，ParseIntPipe 常用于处理：

- URL 参数转换
- 查询字符串参数转换
- 分页参数处理
- ID 参数验证

## 内置管道的使用（重要）

@UsePipes 是 NestJS 中的一个装饰器，用于在控制器或处理程序中应用管道。管道可以用于数据验证、转换和处理请求数据。你可以使用 NestJS 提供的内置管道，如 ValidationPipe，也可以创建自定义管道。

### ValidationPipe（数据验证管道）

ValidationPipe 是最常用的内置管道，它能够根据 DTO 类中的装饰器自动验证传入的数据。

#### ValidationPipe 使用方式分析

1. **全参数验证方式**

   ```ts
   @UsePipes(new ValidationPipe(options))
   ```

   - 验证整个 DTO 对象
   - 可以配置多个验证选项
   - 适用于需要验证完整请求体的场景

2. **单参数验证方式**

   ```ts
   @Body('email', ValidationPipe)
   ```

   - 只验证特定参数
   - 直接提取并验证单个字段
   - 适用于只需验证个别参数的场景

#### 区别对比

1. **作用范围**

   ```ts
   // 整个对象验证
   @UsePipes(new ValidationPipe())
   create(@Body() dto: CreateUserDto) {}
   
   // 单个参数验证
   @Post()
   check(@Body('email', ValidationPipe) email: string) {}
   ```

2. **验证选项**

   ```ts
   // 整体验证可配置更多选项
   @UsePipes(new ValidationPipe({
     transform: true,
     whitelist: true,
     forbidNonWhitelisted: true
   }))
   
   // 单参数验证通常用于简单校验
   @Body('email', new ValidationPipe())
   ```

```ts
// user.dto.ts
import { IsString, IsEmail, IsInt, Min, Max } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  @MaxLength(20)
  name: string;

  @IsEmail()
  email: string;

  @IsInt()
  @Min(0)
  @Max(150)
  age: number;
}

// user.controller.ts
@Controller('users')
export class UserController {
  @Post()
  @UsePipes(new ValidationPipe({
    transform: true,           // 自动转换类型
    whitelist: true,          // 去除未定义的属性
    forbidNonWhitelisted: true, // 出现未定义属性时抛出错误
  }))
  create(@Body() createUserDto: CreateUserDto) {
    // createUserDto 已经过验证，类型也已转换
    return `Creating user: ${createUserDto.name}`;
  }
}
```

在这个例子中，ValidationPipe 会在请求到达控制器之前验证 createUserDto 的数据。



### ParseIntPipe（整数转换管道）

ParseIntPipe 用于确保参数为数字类型，我们来看一些高级用法：

```ts
// user.controller.ts
@Controller('users')
export class UserController {
  // 基本用法
  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return `User ${id}`;
  }

  // 自定义错误处理
  @Get('v2/:id')
  findOneV2(
    @Param('id', new ParseIntPipe({
      errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE,
      exceptionFactory: (error) => {
        throw new HttpException({
          status: HttpStatus.NOT_ACCEPTABLE,
          error: `ID must be a number, received: ${error.value}`,
        }, HttpStatus.NOT_ACCEPTABLE);
      },
    }))
    id: number,
  ) {
    return `User ${id}`;
  }
}
```

### ParseBoolPipe（布尔值转换管道）

ParseBoolPipe 用于将字符串转换为布尔值，支持多种格式的输入：

```ts
// user.controller.ts
@Controller('users')
export class UserController {
  @Get('active')
  findActive(
    @Query('isActive', ParseBoolPipe) isActive: boolean,
    @Query('hasEmail', new ParseBoolPipe({ optional: true })) hasEmail?: boolean,
  ) {
    return {
      message: `Finding ${isActive ? 'active' : 'inactive'} users`,
      withEmail: hasEmail,
    };
  }
}

// 可接受的输入示例：
// GET /users/active?isActive=true
// GET /users/active?isActive=1
// GET /users/active?isActive=on
```

### ParseArrayPipe（数组转换管道）

ParseArrayPipe 用于处理数组类型的参数，可以结合其他管道使用：

```ts
// user.controller.ts
@Controller('users')
export class UserController {
  @Post('batch')
  createBatch(
    @Body(new ParseArrayPipe({
      items: CreateUserDto,
      whitelist: true,
      forbidNonWhitelisted: true,
    }))
    createUserDtos: CreateUserDto[],
  ) {
    return `Creating ${createUserDtos.length} users`;
  }

  // 处理查询参数数组
  @Get('by-ids')
  findByIds(
    @Query('ids', new ParseArrayPipe({
      items: Number,
      separator: ',',
      optional: true,
    }))
    ids: number[] = [],
  ) {
    return `Finding users with IDs: ${ids.join(', ')}`;
  }
}
```

### ParseUUIDPipe（UUID 验证管道）

ParseUUIDPipe 用于验证参数是否为有效的 UUID：

```ts
// user.controller.ts
@Controller('users')
export class UserController {
  @Get(':uuid')
  findByUUID(
    @Param('uuid', new ParseUUIDPipe({
      version: '4',
      errorHttpStatusCode: HttpStatus.BAD_REQUEST,
    }))
    uuid: string,
  ) {
    return `Finding user with UUID: ${uuid}`;
  }
}
```

### 组合使用示例

我们可以组合多个管道来实现更复杂的验证和转换：

```ts
// user.controller.ts
@Controller('users')
export class UserController {
  @Post('bulk-create')
  @UsePipes(new ValidationPipe({ transform: true }))
  bulkCreate(
    @Body(new ParseArrayPipe({ items: CreateUserDto }))
    users: CreateUserDto[],
    @Query('groupId', ParseIntPipe) groupId: number,
    @Query('isActive', ParseBoolPipe) isActive: boolean,
  ) {
    return {
      message: `Creating ${users.length} users in group ${groupId}`,
      active: isActive,
    };
  }
}
```

### 全局配置示例

在实际项目中，我们经常需要全局配置某些管道：

```ts
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalPipes(
    new ValidationPipe({
      transform: true,
      whitelist: true,
      forbidNonWhitelisted: true,
      transformOptions: {
        enableImplicitConversion: true,
      },
    }),
  );
  
  await app.listen(3000);
}
bootstrap();
```



## 管道参数装饰器的应用（重要）

首先，我们先了解一下参数装饰器的本质。在 Nest.js 中，参数装饰器用于标记和处理不同来源的请求数据。每个装饰器都专注于处理特定类型的输入数据，并可以与管道配合使用来实现数据验证和转换。

### @Body() 装饰器的使用

@Body() 装饰器用于获取请求体中的数据。让我们通过一个用户注册的例子来理解：

```ts
// user.dto.ts
import { IsString, IsEmail, IsOptional, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  username: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6)
  password: string;

  @IsOptional()
  @IsString()
  description?: string;
}

// user.controller.ts
import { Body, Controller, Post, ValidationPipe } from '@nestjs/common';

@Controller('users')
export class UserController {
  // 获取整个请求体
  @Post()
  createUser(@Body(ValidationPipe) createUserDto: CreateUserDto) {
    return `Creating user with email: ${createUserDto.email}`;
  }

  // 或者
  @Post()
  @UsePipes(new ValidationPipe())
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }

  // 获取请求体中的特定字段
  @Post('check-email')
  checkEmail(@Body('email', ValidationPipe) email: string) {
    return `Checking email: ${email}`;
  }

  // 处理嵌套的请求体数据
  @Post('with-profile')
  createWithProfile(
  @Body('user', ValidationPipe) user: CreateUserDto,
   @Body('profile.age', ParseIntPipe) age: number
  ) {
    return `Creating user ${user.username} with age ${age}`;
  }

  // 混合使用
  @Post()
  createWithValidation(
  @Body() dto: CreateUserDto,
   @Body('role', new ParseEnumPipe(UserRole)) role: UserRole,
  ) {
    return this.userService.createWithRole(dto, role);
  }
}
```

### @Query() 装饰器的应用

@Query() 用于处理 URL 查询参数。我们来看一个产品搜索的例子：

```ts
// product.dto.ts
import { IsOptional, IsInt, Min, IsString } from 'class-validator';
import { Transform } from 'class-transformer';

export class ProductQueryDto {
  @IsOptional()
  @IsString()
  search?: string;

  @IsOptional()
  @IsInt()
  @Min(1)
  @Transform(({ value }) => parseInt(value))
  page?: number = 1;

  @IsOptional()
  @IsInt()
  @Min(1)
  @Transform(({ value }) => parseInt(value))
  limit?: number = 10;
}

// product.controller.ts
import { Controller, Get, Query } from '@nestjs/common';

@Controller('products')
export class ProductController {
  // 使用 DTO 处理查询参数
  @Get()
  findAll(@Query(ValidationPipe) query: ProductQueryDto) {
    return {
      search: query.search,
      page: query.page,
      limit: query.limit
    };
  }

  // 获取单个查询参数
  @Get('by-category')
  findByCategory(
    @Query('category') category: string,
    @Query('sort', new DefaultValuePipe('asc')) sort: string
  ) {
    return `Finding products in category: ${category}, sorted: ${sort}`;
  }
}
```

### @Param() 装饰器的实践

@Param() 用于获取路由参数。让我们通过一个订单管理的例子来理解：

```ts
// order.controller.ts
import { Controller, Get, Param, ParseUUIDPipe } from '@nestjs/common';

@Controller('orders')
export class OrderController {
  // 获取单个路由参数
  @Get(':id')
  findOne(@Param('id', ParseUUIDPipe) id: string) {
    return `Finding order: ${id}`;
  }

  // 处理多个路由参数
  @Get(':year/:month')
  findByDate(
    @Param('year', ParseIntPipe) year: number,
    @Param('month', ParseIntPipe) month: number
  ) {
    return `Finding orders for ${year}-${month}`;
  }

  // 获取所有路由参数
  @Get(':userId/status/:status')
  findByUserAndStatus(@Param() params: Record<string, string>) {
    return `User: ${params.userId}, Status: ${params.status}`;
  }
}
```

### @Headers() 装饰器的运用

@Headers() 用于访问请求头信息。这在处理认证、版本控制等场景非常有用：

```ts
// auth.controller.ts
import { Controller, Get, Headers, UnauthorizedException } from '@nestjs/common';

@Controller('auth')
export class AuthController {
  // 获取特定请求头
  @Get('profile')
  getProfile(@Headers('authorization') auth: string) {
    if (!auth) {
      throw new UnauthorizedException('No authorization token provided');
    }
    return `Processing auth token: ${auth}`;
  }

  // 获取多个请求头
  @Get('info')
  getInfo(
    @Headers('user-agent') userAgent: string,
    @Headers('accept-language') language: string
  ) {
    return {
      userAgent,
      language
    };
  }

  // 获取所有请求头
  @Get('headers')
  getAllHeaders(@Headers() headers: Record<string, string>) {
    return headers;
  }
}
```

### 组合使用的高级示例

在实际开发中，我们经常需要组合使用多个参数装饰器：

```ts
// payment.controller.ts
import { Controller, Post, Body, Param, Query, Headers } from '@nestjs/common';

@Controller('payments')
export class PaymentController {
  @Post(':orderId/process')
  processPayment(
    @Param('orderId', ParseUUIDPipe) orderId: string,
    @Body(ValidationPipe) paymentDetails: PaymentDetailsDto,
    @Query('currency', new DefaultValuePipe('USD')) currency: string,
    @Headers('x-api-key') apiKey: string
  ) {
    // 验证 API 密钥
    if (!apiKey) {
      throw new UnauthorizedException('API key is required');
    }

    return {
      orderId,
      currency,
      amount: paymentDetails.amount,
      status: 'processing'
    };
  }
}
```

这些装饰器的组合使用让我们能够更优雅地处理来自不同来源的请求数据。每个装饰器都可以配合管道使用，从而实现数据验证和转换。

## 自定义管道的开发（较重要）

- 实现 PipeTransform 接口
- transform() 方法的开发
- 异步管道的实现
- 管道异常处理

### 1. 基础概念和结构

自定义管道需要实现 `PipeTransform` 接口，这个接口要求我们实现 `transform` 方法。让我们先创建一个简单的管道来理解基本结构：

让我来详细讲解如何开发自定义管道。我会从基础概念开始，逐步深入到更复杂的实现。

```ts
// trim.pipe.ts
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class TrimPipe implements PipeTransform {
  // transform 方法接收两个参数：
  // value: 当前处理的参数值
  // metadata: 参数的元数据
  transform(value: any, metadata: ArgumentMetadata) {
    // 如果值是字符串，去除两端空格
    if (typeof value === 'string') {
      return value.trim();
    }
    // 如果是对象，递归处理所有字符串属性
    if (typeof value === 'object' && value !== null) {
      Object.keys(value).forEach(key => {
        if (typeof value[key] === 'string') {
          value[key] = value[key].trim();
        }
      });
    }
    return value;
  }
}
```

### 2. 实现带验证的管道

让我们创建一个更实用的管道，用于验证年龄范围：

```ts
// age-validation.pipe.ts
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class AgeValidationPipe implements PipeTransform<string, number> {
  constructor(private readonly minAge: number = 0, private readonly maxAge: number = 120) {}

  transform(value: string, metadata: ArgumentMetadata): number {
    // 将输入转换为数字
    const age = parseInt(value, 10);

    // 验证是否为有效数字
    if (isNaN(age)) {
      throw new BadRequestException('Age must be a number');
    }

    // 验证年龄范围
    if (age < this.minAge || age > this.maxAge) {
      throw new BadRequestException(
        `Age must be between ${this.minAge} and ${this.maxAge}`
      );
    }

    return age;
  }
}

// 使用示例：
@Controller('users')
export class UserController {
  @Post()
  create(
    @Body('age', new AgeValidationPipe(18, 100)) age: number
  ) {
    return `Creating user with age: ${age}`;
  }
}
```

### 3.异步管道的实现

有时我们需要在管道中进行异步操作，比如检查数据库中的唯一性：

```ts
// unique-username.pipe.ts
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { UserService } from './user.service';

@Injectable()
export class UniqueUsernamePipe implements PipeTransform<string, Promise<string>> {
  constructor(private readonly userService: UserService) {}

  async transform(value: string, metadata: ArgumentMetadata): Promise<string> {
    // 首先进行基本验证
    if (!value || value.trim().length < 3) {
      throw new BadRequestException('Username must be at least 3 characters long');
    }

    // 检查用户名是否已存在
    const exists = await this.userService.checkUsernameExists(value);
    if (exists) {
      throw new BadRequestException(`Username ${value} is already taken`);
    }

    return value.trim();
  }
}

// 使用示例：
@Controller('users')
export class UserController {
  @Post('register')
  async register(
    @Body('username', UniqueUsernamePipe) username: string
  ) {
    return `Registering user: ${username}`;
  }
}
```

### 4. 复杂数据转换管道

下面是一个处理复杂数据转换的例子，它可以将逗号分隔的字符串转换为数组并验证每个元素：

```ts
// parse-array-with-validation.pipe.ts
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

interface ParseArrayOptions {
  separator?: string;
  minSize?: number;
  maxSize?: number;
  unique?: boolean;
  transform?: (item: string) => any;
}

@Injectable()
export class ParseArrayWithValidationPipe implements PipeTransform {
  constructor(private options: ParseArrayOptions = {}) {
    this.options = {
      separator: ',',
      minSize: 1,
      maxSize: 100,
      unique: true,
      transform: (item: string) => item.trim(),
      ...options
    };
  }

  transform(value: string, metadata: ArgumentMetadata) {
    // 如果值为空，且允许空值
    if (!value && this.options.minSize === 0) {
      return [];
    }

    // 分割字符串为数组
    const array = value.split(this.options.separator);

    // 验证数组大小
    if (array.length < this.options.minSize) {
      throw new BadRequestException(
        `Array must contain at least ${this.options.minSize} elements`
      );
    }

    if (array.length > this.options.maxSize) {
      throw new BadRequestException(
        `Array must contain no more than ${this.options.maxSize} elements`
      );
    }

    // 转换并验证每个元素
    let result = array.map(item => {
      try {
        return this.options.transform(item);
      } catch (error) {
        throw new BadRequestException(`Failed to transform item: ${item}`);
      }
    });

    // 如果需要去重
    if (this.options.unique) {
      result = [...new Set(result)];
    }

    return result;
  }
}

// 使用示例：
@Controller('products')
export class ProductController {
  @Get('by-ids')
  findByIds(
    @Query('ids', new ParseArrayWithValidationPipe({
      transform: (item) => {
        const id = parseInt(item);
        if (isNaN(id)) {
          throw new Error('Invalid ID');
        }
        return id;
      },
      minSize: 1,
      maxSize: 10,
      unique: true
    }))
    ids: number[]
  ) {
    return `Finding products with IDs: ${ids.join(', ')}`;
  }
}
```

### 5. 管道异常处理

为了使管道更加健壮，我们应该合理处理各种可能的异常：

```ts
// validated-parse.pipe.ts
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class ValidatedParsePipe implements PipeTransform {
  constructor(private readonly validationFn: (value: any) => boolean) {}

  transform(value: any, metadata: ArgumentMetadata) {
    try {
      // 尝试验证值
      if (!this.validationFn(value)) {
        throw new BadRequestException('Validation failed');
      }

      return value;
    } catch (error) {
      // 处理验证过程中的错误
      if (error instanceof BadRequestException) {
        throw error;
      }

      // 处理其他意外错误
      throw new BadRequestException(
        `Validation error: ${error.message || 'Unknown error'}`
      );
    }
  }
}
```

## 组合使用多个管道

### 1. 顺序执行方式

当我们使用多个管道时，它们会按照声明的顺序从左到右执行。让我们通过一个例子来理解:

```ts
// pipes/trim-transform.pipe.ts
@Injectable()
export class TrimTransformPipe implements PipeTransform {
  transform(value: any) {
    console.log('Executing TrimTransformPipe');
    if (typeof value === 'string') {
      return value.trim();
    }
    return value;
  }
}

// pipes/validation.pipe.ts
@Injectable()
export class CustomValidationPipe implements PipeTransform {
  transform(value: any) {
    console.log('Executing CustomValidationPipe');
    if (!value || value.length < 3) {
      throw new BadRequestException('Value must be at least 3 characters long');
    }
    return value;
  }
}

// user.controller.ts
@Controller('users')
export class UserController {
  @Post()
  createUser(
    @Body('username', TrimTransformPipe, CustomValidationPipe) username: string
  ) {
    // TrimTransformPipe 先执行，然后结果传递给 CustomValidationPipe
    return `Creating user: ${username}`;
  }
}
```

### 2. 使用装饰器组合

我们可以通过不同级别的装饰器来组合管道：

```ts
// user.controller.ts
@Controller('users')
export class UserController {
  @Post()
  @UsePipes(new ValidationPipe()) // 控制器方法级别的管道
  createUser(
    @Body() createUserDto: CreateUserDto,
    @Param('id', ParseIntPipe, CustomValidationPipe) id: number // 参数级别的管道
  ) {
    return `Creating user with ID: ${id}`;
  }
}
```

### 3. 创建复合管道

我们可以创建一个管道来封装多个验证和转换逻辑：

```ts
// pipes/composite.pipe.ts
@Injectable()
export class CompositePipe implements PipeTransform {
  private readonly pipes: PipeTransform[];

  constructor(...pipes: PipeTransform[]) {
    this.pipes = pipes;
  }

  async transform(value: any, metadata: ArgumentMetadata) {
    let result = value;
    
    for (const pipe of this.pipes) {
      // 处理异步和同步管道
      result = pipe.transform instanceof Promise 
        ? await pipe.transform(result, metadata)
        : pipe.transform(result, metadata);
    }
    
    return result;
  }
}

// 使用示例
@Controller('products')
export class ProductController {
  @Post()
  createProduct(
    @Body('price', new CompositePipe(
      new TrimTransformPipe(),
      new ParseFloatPipe(),
      new PriceValidationPipe(),
    )) 
    price: number
  ) {
    return `Creating product with price: ${price}`;
  }
}
```

### 4. 实际应用场景

让我们看一个更复杂的实际应用场景，涉及多个验证和转换步骤：

```ts
// dto/create-order.dto.ts
export class CreateOrderDto {
  @IsString()
  @MinLength(3)
  productId: string;

  @IsNumber()
  @Min(1)
  quantity: number;

  @IsOptional()
  @IsString()
  couponCode?: string;
}

// pipes/order-validation.pipe.ts
@Injectable()
export class OrderValidationPipe implements PipeTransform {
  constructor(private readonly orderService: OrderService) {}

  async transform(value: CreateOrderDto) {
    // 验证产品是否存在
    const productExists = await this.orderService.checkProductExists(value.productId);
    if (!productExists) {
      throw new BadRequestException('Product not found');
    }

    // 验证库存
    const hasStock = await this.orderService.checkStock(value.productId, value.quantity);
    if (!hasStock) {
      throw new BadRequestException('Insufficient stock');
    }

    // 验证优惠券（如果有的话）
    if (value.couponCode) {
      const isValidCoupon = await this.orderService.validateCoupon(value.couponCode);
      if (!isValidCoupon) {
        throw new BadRequestException('Invalid coupon code');
      }
    }

    return value;
  }
}

// order.controller.ts
@Controller('orders')
export class OrderController {
  @Post()
  @UsePipes(new ValidationPipe({ transform: true }))
  async createOrder(
    @Body(OrderValidationPipe) createOrderDto: CreateOrderDto,
    @Headers('x-user-id', new UserValidationPipe()) userId: string,
  ) {
    // 在这里，所有数据都已经经过验证和转换
    return this.orderService.createOrder(createOrderDto, userId);
  }
}
```

### 5. 全局和局部管道的组合

我们可以在不同层级组合管道：

```ts
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // 全局管道
  app.useGlobalPipes(
    new ValidationPipe({
      transform: true,
      whitelist: true,
    })
  );
  
  await app.listen(3000);
}

// app.module.ts
@Module({
  providers: [
    // 模块级别的管道
    {
      provide: APP_PIPE,
      useClass: CustomGlobalPipe,
    },
  ],
})
export class AppModule {}

// user.controller.ts
@Controller('users')
@UsePipes(new CustomControllerPipe()) // 控制器级别的管道
export class UserController {
  @Post()
  @UsePipes(new CustomMethodPipe()) // 方法级别的管道
  createUser(
    @Body(new CustomParameterPipe()) createUserDto: CreateUserDto // 参数级别的管道
  ) {
    return 'User created';
  }
}
```

在使用多个管道时，需要注意以下几点：

1. 执行顺序：管道按照声明顺序执行，从全局到局部
2. 错误处理：一旦某个管道抛出异常，后续管道将不会执行
3. 性能考虑：组合使用多个管道可能会影响性能，应该适当合并管道功能
4. 依赖注入：每个管道都可以注入所需的服务

## 管道的绑定范围（中等重要）

### 1.参数级绑定

```ts
import { Controller, Post, Body, ParseIntPipe } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Post()
  create(
    @Body('age', ParseIntPipe) age: number,
    @Body('id', ParseIntPipe) id: number
  ) {
    return { age, id };
  }
}
```

### 2.方法级绑定

```ts
import { Controller, Post, UsePipes, ValidationPipe } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Post()
  @UsePipes(new ValidationPipe())
  create(@Body() createUserDto: CreateUserDto) {
    return createUserDto;
  }
}
```

### 3.全局级绑定

```ts
// filepath: src/main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // 全局管道
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,
    transform: true
  }));
  await app.listen(3000);
}
```

### 4.控制器级绑定

```ts
import { Controller, UseFilters } from '@nestjs/common';

@Controller('users')
@UsePipes(new ValidationPipe({
  transform: true,
  whitelist: true
}))
export class UsersController {
  // import { Controller, UseFilters } from '@nestjs/common';

@Controller('users')
@UsePipes(new ValidationPipe({
  transform: true,
  whitelist: true
}))
export class UsersController {
  // 
```

## 管道的高级特性（提高部分）

### 管道的组合使用

```ts
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class LogPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    console.log(`[LogPipe] 参数值: ${value}`);
    return value;
  }
}
```

```ts
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class TrimPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    if (typeof value === 'string') {
      return value.trim();
    }
    return value;
  }
}
```

```ts
import { PipeTransform, Injectable, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParseMobilePipe implements PipeTransform {
  transform(value: any) {
    if (!/^1[3-9]\d{9}$/.test(value)) {
      throw new BadRequestException('手机号格式错误');
    }
    return value;
  }
}
```

使用以上的管道组合使用：

```ts
import { Controller, Post, Body, UsePipes } from '@nestjs/common';
import { ValidationPipe } from '@nestjs/common';
import { LogPipe } from '../pipes/log.pipe';
import { TrimPipe } from '../pipes/trim.pipe';
import { ParseMobilePipe } from '../pipes/parse-mobile.pipe';

@Controller('users')
export class UsersController {
  // 方式1: 装饰器组合
  @Post('register')
  @UsePipes(LogPipe)
  @UsePipes(TrimPipe)
  @UsePipes(ValidationPipe)
  register(@Body() createUserDto: CreateUserDto) {
    return createUserDto;
  }

  // 方式2: 参数级别组合
  @Post('validate-mobile')
  validateMobile(
    @Body('mobile', new LogPipe(), new TrimPipe(), new ParseMobilePipe())
    mobile: string,
  ) {
    return { mobile };
  }

  // 方式3: 混合使用
  @Post('create')
  @UsePipes(ValidationPipe)
  create(
    @Body() createUserDto: CreateUserDto,
    @Body('mobile', ParseMobilePipe) mobile: string,
    @Body('email', TrimPipe) email: string,
  ) {
    return { ...createUserDto, mobile, email };
  }
}
```

```ts
import { IsString, IsEmail, IsNotEmpty } from 'class-validator';

export class CreateUserDto {
  @IsNotEmpty()
  @IsString()
  username: string;

  @IsEmail()
  email: string;

  @IsString()
  mobile: string;
}
```

使用场景

1. 基础数据验证 + 业务规则验证
2. 数据转换 + 日志记录
3. 多重格式化处理
4. 复杂参数校验

### 管道的性能优化

在大型应用中，管道的频繁使用可能会影响整体性能。优化管道可以提高请求处理速度，减少服务器负载，提升用户体验。

#### 常见的性能优化策略

1. 减少不必要的管道使用
   - 只在需要的地方使用管道，避免全局绑定不必要的管道。

2. 使用轻量级管道
   - 尽量使用简单、轻量级的管道，避免复杂的逻辑处理。


3. 缓存结果
   - 对于重复计算的结果，可以使用缓存机制。


4. 异步处理
   - 使用异步管道处理耗时操作，避免阻塞主线程。


#### 代码示例和最佳实践

1. 减少不必要的管道使用

   ```ts
   import { ValidationPipe } from '@nestjs/common';
   
   async function bootstrap() {
     const app = await NestFactory.create(AppModule);
     // 只在需要的地方使用全局管道
     app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
     await app.listen(3000);
   }
   bootstrap();
   ```

2. 使用轻量级管道

   ```ts
   import { PipeTransform, Injectable } from '@nestjs/common';
   
   @Injectable()
   export class SimpleLoggerPipe implements PipeTransform {
     transform(value: any) {
       console.log('Processing value:', value);
       return value;
     }
   }
   ```

3. 缓存结果

   ```ts
   import { PipeTransform, Injectable } from '@nestjs/common';
   
   @Injectable()
   export class CachePipe implements PipeTransform {
     private cache = new Map();
   
     transform(value: any) {
       if (this.cache.has(value)) {
         return this.cache.get(value);
       }
       const result = this.processValue(value);
       this.cache.set(value, result);
       return result;
     }
   
     private processValue(value: any) {
       // 模拟复杂计算
       return value * 2;
     }
   }
   ```

4. 异步处理

   ```ts
   import { PipeTransform, Injectable } from '@nestjs/common';
   
   @Injectable()
   export class AsyncPipe implements PipeTransform {
     async transform(value: any) {
       const result = await this.asyncProcess(value);
       return result;
     }
   
     private async asyncProcess(value: any) {
       // 模拟异步操作
       return new Promise(resolve => setTimeout(() => resolve(value * 2), 100));
     }
   }
   ```

5.  综合示例

   ```ts
   import { Controller, Post, Body, UsePipes } from '@nestjs/common';
   import { ValidationPipe } from '@nestjs/common';
   import { SimpleLoggerPipe } from '../pipes/simple-logger.pipe';
   import { CachePipe } from '../pipes/cache.pipe';
   import { AsyncPipe } from '../pipes/async.pipe';
   
   @Controller('users')
   export class UsersController {
     @Post('create')
     @UsePipes(ValidationPipe, SimpleLoggerPipe, CachePipe, AsyncPipe)
     create(@Body() createUserDto: CreateUserDto) {
       return createUserDto;
     }
   }
   ```

   最佳实践:

   1. **按需使用管道**：避免全局绑定不必要的管道。
   2. **轻量级优先**：尽量使用简单的管道，减少复杂逻辑。
   3. **缓存机制**：对重复计算的结果进行缓存。
   4. **异步处理**：耗时操作使用异步管道，避免阻塞主线程。

## 与其他功能的协同（补充知识）

### 1.管道与 DTO 的配合使用

- DTO（数据传输对象）用于定义请求数据的结构
- 管道用于验证和转换 DTO

```ts
import { IsString, IsEmail, IsNotEmpty } from 'class-validator';

export class CreateUserDto {
  @IsNotEmpty()
  @IsString()
  username: string;

  @IsEmail()
  email: string;

  @IsNotEmpty()
  @IsString()
  password: string;
}
```

```ts
import { Controller, Post, Body, UsePipes, ValidationPipe } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  @Post()
  @UsePipes(new ValidationPipe())
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }
}
```

### 2.管道与异常过滤器的配合

- 管道用于验证和转换数据
- 异常过滤器用于捕获和处理异常

```ts
import { ExceptionFilter, Catch, ArgumentsHost, BadRequestException } from '@nestjs/common';

@Catch(BadRequestException)
export class BadRequestExceptionFilter implements ExceptionFilter {
  catch(exception: BadRequestException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      message: exception.message,
    });
  }
}
```

```ts
import { Controller, Post, Body, UsePipes, ValidationPipe, UseFilters } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';
import { BadRequestExceptionFilter } from './filters/bad-request-exception.filter';

@Controller('users')
@UseFilters(BadRequestExceptionFilter)
export class UsersController {
  @Post()
  @UsePipes(new ValidationPipe())
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }
}
```

### 3.管道与拦截器的协同工作

- 管道用于验证和转换数据
- 拦截器用于在请求前后进行额外处理

```ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(map(data => ({ data })));
  }
}
```

```ts
import { Controller, Post, Body, UsePipes, ValidationPipe, UseInterceptors } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';
import { TransformInterceptor } from './interceptors/transform.interceptor';

@Controller('users')
@UseInterceptors(TransformInterceptor)
export class UsersController {
  @Post()
  @UsePipes(new ValidationPipe())
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }
}
```

### 4.管道与守卫的配合

- 管道用于验证和转换数据
- 守卫用于权限验证

```ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return user && user.roles && user.roles.includes('admin');
  }
}
```

```ts
import { Controller, Post, Body, UsePipes, ValidationPipe, UseGuards } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';
import { RolesGuard } from './guards/roles.guard';

@Controller('users')
@UseGuards(RolesGuard)
export class UsersController {
  @Post()
  @UsePipes(new ValidationPipe())
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }
}
```
