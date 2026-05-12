---
tags:
  - Node.js
  - TypeScript
  - tsconfig
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# TypeScript 与 Node

## What — 是什么

> TypeScript 与 Node.js 的结合是现代服务端 JavaScript 开发的标准实践，通过静态类型检查在编译期捕获错误，提升代码可维护性和团队协作效率。

**核心概念：**

- **tsconfig.json**：TypeScript 项目配置文件，定义编译选项、文件范围、路径映射等
- **@types/node**：Node.js API 的类型声明包，提供 `fs`/`http`/`path` 等模块的类型定义
- **路径映射（Path Aliases）**：`paths` 配置将 `@/utils` 映射到 `src/utils`，简化导入路径
- **声明文件（.d.ts）**：描述 JavaScript 模块的类型信息，让 TS 项目使用无类型的 JS 库
- **ESM + CJS 混用**：Node.js 项目同时支持 ESM 和 CJS 的双模式发布
- **tsx / ts-node**：开发时直接运行 TypeScript 文件的工具，tsx 基于 esbuild 更快
- **tsc**：TypeScript 编译器，将 `.ts` 编译为 `.js`
- **SWC**：Rust 编写的超快 TypeScript/JavaScript 编译器，替代 tsc 用于开发构建
- **运行时类型验证（zod）**：Schema 验证库，可与 TS 类型双向推导，弥补 TS 运行时不检查类型的不足

**关键特性：**

- `strict: true` 启用所有严格检查（strictNullChecks/noImplicitAny 等）
- `moduleResolution: "bundler"` 适配现代打包工具的模块解析
- 条件导出（conditional exports）在 `package.json` 中为 ESM/CJS 提供不同入口
- `declaration: true` 生成 `.d.ts` 声明文件供其他项目使用
- `isolatedModules: true` 确保每个文件可独立转译（兼容 esbuild/SWC）

## Why — 为什么

**适用场景：**

- 大型 Node.js 项目：类型检查防止重构引入 bug
- 库/SDK 开发：声明文件让使用者获得类型提示
- 团队协作：类型即文档，接口契约明确
- 微服务项目：多个服务共享类型定义

**对比开发方式：**

| 维度 | 纯 JavaScript | TypeScript | JSDoc |
|------|-------------|-----------|-------|
| 类型安全 | 无 | 编译期完整检查 | 部分（编辑器提示） |
| 开发体验 | 无自动补全 | 完整自动补全+重构 | 有限自动补全 |
| 学习成本 | 无 | 中（类型系统复杂） | 低（注释语法） |
| 构建步骤 | 无 | 需编译（tsc/SWC） | 无 |
| 运行时检查 | 需手动 | 需 zod 等库补充 | 无 |
| 生态支持 | 全部 | 绝大多数（@types） | 部分 |

**优缺点：**

- ✅ 优点：
  - 编译期捕获类型错误，减少运行时 bug
  - IDE 自动补全和重构支持极好
  - 类型即文档，接口契约清晰
  - 重构安全——类型变更会级联报错
- ❌ 缺点：
  - 学习曲线陡峭（泛型/条件类型/映射类型）
  - 增加构建步骤和编译时间
  - 类型系统复杂度可能过度设计
  - 运行时无类型检查，需额外库补充

## How — 怎么用

### 快速上手

```bash
# 初始化 TS 项目
npm init -y
npm install --save-dev typescript @types/node tsx
npx tsc --init
```

### 代码示例

**tsconfig Node 项目配置：**

