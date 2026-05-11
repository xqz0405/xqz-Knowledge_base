---
tags:
  - Web前端
  - Angular
  - TypeScript
  - 框架
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Angular 核心

## What — 什么是 Angular

Angular 是 Google 维护的企业级前端框架，使用 TypeScript 开发，提供完整的开发平台——路由、表单、HTTP 客户端、测试工具、CLI 等全部内置。它是三大框架中最"大而全"的，也是对 TypeScript 支持最原生的。

### 核心概念

| 概念 | 说明 |
|------|------|
| Component | 组件 = 模板 + 类 + 样式 |
| Template | HTML 模板，支持数据绑定和指令 |
| Directive | 指令，改变 DOM 行为或外观 |
| Service | 服务，封装业务逻辑和数据访问 |
| Dependency Injection | 依赖注入，自动管理服务实例 |
| Module (NgModule) | 模块，组织应用结构（Angular 17+ 可选） |
| Signal | 信号，Angular 16+ 的新响应式原语 |
| Standalone | 独立组件，无需 NgModule（Angular 14+） |

### 与 React/Vue 的对比

| 维度 | Angular | React | Vue |
|------|---------|-------|-----|
| 语言 | TypeScript 原生 | JS + TS 可选 | JS + TS 可选 |
| 模板 | HTML 模板 + 指令 | JSX | HTML 模板 + 指令 |
| 响应式 | Signal / Zone.js | Hooks / setState | Proxy 响应式 |
| 依赖注入 | 内置 | 无（手动） | 无（手动） |
| 路由 | 内置 | react-router | vue-router |
| 表单 | 内置（模板驱动+响应式） | 第三方库 | vee-validate 等 |
| HTTP | 内置 | fetch / axios | fetch / axios |
| CLI | Angular CLI | CRA / Vite | Vue CLI / Vite |
| 包体积 | 较大（~130KB） | 小（~45KB） | 小（~35KB） |

---

## Why — 为什么选择 Angular

### 1. 企业级完整性

Angular 提供了构建大型应用所需的一切：路由、表单验证、HTTP 拦截器、权限守卫、国际化、SSR——全部官方维护，无需选型第三方库。

### 2. TypeScript 原生

Angular 从第一个字符就是 TypeScript，类型安全覆盖模板（通过语言服务）、服务、路由参数等所有层面。

### 3. 依赖注入

Angular 的 DI 系统让服务的创建、注入、生命周期管理自动化，特别适合大型项目中的服务编排。

### 4. 一致性

Angular 的强约定（文件命名、模块组织、编码规范）确保团队所有人写出的代码风格一致，降低协作成本。

### 优缺点

- ✅ 优点：完整平台、TypeScript 原生、依赖注入、强约定、Google 维护
- ❌ 缺点：学习曲线陡峭、包体积大、模板语法独特、Zone.js 性能开销

---

## How — 怎么用

### 1. 创建项目

```bash
npm install -g @angular/cli
ng new my-app --standalone --style=scss --routing
cd my-app
ng serve
```

### 2. Standalone Component（现代 Angular 推荐）

```ts
// user-card.component.ts
import { Component, Input, Output, EventEmitter, signal, computed } from '@angular/core'
import { CommonModule } from '@angular/common'

@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="card" [class.featured]="isFeatured()">
      <img [src]="user().avatar" [alt]="user().name" class="avatar" />
      <div class="info">
        <h3>{{ user().name }}</h3>
        <p>{{ user().email }}</p>
        <span class="badge">{{ roleLabel() }}</span>
      </div>
      <button (click)="edit.emit(user())">Edit</button>
    </div>
  `,
  styles: [`
    .card { display: flex; gap: 12px; padding: 16px; border-radius: 8px; background: white; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
    .featured { border: 2px solid #3b82f6; }
    .avatar { width: 48px; height: 48px; border-radius: 50%; }
    .info { flex: 1; }
    .badge { background: #dbeafe; color: #1d4ed8; padding: 2px 8px; border-radius: 12px; font-size: 12px; }
  `],
})
export class UserCardComponent {
  @Input() user = signal({ name: '', email: '', avatar: '', role: 'user' })
  @Input() isFeatured = signal(false)
  @Output() edit = new EventEmitter<any>()

  roleLabel = computed(() => {
    const role = this.user().role
    const labels: Record<string, string> = { admin: 'Admin', user: 'User', guest: 'Guest' }
    return labels[role] || 'Unknown'
  })
}
```

### 3. Signal — 新响应式原语

Angular 16+ 引入 Signal，替代 Zone.js 做变更检测。

```ts
import { signal, computed, effect } from '@angular/core'

