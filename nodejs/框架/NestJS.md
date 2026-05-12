---
tags:
  - Node.js
  - NestJS
  - 企业级框架
date: 2026-05-12
status: 已完成
difficulty: 进阶
---

# NestJS

## What — 是什么

> NestJS 是一个用于构建高效、可扩展的 Node.js 服务端应用的企业级框架，采用 TypeScript 开发，融合了 Angular 的架构理念（装饰器/依赖注入/模块化），底层可搭配 Express 或 Fastify。

**核心概念：**

- **装饰器（Decorator）**：`@Controller`/`@Get`/`@Post`/`@Injectable`/`@Module` 等声明式元数据标注，替代手动注册和路由配置
- **依赖注入（DI）**：IoC 容器自动管理依赖关系，`@Injectable()` 标注的 Provider 由容器创建和注入
- **模块化（Module）**：`@Module()` 组织应用结构，每个模块封装相关的控制器、服务和提供者
- **守卫（Guard）**：`@Injectable()` + `CanActivate` 接口，决定请求是否被允许（认证/授权/角色检查）
- **管道（Pipe）**：`@Injectable()` + `PipeTransform` 接口，数据转换和验证（`ValidationPipe` + class-validator）
- **拦截器（Interceptor）**：`@Injectable()` + `NestInterceptor` 接口，请求前后执行逻辑（日志/缓存/响应转换）
- **微服务支持**：内置 TCP/Redis/MQTT/NATS/gRPC 等传输层，同一套代码支持 HTTP 和微服务

**核心架构：**

- 设计理念：Angular 风格 + 企业级可扩展。装饰器声明式编程减少样板代码，DI 解耦依赖关系，模块化清晰划分业务边界
- 核心模块：**Modules**（组织结构）、**Controllers**（路由处理）、**Providers**（业务逻辑/服务）、**Middleware**（中间件）、**Guards**（守卫）、**Pipes**（管道）、**Interceptors**（拦截器）、**Filters**（异常过滤器）
- 数据流：请求 → Middleware → Guard → Interceptor(before) → Pipe(validation) → Controller → Service → Interceptor(after) → Response

**插件生态：**

- 官方：`@nestjs/microservices`、`@nestjs/websockets`、`@nestjs/graphql`、`@nestjs/typeorm`、`@nestjs/prisma`、`@nestjs/passport`、`@nestjs/jwt`、`@nestjs/config`、`@nestjs/swagger`、`@nestjs/schedule`、`@nestjs/bull`、`@nestjs/event-emitter`
- 社区：`@nestjs/terminus`（健康检查）、`@nestjs/throttler`（限流）、`@nestjs/cqrs`（CQRS 模式）

## Why — 为什么

**适用场景：**

- 企业级 API 服务：大型团队协作，需要清晰的架构和规范
- 微服务架构：同一套代码支持 HTTP + 多种传输协议
- GraphQL 服务：`@nestjs/graphql` 一等公民支持
- 需要 DI 和强类型的项目：TypeScript 原生，类型安全

**对比企业级框架：**

| 维度 | NestJS | Express + 自组织 | LoopBack |
|------|--------|----------------|----------|
| 架构约束 | 强（模块化/DI/装饰器） | 无（自行设计） | 中（Model驱动） |
| 学习曲线 | 高（需理解 DI/装饰器） | 低 | 中 |
| TypeScript | 一等公民 | 需手动配置 | 支持 |
| 微服务 | 内置多传输层 | 需自建 | 有限 |
| 代码规范 | 框架级统一 | 因团队而异 | 框架级 |
| 灵活性 | 中（约定优于配置） | 高 | 中 |

**优缺点：**

- ✅ 优点：
  - 清晰的模块化架构，大项目不混乱
  - DI 容器自动管理依赖，易于测试和替换
  - 装饰器声明式编程减少样板代码
  - 内置微服务、GraphQL、WebSocket 支持
  - 完善的 TypeScript 支持
- ❌ 缺点：
  - 学习曲线陡峭（装饰器/DI/模块/管道等概念多）
  - 框架约束强，小项目显得重
  - 装饰器元数据增加运行时开销
  - 底层仍是 Express，性能受其限制

## How — 怎么用

### 快速上手

```bash
npm i -g @nestjs/cli
nest new my-project
cd my-project
npm run start:dev
```

```typescript
// app.controller.ts
import { Controller, Get } from '@nestjs/common';

@Controller('users')
export class UsersController {
    constructor(private readonly usersService: UsersService) {}

    @Get()
    findAll(): Promise<User[]> {
        return this.usersService.findAll();
    }

    @Get(':id')
    findOne(@Param('id') id: string): Promise<User> {
        return this.usersService.findOne(+id);
    }

    @Post()
    create(@Body() createUserDto: CreateUserDto): Promise<User> {
        return this.usersService.create(createUserDto);
    }
}

// app.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
    constructor(@InjectRepository(User) private repo: Repository<User>) {}

    findAll(): Promise<User[]> { return this.repo.find(); }
    findOne(id: number): Promise<User> { return this.repo.findOneBy({ id }); }
    create(dto: CreateUserDto): Promise<User> { return this.repo.save(dto); }
}

// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
    imports: [TypeOrmModule.forFeature([User])],
    controllers: [UsersController],
    providers: [UsersService],
    exports: [UsersService],
})
export class UsersModule {}
```