```jsonc
// tsconfig.json — Node.js 后端项目推荐配置
{
    "compilerOptions": {
        "target": "ES2022",                    // Node.js 18+ 支持
        "module": "NodeNext",                  // 使用 Node.js 原生模块解析
        "moduleResolution": "NodeNext",        // 匹配 module 设置
        "outDir": "./dist",                    // 编译输出目录
        "rootDir": "./src",                    // 源码根目录
        "declaration": true,                   // 生成 .d.ts 声明文件
        "declarationMap": true,                // 声明文件映射（调试用）
        "sourceMap": true,                     // 生成 source map
        "strict": true,                        // 启用所有严格检查
        "esModuleInterop": true,               // CJS/ESM 互操作
        "skipLibCheck": true,                  // 跳过 .d.ts 检查（加速编译）
        "forceConsistentCasingInFileNames": true,
        "resolveJsonModule": true,             // 允许导入 JSON
        "isolatedModules": true,               // 兼容 esbuild/SWC
        "paths": {                             // 路径映射
            "@/*": ["./src/*"]
        }
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

**声明文件编写：**

```typescript
// src/types/express.d.ts — 扩展 Express Request 类型
declare namespace Express {
    interface Request {
        user?: {
            id: string;
            name: string;
            role: 'admin' | 'user';
        };
    }
}

// src/types/env.d.ts — 环境变量类型声明
interface ProcessEnv {
    NODE_ENV: 'development' | 'production' | 'test';
    PORT: string;
    DATABASE_URL: string;
    REDIS_URL: string;
    JWT_SECRET: string;
}

// src/types/custom-module.d.ts — 为无类型的 JS 模块写声明
declare module 'some-untyped-lib' {
    interface Options {
        timeout: number;
        retries: number;
    }
    export function init(options: Options): void;
    export function process(data: string): Promise<string>;
}

// src/types/utility.d.ts — 通用工具类型
type Nullable<T> = T | null;
type Optional<T> = T | undefined;
type AsyncReturnType<T extends (...args: any) => Promise<any>> =
    T extends (...args: any) => Promise<infer R> ? R : never;
```

**ESM + CJS 双模式发布：**

```jsonc
// package.json — 双模式发布配置
{
    "name": "my-lib",
    "version": "1.0.0",
    "type": "module",                          // 默认 ESM
    "exports": {
        ".": {
            "import": {                        // ESM 入口
                "types": "./dist/esm/index.d.ts",
                "default": "./dist/esm/index.js"
            },
            "require": {                       // CJS 入口
                "types": "./dist/cjs/index.d.ts",
                "default": "./dist/cjs/index.js"
            }
        }
    },
    "files": ["dist"],
    "scripts": {
        "build:cjs": "tsc -p tsconfig.cjs.json",
        "build:esm": "tsc -p tsconfig.esm.json",
        "build": "npm run build:cjs && npm run build:esm"
    }
}

// tsconfig.cjs.json
{
    "extends": "./tsconfig.json",
    "compilerOptions": {
        "module": "CommonJS",
        "moduleResolution": "Node",
        "outDir": "./dist/cjs"
    }
}

// tsconfig.esm.json
{
    "extends": "./tsconfig.json",
    "compilerOptions": {
        "module": "NodeNext",
        "moduleResolution": "NodeNext",
        "outDir": "./dist/esm"
    }
}
```

**运行时类型验证（zod）：**

```typescript
import { z } from 'zod';

// 定义 Schema，自动推导 TS 类型
const UserSchema = z.object({
    id: z.string().uuid(),
    name: z.string().min(1).max(100),
    email: z.string().email(),
    age: z.number().int().min(0).max(150).optional(),
    role: z.enum(['admin', 'user', 'moderator']).default('user'),
    createdAt: z.date().default(() => new Date())
});

// 从 Schema 推导 TypeScript 类型
type User = z.infer<typeof UserSchema>;
// 等价于: { id: string; name: string; email: string; age?: number; role: 'admin' | 'user' | 'moderator'; createdAt: Date }

// Express 路由中使用
import { Request, Response, NextFunction } from 'express';

function validate<T extends z.ZodType>(schema: T) {
    return (req: Request, res: Response, next: NextFunction) => {
        const result = schema.safeParse(req.body);
        if (!result.success) {
            return res.status(400).json({
                error: 'Validation failed',
                details: result.error.flatten()
            });
        }
        req.body = result.data; // 替换为验证后的数据
        next();
    };
}