// 可写信号
const count = signal(0)
const name = signal('Angular')

// 读取值
console.log(count()) // 0

// 更新值
count.set(5)            // 直接设置
count.update(v => v + 1) // 基于前值更新

// 计算信号（派生值）
const double = computed(() => count() * 2)
const greeting = computed(() => `Hello, ${name()}!`)

// 副作用
effect(() => {
  console.log(`Count changed to ${count()}`)
  // 自动追踪 count 依赖
})
```

**Signal 在组件中使用**：

```ts
@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>Count: {{ count() }}</p>
    <p>Double: {{ double() }}</p>
    <button (click)="increment()">+1</button>
    <button (click)="decrement()">-1</button>
    <button (click)="reset()">Reset</button>
  `,
})
export class CounterComponent {
  count = signal(0)
  double = computed(() => this.count() * 2)

  increment() { this.count.update(v => v + 1) }
  decrement() { this.count.update(v => v - 1) }
  reset() { this.count.set(0) }
}
```

### 4. 依赖注入

```ts
// 服务定义
import { Injectable, inject } from '@angular/core'
import { HttpClient } from '@angular/common/http'

@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient)
  private apiUrl = '/api/users'

  getUsers() {
    return this.http.get<User[]>(this.apiUrl)
  }

  getUser(id: number) {
    return this.http.get<User>(`${this.apiUrl}/${id}`)
  }

  createUser(data: CreateUserDto) {
    return this.http.post<User>(this.apiUrl, data)
  }
}
```

```ts
// 组件中使用
@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div *ngIf="loading()">Loading...</div>
    <ul *ngIf="!loading()">
      <li *ngFor="let user of users()">{{ user.name }} - {{ user.email }}</li>
    </ul>
  `,
})
export class UserListComponent implements OnInit {
  private userService = inject(UserService)

  users = signal<User[]>([])
  loading = signal(true)

  ngOnInit() {
    this.userService.getUsers().subscribe({
      next: (data) => {
        this.users.set(data)
        this.loading.set(false)
      },
      error: (err) => {
        console.error(err)
        this.loading.set(false)
      },
    })
  }
}
```

### 5. 新控制流语法（Angular 17+）

```html
<!-- @if / @else 替代 *ngIf -->
@if (user()) {
  <p>{{ user().name }}</p>
} @else if (loading()) {
  <p>Loading...</p>
} @else {
  <p>No user found</p>
}

<!-- @for 替代 *ngFor -->
@for (item of items(); track item.id) {
  <div class="item">{{ item.name }}</div>
} @empty {
  <p>No items</p>
}

<!-- @switch 替代 [ngSwitch] -->
@switch (status()) {
  @case ('active') { <span class="active">Active</span> }
  @case ('inactive') { <span class="inactive">Inactive</span> }
  @default { <span>Unknown</span> }
}
```

### 6. 路由

```ts
// app.routes.ts
import { Routes } from '@angular/router'

export const routes: Routes = [
  { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
  {
    path: 'dashboard',
    loadComponent: () => import('./pages/dashboard/dashboard.component')
      .then(m => m.DashboardComponent),
  },
  {
    path: 'users',
    children: [
      {
        path: '',
        loadComponent: () => import('./pages/user-list/user-list.component')
          .then(m => m.UserListComponent),
      },
      {
        path: ':id',
        loadComponent: () => import('./pages/user-detail/user-detail.component')
          .then(m => m.UserDetailComponent),
      },
    ],
  },
  {
    path: 'admin',
    canActivate: [authGuard],
    loadChildren: () => import('./admin/admin.routes')
      .then(m => m.ADMIN_ROUTES),
  },
]
```

```ts
// 路由守卫（函数式守卫，Angular 15+）
import { inject } from '@angular/core'
import { Router, CanActivateFn } from '@angular/router'
import { AuthService } from './auth.service'

export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService)
  const router = inject(Router)

  if (auth.isLoggedIn()) {
    return true
  }

  router.navigate(['/login'], { queryParams: { returnUrl: state.url } })
  return false
}
```

### 7. 响应式表单

```ts
import { Component, inject } from '@angular/core'
import { ReactiveFormsModule, FormBuilder, Validators } from '@angular/forms'