### 代码示例

**Guard 守卫 + JWT 认证：**

```typescript
// auth.guard.ts
import { Injectable, CanActivate, ExecutionContext, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { Reflector } from '@nestjs/core';

@Injectable()
export class AuthGuard implements CanActivate {
    constructor(private jwt: JwtService, private reflector: Reflector) {}

    async canActivate(context: ExecutionContext): Promise<boolean> {
        const isPublic = this.reflector.get<boolean>('isPublic', context.getHandler());
        if (isPublic) return true;

        const request = context.switchToHttp().getRequest();
        const token = request.headers.authorization?.replace('Bearer ', '');
        if (!token) throw new UnauthorizedException();

        try {
            const payload = await this.jwt.verifyAsync(token);
            request.user = payload;
            return true;
        } catch {
            throw new UnauthorizedException();
        }
    }
}

// roles.guard.ts
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
    constructor(private reflector: Reflector) {}

    canActivate(context: ExecutionContext): boolean {
        const requiredRoles = this.reflector.get<string[]>('roles', context.getHandler());
        if (!requiredRoles) return true;

        const { user } = context.switchToHttp().getRequest();
        if (!requiredRoles.includes(user.role)) {
            throw new ForbiddenException('Insufficient permissions');
        }
        return true;
    }
}

// 装饰器
import { SetMetadata } from '@nestjs/common';
export const Public = () => SetMetadata('isPublic', true);
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

// 使用
@Controller('admin')
@UseGuards(AuthGuard, RolesGuard)
export class AdminController {
    @Get('dashboard')
    @Roles('admin')
    getDashboard() { return { data: 'admin only' }; }
}
```

**Pipe 验证 + Interceptor 日志：**

