---
tags:
  - Web前端
  - 工程化
  - 测试
  - Jest
  - Vitest
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# 前端测试Jest与Vitest

## What — 是什么

> Jest 是 Meta 出品的 JavaScript 测试框架，Vitest 是 Vite 生态的测试框架，两者是前端单测和组件测试的主流选择。

**核心概念：**

- **测试类型**：单元测试（函数/工具）、组件测试（UI 交互）、集成测试（多模块协作）、E2E 测试（用户流程）
- **Jest 核心**：`describe`/`it`/`expect`、Mock 函数、快照测试、覆盖率
- **Vitest 核心**：兼容 Jest API、Vite 原生转换管线、极速 HMR 模式
- **测试工具库**：Testing Library（DOM 查询）、Happy-DOM/jsdom（浏览器模拟）

**关键特性：**

- Vitest 与 Vite 共享配置（alias、插件、transform），零额外配置
- Jest 生态成熟，Vitest 速度更快
- Testing Library 核心原则：测试用户行为，不测试实现细节

## Why — 为什么

**适用场景：**

- 工具函数/业务逻辑的单元测试
- 组件交互行为的测试
- API 请求的 Mock 测试
- CI/CD 质量门禁

**对比替代方案：**

| 维度 | Vitest | Jest | Mocha | Playwright |
|------|--------|------|-------|------------|
| 速度 | 极快（Vite 管线） | 快 | 中 | 慢（E2E） |
| Vite 兼容 | 原生 | 需额外配置 | 无关 | 无关 |
| 生态 | 快速追赶 | 极成熟 | 丰富 | E2E 生态 |
| HMR 模式 | 支持 | 不支持 | 不支持 | 不适用 |
| E2E | 不支持 | 不支持 | 不支持 | 原生支持 |

**优缺点：**

- ✅ Vitest 优点：
  - 与 Vite 共享配置，零额外工作
  - 速度极快，HMR 模式即时反馈
  - API 兼容 Jest，迁移成本低
- ❌ Vitest 缺点：
  - 部分 Jest 生态插件不兼容
  - 快照测试格式与 Jest 不完全一致

## How — 怎么用

### 快速上手

**Vitest 配置：**

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
    plugins: [react()],
    resolve: {
        alias: { '@': resolve(__dirname, 'src') },
    },
    test: {
        environment: 'happy-dom', // 或 jsdom
        globals: true,
        setupFiles: ['./src/test/setup.ts'],
        coverage: {
            provider: 'v8',
            reporter: ['text', 'html'],
            include: ['src/**/*.{ts,tsx}'],
            exclude: ['src/**/*.d.ts', 'src/types/**'],
        },
    },
});
```

**测试文件：**

```typescript
// src/utils/format.test.ts
import { describe, it, expect } from 'vitest';
import { formatCurrency, formatDate } from './format';

describe('formatCurrency', () => {
    it('格式化金额', () => {
        expect(formatCurrency(1234.5)).toBe('¥1,234.50');
    });

    it('处理零', () => {
        expect(formatCurrency(0)).toBe('¥0.00');
    });

    it('处理负数', () => {
        expect(formatCurrency(-100)).toBe('-¥100.00');
    });
});

describe('formatDate', () => {
    it('格式化日期', () => {
        const date = new Date('2026-01-15');
        expect(formatDate(date)).toBe('2026-01-15');
    });
});
```

### 代码示例

**Mock 函数：**

```typescript
import { describe, it, expect, vi } from 'vitest';

// Mock 模块
vi.mock('@/api/user', () => ({
    fetchUser: vi.fn(),
    updateUser: vi.fn(),
}));

import { fetchUser, updateUser } from '@/api/user';

it('Mock API 调用', async () => {
    fetchUser.mockResolvedValue({ id: 1, name: 'Alice' });

    const user = await fetchUser(1);
    expect(user.name).toBe('Alice');
    expect(fetchUser).toHaveBeenCalledWith(1);
});

it('验证调用次数', () => {
    updateUser('Alice');
    updateUser('Bob');
    expect(updateUser).toHaveBeenCalledTimes(2);
    expect(updateUser).toHaveBeenLastCalledWith('Bob');
});
```

**React 组件测试：**

```tsx
// src/components/SearchInput.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import SearchInput from './SearchInput';