app.post('/users',
    validate(UserSchema.pick({ name: true, email: true, age: true })),
    async (req: Request, res: Response) => {
        // req.body 类型自动推导为 { name: string; email: string; age?: number }
        const user = await createUser(req.body);
        res.status(201).json(user);
    }
);

// 环境变量验证
const EnvSchema = z.object({
    NODE_ENV: z.enum(['development', 'production', 'test']),
    PORT: z.coerce.number().default(3000),
    DATABASE_URL: z.string().url(),
    JWT_SECRET: z.string().min(32)
});

const env = EnvSchema.parse(process.env); // 验证失败直接抛错
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `Cannot find module` 错误 | 缺少 @types 包或路径映射未生效 | 安装 `@types/xxx`，检查 `paths` 和 `rootDir` |
| ESM import CJS 报错 | CJS 默认导出在 ESM 中需 `.default` | 用 `esModuleInterop` 或解构 `{ default }` |
| `ts-node` 启动慢 | 每次启动都需类型检查 | 换用 `tsx`（基于 esbuild，快 10 倍+） |
| 运行时类型与编译时不一致 | TS 编译后类型擦除，运行时无检查 | 用 zod 在运行时验证边界数据 |
| `paths` 映射运行时不生效 | tsc 编译后 `@/` 路径不转换 | 用 `tsc-alias` 或 `tsconfig-paths` |
| `.d.ts` 不生效 | 声明文件不在 `include` 范围 | 将 `src/types` 加入 `include` 或用 `typeRoots` |
| 枚举编译后体积大 | `const enum` 在隔离模块下不可用 | 用 `as const` 对象替代 enum |
| `any` 类型泄漏 | 旧代码或第三方库引入 any | `noImplicitAny` + `strict` 强制标注 |

### 最佳实践

- 新项目直接用 TypeScript，不要"后期迁移"
- `tsconfig.json` 必须开启 `strict: true`
- 开发时用 `tsx` 运行，生产构建用 `tsc` 或 `SWC`
- 用 zod 在 API 边界（请求体/环境变量/外部数据）做运行时验证
- 路径映射 `@/` 简化导入，配合 `tsc-alias` 处理编译路径
- 声明文件统一放在 `src/types/` 目录
- 库项目使用条件导出支持 ESM + CJS 双模式
- 避免 `any`，用 `unknown` 替代并逐步收窄类型

## 面试题

**Q1: tsconfig.json 中最关键的配置项有哪些？**
> 关键配置：① `strict: true`——启用所有严格检查（noImplicitAny/strictNullChecks 等），是最重要的配置；② `target`——编译目标（ES2022 适合 Node.js 18+）；③ `module`/`moduleResolution`——模块系统（`NodeNext` 适配 ESM 项目，`CommonJS`/`Node` 适配 CJS 项目）；④ `outDir`/`rootDir`——编译输出和源码根目录；⑤ `declaration: true`——生成 `.d.ts` 供其他项目使用；⑥ `paths`——路径别名映射；⑦ `isolatedModules: true`——确保兼容 esbuild/SWC 单文件编译。

**Q2: @types/node 的工作原理是什么？**
> `@types/node` 是 DefinitelyTyped 社区维护的 Node.js API 类型声明包。TypeScript 编译器在解析模块时会自动查找 `node_modules/@types/` 目录下的声明文件。当你在代码中 `import fs from 'fs'`，TS 通过 `@types/node/fs.d.ts` 获取 `fs` 模块的所有类型信息。`typeRoots` 配置控制 TS 查找声明文件的目录（默认 `node_modules/@types`），`types` 配置限定只加载指定的 `@types` 包。