```typescript
// validation.pipe.ts — 全局验证管道
import { ValidationPipe } from '@nestjs/common';

app.useGlobalPipes(new ValidationPipe({
    whitelist: true,           // 去掉未装饰的属性
    forbidNonWhitelisted: true, // 有未装饰属性时报错
    transform: true,           // 自动类型转换（字符串→数字）
}));

// dto
import { IsString, IsEmail, IsInt, Min } from 'class-validator';

export class CreateUserDto {
    @IsString() name: string;
    @IsEmail() email: string;
    @IsInt() @Min(0) age?: number;
}

// logging.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler, Logger } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
    private readonly logger = new Logger('HTTP');

    intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        const request = context.switchToHttp().getRequest();
        const { method, url } = request;
        const now = Date.now();

        return next.handle().pipe(
            tap(() => {
                this.logger.log(`${method} ${url} - ${Date.now() - now}ms`);
            })
        );
    }
}

// 响应转换拦截器
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, { data: T }> {
    intercept(context: ExecutionContext, next: CallHandler): Observable<{ data: T }> {
        return next.handle().pipe(map(data => ({ data })));
    }
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 循环依赖 | 模块间互相 import 导致 | 使用 `forwardRef()` 或提取共享模块 |
| Provider 注入为 undefined | 未在 Module 中注册或作用域不对 | 检查 `providers` 数组，确保 `@Injectable()` |
| DTO 验证不生效 | 未配置全局 ValidationPipe | `app.useGlobalPipes(new ValidationPipe())` |
| 守卫不执行 | 未在 Controller/方法上挂载 | `@UseGuards(AuthGuard)` |
| 热重载丢失状态 | 默认 HRM 不保持 DI 容器 | 开发环境用 `webpack-hmr` 配置 |
| 全局 Provider 无法注入 | `useGlobalPipes/Guards` 不走 DI | 用模块注册 `APP_GUARD`/`APP_PIPE` token |

### 最佳实践

- 按业务领域划分模块，每个模块独立封装
- 使用 `class-validator` + `ValidationPipe` 统一验证
- 认证用 Guard + JWT，不手动检查每个路由
- Interceptor 统一处理日志、缓存、响应格式
- 全局 Provider 通过模块注册（`APP_GUARD`/`APP_INTERCEPTOR`）确保 DI
- 善用 CLI 生成代码：`nest g module/users --no-spec`

## 面试题

**Q1: NestJS 的 DI 容器如何工作？**
> NestJS 的 IoC 容器基于 TypeScript 装饰器元数据实现依赖解析。流程：① 启动时扫描所有 `@Module()` 注册的 `providers`；② 根据 `@Injectable()` 装饰器收集类型信息；③ 按依赖拓扑排序创建实例——如果 `UsersService` 依赖 `Repository`，先创建 `Repository` 再注入到 `UsersService`；④ 将创建的实例缓存在容器中（默认单例）。`@Inject()` 用于注入非类类型的依赖（如字符串 token）。循环依赖通过 `forwardRef()` 延迟解析解决。

**Q2: NestJS 中 Guard、Middleware、Interceptor、Pipe 的区别？**
> 执行顺序和职责：① **Middleware**（最先执行）——类似 Express 中间件，可访问 req/res，适合日志、CORS；② **Guard**（认证授权）——返回 boolean 决定是否继续，基于角色/权限；③ **Interceptor**（前后逻辑）——用 RxJS `tap/map` 在 Handler 前后执行，适合日志/缓存/响应转换；④ **Pipe**（数据转换验证）——处理参数，转换类型或验证格式，失败抛 BadRequestException。关键区别：Guard 关注"能不能执行"，Pipe 关注"数据对不对"，Interceptor 关注"执行前后的增强"，Middleware 最通用但不在 DI 容器中。

**Q3: NestJS 的模块系统如何组织？全局模块 vs 功能模块？**
> 每个应用至少有一个根模块（`AppModule`）。功能模块封装业务：`controllers`（路由）、`providers`（服务）、`imports`（依赖的其他模块）、`exports`（对外暴露的 Provider）。全局模块用 `@Global()` 装饰，注册后所有模块无需 `imports` 即可使用其 Provider（如 `ConfigModule`、`DatabaseModule`）。模块默认是单例——同一模块在不同地方 `import` 共享同一组 Provider 实例。

**Q4: NestJS 如何支持微服务？**
> `@nestjs/microservices` 提供传输层抽象。创建微服务：`const app = await NestFactory.createMicroservice(AppModule, { transport: Transport.TCP })`。支持的传输层：TCP、Redis、MQTT、NATS、RabbitMQ、Kafka、gRPC。通信模式：① 消息模式——`@MessagePattern('cmd')` 请求/响应；② 事件模式——`@EventPattern('evt')` 触发即忘。混合模式：`const app = await NestFactory.create(AppModule); app.connectMicroservice({...});` 同时提供 HTTP 和微服务端点。同一套 Controller/Service 代码，切换传输层只需改配置。

**Q5: NestJS 如何实现 JWT 认证？**
> 步骤：① 安装 `@nestjs/jwt` + `@nestjs/passport` + `passport-jwt`；② 创建 AuthModule 注册 JWT 和 Passport；③ 实现 `JwtStrategy`（继承 `PassportStrategy(Strategy)`），定义如何从 token 提取用户；④ 创建 `AuthGuard('jwt')` 守卫；⑤ 在路由上 `@UseGuards(AuthGuard('jwt'))`。登录流程：Controller 接收用户名密码 → Service 验证 → `jwtService.sign()` 生成 token → 返回给客户端。后续请求带 `Authorization: Bearer <token>`，Guard 自动验证。

**Q6: ValidationPipe 和 class-validator 如何配合？**
> `class-validator` 提供装饰器定义验证规则（`@IsString()`/`@IsEmail()`/`@Min()`），`ValidationPipe` 是 NestJS 内置管道，在请求到达 Handler 前自动用 class-validator 验证 DTO。配置 `whitelist: true` 自动去掉 DTO 未定义的属性（防止批量赋值攻击），`transform: true` 自动将字符串转为 DTO 定义的类型（URL 参数 `"123"` → 数字 `123`），`forbidNonWhitelisted: true` 拒绝包含额外字段的请求。

**Q7: NestJS 的拦截器（Interceptor）能做什么？**
> 四种用途：① 响应转换——`map(data => ({ data }))` 统一响应格式；② 日志记录——`tap()` 记录请求耗时和方法；③ 缓存——检查缓存有则直接返回，无则执行 Handler 并缓存结果；④ 超时控制——`timeout(5000)` 配合 `throwIfEmpty()` 限制 Handler 执行时间。拦截器基于 RxJS 操作符，`next.handle()` 返回 Observable，可用 `map/tap/catchError/timeout` 等操作。拦截器可作用于单个路由、整个 Controller 或全局。

**Q8: 从 Express 迁移到 NestJS 的要点？**
> 迁移要点：① 路由 → Controller + 装饰器（`@Get`/`@Post` 替代 `app.get`/`app.post`）；② 中间件逻辑拆分到 Guard（认证）/Pipe（验证）/Interceptor（日志/缓存）；③ 业务逻辑从路由回调提取到 `@Injectable()` Service；④ 全局错误处理用 `@Catch()` ExceptionFilter 替代 `app.use((err, req, res, next) => {})`；⑤ 配置用 `@nestjs/config` 的 `ConfigModule` 替代 `dotenv`；⑥ 数据库用 `@nestjs/typeorm` 或 `@nestjs/prisma` 集成。渐进迁移：可先用 `@nestjs/express` 在 NestJS 中桥接 Express 中间件。

---

**相关链接：**
- [[Express]]
- [[Fastify]]
- [[TypeScript与Node]]
- [[认证与授权]]
- NestJS 官方文档：https://docs.nestjs.com/