describe('SearchInput', () => {
    it('输入触发搜索', async () => {
        const onSearch = vi.fn();
        render(<SearchInput onSearch={onSearch} />);

        const input = screen.getByPlaceholderText('搜索...');
        await userEvent.type(input, 'hello');
        fireEvent.keyDown(input, { key: 'Enter' });

        expect(onSearch).toHaveBeenCalledWith('hello');
    });

    it('显示加载状态', () => {
        render(<SearchInput onSearch={vi.fn()} loading />);
        expect(screen.getByTestId('spinner')).toBeInTheDocument();
    });
});
```

**Vue 组件测试：**

```typescript
// src/components/Counter.test.ts
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import Counter from './Counter.vue';

describe('Counter', () => {
    it('点击按钮递增', async () => {
        const wrapper = mount(Counter);
        expect(wrapper.text()).toContain('0');

        await wrapper.find('button.increment').trigger('click');
        expect(wrapper.text()).toContain('1');
    });

    it('emit 事件', async () => {
        const wrapper = mount(Counter);
        await wrapper.find('button.reset').trigger('click');
        expect(wrapper.emitted('reset')).toBeTruthy();
    });
});
```

**自定义 Hook 测试：**

```typescript
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

it('useCounter 递增递减', () => {
    const { result } = renderHook(() => useCounter(0));

    expect(result.current.count).toBe(0);

    act(() => result.current.increment());
    expect(result.current.count).toBe(1);

    act(() => result.current.decrement());
    expect(result.current.count).toBe(0);
});
```

**快照测试：**

```typescript
it('组件渲染快照', () => {
    const { container } = render(<UserCard name="Alice" role="admin" />);
    expect(container).toMatchSnapshot();
});

// 首次运行生成快照文件，后续运行对比
// 快照不一致时：确认是否期望变更，是则 vitest -u 更新
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Mock 不生效 | `vi.mock` 在 import 之后 | 将 `vi.mock` 放在文件顶部（hoisted） |
| 定时器测试慢 | 真实等待 setTimeout | `vi.useFakeTimers()` + `vi.advanceTimersByTime()` |
| 组件测试样式缺失 | jsdom 不支持 CSS | 用 `@testing-library/jest-dom` 的 `toBeInTheDocument` |
| 快照频繁失败 | 组件变化频繁 | 快照只用于稳定 UI，动态内容避免快照 |
| 测试间互相影响 | 全局状态未清理 | `afterEach(() => vi.restoreAllMocks())` |

### 最佳实践

- Vite 项目优先选 Vitest，非 Vite 项目用 Jest
- 测试用户行为，不测实现细节（Testing Library 原则）
- Mock 最小化：只 Mock 外部依赖（API/时间），不 Mock 内部函数
- `afterEach` 清理 Mock 和状态，避免测试间干扰
- CI 中跑覆盖率门禁：核心逻辑 > 80%
- 快照测试用于稳定 UI，频繁变更的组件避免快照

## 面试题

**Q1: 单元测试和集成测试的区别是什么？**
> 单元测试针对最小功能单元（函数/类/工具方法），隔离外部依赖，测试逻辑正确性；集成测试验证多个模块协作时的行为，涉及真实或部分真实的依赖组合。单元测试快而细，集成测试慢但更接近真实场景。

**Q2: Mock 的作用是什么？什么时候该用/不该用？**
> Mock 用于替代外部依赖（API 请求、定时器、第三方模块），使测试聚焦于被测单元自身逻辑。应 Mock 外部依赖（网络请求、时间、随机值），不应 Mock 被测函数的内部实现细节，否则测试与实现耦合，重构时测试会大量失效。

**Q3: Vitest 为什么比 Jest 快？**
> Vitest 复用 Vite 的转换管线（esbuild 预编译 + 按需转换），而 Jest 每个测试文件都需经过完整的 Babel/TS 编译；Vitest 支持 HMR 模式，修改代码只重新运行相关测试；Vitest 的模块池可复用已编译模块，减少重复转换开销。

**Q4: Testing Library 的核心原则是什么？**
> 核心原则是"测试用户行为，不测试实现细节"——通过用户可感知的方式查询元素（`getByRole`、`getByText`），模拟用户交互（`userEvent`），断言可见结果。避免依赖组件内部状态、类名、CSS 选择器等实现细节，使测试更具韧性。

---

**相关链接：**
- [[Webpack与Vite]]
- [[代码规范ESLint与Prettier]]
- [[React Hooks详解]]