@Component({
  selector: 'app-register',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div>
        <label>Email</label>
        <input formControlName="email" type="email" />
        @if (email.errors?.['required'] && email.touched) {
          <span class="error">Email is required</span>
        }
        @if (email.errors?.['email'] && email.touched) {
          <span class="error">Invalid email format</span>
        }
      </div>

      <div formGroupName="passwords">
        <div>
          <label>Password</label>
          <input formControlName="password" type="password" />
        </div>
        <div>
          <label>Confirm Password</label>
          <input formControlName="confirm" type="password" />
        </div>
      </div>

      <button type="submit" [disabled]="form.invalid">Register</button>
    </form>
  `,
})
export class RegisterComponent {
  private fb = inject(FormBuilder)

  form = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    passwords: this.fb.group({
      password: ['', [Validators.required, Validators.minLength(8)]],
      confirm: ['', Validators.required],
    }, { validators: this.passwordMatch }),
  })

  get email() { return this.form.get('email')! }

  passwordMatch(group: any) {
    const password = group.get('password')?.value
    const confirm = group.get('confirm')?.value
    return password === confirm ? null : { mismatch: true }
  }

  onSubmit() {
    if (this.form.valid) {
      console.log(this.form.value)
    }
  }
}
```

### 8. HTTP 拦截器

```ts
// 认证拦截器（函数式，Angular 15+）
import { HttpInterceptorFn } from '@angular/common/http'

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = localStorage.getItem('token')

  if (token) {
    const authReq = req.clone({
      headers: req.headers.set('Authorization', `Bearer ${token}`),
    })
    return next(authReq)
  }

  return next(req)
}

// 错误处理拦截器
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        // 跳转登录
        inject(Router).navigate(['/login'])
      }
      if (error.status === 0) {
        // 网络错误
        console.error('Network error')
      }
      return throwError(() => error)
    })
  )
}
```

---

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Zone.js 性能差 | Zone.js monkey-patch 所有异步 API | 使用 Signal + `zoneless` 模式（Angular 18+） |
| 包体积大 | Angular 框架本身较大 | 按需加载路由、Tree Shaking、SSR |
| 学习曲线陡 | DI、RxJS、NgModule 概念多 | 使用 Standalone + Signal 减少概念 |
| 模板类型检查弱 | 模板中的类型错误默认不报错 | 启用 `strictTemplates` |
| RxJS 复杂 | Angular 大量使用 RxJS | 简单场景用 Signal + async/await |

### 最佳实践

1. **新项目用 Standalone + Signal**：无需 NgModule，无需 Zone.js，代码更简洁。
2. **函数式拦截器和守卫**：Angular 15+ 推荐函数式写法，更易测试。
3. **懒加载路由**：`loadComponent` / `loadChildren` 按需加载。
4. **启用 strictTemplates**：`angular.json` 中配置，模板类型错误编译时捕获。
5. **Signal 替代 RxJS**：简单状态用 Signal，复杂异步流用 RxJS。

---

## 面试题

### 1. Angular 的依赖注入（DI）是如何工作的？

**答**：Angular 的 DI 分三步：(1) **注册**——`@Injectable({ providedIn: 'root' })` 在根注入器注册服务，整个应用单例；(2) **解析**——组件/指令通过构造函数参数声明依赖，Angular 的注入器根据类型（Token）找到对应的服务实例；(3) **创建**——如果服务尚未实例化，注入器调用其构造函数创建，并递归解析其依赖。现代写法用 `inject()` 函数替代构造函数注入，更简洁：`private userService = inject(UserService)`。DI 的好处是服务的创建和生命周期完全由框架管理，组件只关心使用，不关心创建。

---

### 2. Angular Signal 和 Zone.js 的区别是什么？为什么要用 Signal 替代 Zone.js？

**答**：Zone.js 通过 monkey-patch 浏览器异步 API（setTimeout、Promise、EventTarget 等）来追踪异步操作，当任何异步回调执行后，Zone.js 通知 Angular 进行变更检测——扫描整棵组件树。问题：(1) 全局 monkey-patch 有性能开销；(2) 变更检测范围大（整棵树），即使只有一个组件变化；(3) 与第三方库集成时可能需要手动触发 `NgZone.run()`。Signal 是细粒度响应式——只有依赖 Signal 值的视图会被更新，无需扫描整棵组件树。Signal 的变更检测是同步的、确定性的，性能比 Zone.js 好 5-10 倍。

---

### 3. Standalone Component 和 NgModule 有什么区别？

**答**：NgModule 是 Angular 早期的组织方式，需要声明组件属于哪个模块、导入哪些模块、导出哪些组件。Standalone Component（Angular 14+）让组件自己声明依赖，无需 NgModule。区别：(1) 声明方式——Standalone 组件直接在 `imports` 数组中导入依赖（其他 Standalone 组件、指令、管道），NgModule 需要在 `declarations` 中声明组件；(2) 路由配置——Standalone 组件直接用于路由 `loadComponent`，无需在模块中声明；(3) 简洁性——Standalone 减少了 NgModule 这个中间层，代码量少 30-50%。Angular 17+ 新项目推荐全部使用 Standalone。

---

### 4. Angular 的变更检测策略 OnPush 是什么？什么时候应该用？

**答**：Default 策略下，Angular 在任何异步事件后检查整棵组件树；OnPush 策略下，Angular 只在以下情况检查组件：(1) `@Input()` 属性引用变化；(2) 组件内部的事件（click、submit 等）；(3) Async pipe 发出新值；(4) 手动调用 `ChangeDetectorRef.markForCheck()`。OnPush 显著减少不必要的变更检测，提升性能。使用场景：纯展示组件、列表项组件等输入明确的组件。注意：OnPush 只比较引用，`items.push(newItem)` 不会触发检测，必须 `items = [...items, newItem]` 创建新引用。

---

### 5. Angular 的响应式表单和模板驱动表单有什么区别？

**答**：响应式表单在组件类中用 `FormBuilder`/`FormControl` 程序化定义表单模型，模板通过 `formControlName` 绑定。模板驱动表单在模板中用 `ngModel` 声明式定义，Angular 自动创建表单模型。区别：(1) **控制权**——响应式在代码中控制，模板驱动在模板中控制；(2) **验证**——响应式用 Validator 函数，模板驱动用 HTML 属性（required、minlength）；(3) **动态表单**——响应式可动态添加/移除控件，模板驱动不适合；(4) **测试**——响应式可直接测试表单模型，模板驱动需要依赖 DOM。推荐：复杂表单用响应式，简单表单两者皆可。

---

### 6. Angular 17+ 的新控制流语法 @if/@for 和 *ngIf/*ngFor 有什么区别？

**答**：三个核心区别：(1) **语法**——`@if (x) { }` 是块语法，`*ngIf="x"` 是指令语法；(2) **性能**——`@for` 内置 `track` 优化（类似 React 的 key），`*ngFor` 需要手动 `trackBy`；(3) **功能**——`@for` 有 `@empty` 块处理空列表，`*ngFor` 没有；`@if` 支持 `@else if`，`*ngIf` 的 else 需要额外 `<ng-template>`。新控制流是编译时优化——Angular 在编译时将 `@if/@for` 转换为更高效的指令，减少运行时开销。新项目推荐全部使用新语法。

---

### 7. RxJS 在 Angular 中的角色是什么？Signal 能完全替代 RxJS 吗？

**答**：RxJS 在 Angular 中处理异步数据流：HTTP 请求、路由参数、表单值变化、WebSocket 等。Angular 的 HttpClient、Router、Reactive Forms 都基于 RxJS Observable。Signal 不能完全替代 RxJS——Signal 适合同步状态管理（简单的值+依赖追踪），RxJS 适合复杂异步流（防抖、重试、并发、取消等操作符）。分工：简单状态用 Signal（计数器、开关、选中项），复杂异步用 RxJS（搜索自动完成、实时数据流、多请求编排）。Angular 的 `toSignal()` 和 `toObservable()` 可以在两者间桥接。

---

### 8. Angular 和 React 在大型企业项目中各有什么优劣？

**答**：Angular 优势：(1) 完整平台——路由、表单、HTTP、i18n 全官方，无需选型；(2) 依赖注入——大型项目服务编排更规范；(3) 强约定——所有人写法一致，新人上手有章可循；(4) TypeScript 原生——模板都有类型检查。Angular 劣势：(1) 学习曲线——DI、RxJS、NgModule 概念多；(2) 包体积——比 React 大 2-3 倍；(3) 生态小——第三方库不如 React 丰富。React 优势：(1) 灵活——自由选型路由/状态/表单方案；(2) 生态大——第三方库极其丰富；(3) 轻量——核心小，按需引入。React 劣势：(1) 自由过度——团队需自建规范，否则代码风格混乱；(2) 选型成本——每个功能都要选库，升级时可能不兼容。选型建议：团队大、项目长周期、需要强约束选 Angular；团队小、项目快速迭代、追求灵活选 React。

---

## 相关链接

- [Angular 官方文档](https://angular.dev/)
- [Angular Signals](https://angular.dev/signals)
- [Angular Standalone Components](https://angular.dev/overview#standalone-components)
- [RxJS 官方文档](https://rxjs.dev/)
- [Angular CLI](https://angular.dev/tools/cli)