**Q3: .d.ts 声明文件的作用是什么？如何编写？**
> 声明文件（`.d.ts`）描述 JavaScript 模块的类型信息，让 TypeScript 项目使用无类型的 JS 库时获得类型提示和检查。编写方式：① 全局声明——`declare function foo(x: number): string;` 直接在全局可用；② 模块声明——`declare module 'lib-name' { export function foo(x: number): string; }` 为特定模块提供类型；③ 三斜线指令——`/// <reference types="node" />` 引入其他声明文件。发布时在 `package.json` 中用 `types`/`typings` 字段指向声明文件。

**Q4: ESM 与 CJS 混用时有哪些常见问题？**
> 主要问题：① ESM 中 `import` CJS 模块时，CJS 的 `module.exports` 被包装为 `{ default }`，需 `esModuleInterop` 或手动解构；② CJS 中 `require()` ESM 模块不可用（ESM 是异步加载），需用动态 `import()`；③ `__dirname`/`__require` 在 ESM 中不存在，需用 `import.meta.url` + `fileURLToPath` 替代；④ 文件扩展名：ESM 的 `import` 必须带 `.js` 扩展名（即使源码是 `.ts`）；⑤ 双模式发布需要两套 tsconfig 和 `package.json` 的 `exports` 条件导出。

**Q5: ts-node 和 tsx 有什么区别？**
> `ts-node` 是 TypeScript 官方的 REPL/运行器，通过注册钩子在运行时编译 TS 代码。它使用 tsc 编译，支持完整的类型检查，但启动慢（冷启动 2-5 秒）。`tsx` 是基于 esbuild 的运行器，只做语法转译不做类型检查，启动极快（冷启动 < 200ms）。开发时推荐 `tsx`——类型检查交给编辑器 + `tsc --noEmit`；生产构建用 `tsc` 编译。`ts-node` 的 `--transpile-only` 模式可跳过类型检查加速，但仍不如 esbuild 快。

**Q6: 运行时类型验证方案有哪些？为什么需要？**
> TypeScript 编译后类型信息被擦除，运行时无法验证外部数据（HTTP 请求体/环境变量/数据库查询结果）的格式。方案：① zod——Schema 定义 + TS 类型推导，最流行，支持 `z.infer<>` 自动推导类型；② joi——老牌验证库，不支持 TS 类型推导；③ class-validator——装饰器风格，适合 NestJS；④ io-ts——函数式风格，基于 fp-ts；⑤ TypeBox——JSON Schema 风格，轻量。推荐 zod：一套 Schema 同时提供编译时类型和运行时验证。

**Q7: 路径映射（path aliases）如何配置？运行时怎么处理？**
> `tsconfig.json` 中 `paths` 配置：`"@/*": ["./src/*"]`，让 `import { x } from '@/utils'` 映射到 `./src/utils`。但 tsc 编译后 JS 文件中 `@/utils` 不会被转换为相对路径，Node.js 运行时会报 `Cannot find module`。解决方案：① `tsc-alias`——编译后替换路径别名为相对路径（推荐）；② `tsconfig-paths`——运行时注册钩子解析路径（有性能开销）；③ `module-alias`——运行时路径映射；④ 打包工具（esbuild/tsup）自动处理路径。

**Q8: TypeScript 项目在 Monorepo 中如何配置？**
> Monorepo 推荐使用 Project References：① 根 `tsconfig.json` 设置 `composite: true` + `references: [{ path: "packages/core" }]`；② 每个 package 有独立 `tsconfig.json`，通过 `references` 声明依赖关系；③ `tsc --build` 增量编译，只重新编译变更的包；④ 共享类型放在 `packages/types`，其他包通过 `references` 引用；⑤ `paths` 映射各包路径，开发时无需 `npm link`。用 Turborepo/Nx 编排构建顺序，确保依赖包先编译。

---

**相关链接：**
- [[模块系统]]
- [[npm与包管理]]
- [[NestJS]]
- TypeScript Handbook：https://www.typescriptlang.org/docs/handbook/
