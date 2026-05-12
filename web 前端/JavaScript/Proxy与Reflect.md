---
tags:
  - Web前端
  - JavaScript
  - Proxy
  - Reflect
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Proxy与Reflect

## What — 是什么

> Proxy 是对象的"拦截器"，能在目标对象的任意操作（读、写、删除、枚举等）上插入自定义行为；Reflect 是与 Proxy handler 一一对应的反射 API，提供操作对象的默认行为。

**核心概念：**

- **Proxy**：创建一个对象的代理，拦截并自定义对象的基本操作（属性查找、赋值、枚举、函数调用等）
- **handler**：一个普通对象，其属性是定义拦截行为的陷阱函数（trap）
- **target**：被代理的原始对象
- **Reflect**：内置的静态对象，提供与 Proxy handler 陷阱函数一一对应的方法，用于执行对象的默认行为

**Proxy 的 13 种拦截操作：**

| 拦截操作 | handler 陷阱 | 触发方式 |
|----------|-------------|----------|
| 读取属性 | `get` | `proxy.key` / `proxy[key]` |
| 设置属性 | `set` | `proxy.key = value` |
| 删除属性 | `deleteProperty` | `delete proxy.key` |
| 判断属性存在 | `has` | `key in proxy` |
| 遍历属性 | `ownKeys` | `Object.keys()` / `for...in` |
| 获取属性描述 | `getOwnPropertyDescriptor` | `Object.getOwnPropertyDescriptor()` |
| 定义属性 | `defineProperty` | `Object.defineProperty()` |
| 获取原型 | `getPrototypeOf` | `Object.getPrototypeOf()` |
| 设置原型 | `setPrototypeOf` | `Object.setPrototypeOf()` |
| 判断是否可扩展 | `isExtensible` | `Object.isExtensible()` |
| 禁止扩展 | `preventExtensions` | `Object.preventExtensions()` |
| 函数调用 | `apply` | `proxy(...args)` |
| 构造函数调用 | `construct` | `new proxy(...args)` |

**Reflect 的静态方法（与 Proxy 一一对应）：**

`Reflect.get` / `Reflect.set` / `Reflect.has` / `Reflect.deleteProperty` / `Reflect.ownKeys` / `Reflect.getOwnPropertyDescriptor` / `Reflect.defineProperty` / `Reflect.getPrototypeOf` / `Reflect.setPrototypeOf` / `Reflect.isExtensible` / `Reflect.preventExtensions` / `Reflect.apply` / `Reflect.construct`

**关键特性：**

- Proxy 拦截的是操作而非方法调用，是"元编程"能力
- Reflect 将 `Object` 上的内部方法统一到函数式 API，返回值更合理（布尔值而非抛异常）
- Proxy + Reflect 配合使用是最佳实践：在 trap 中用 Reflect 执行默认行为
- Proxy 是惰性的，只有被操作时才触发拦截

## Why — 为什么

**适用场景：**

- Vue 3 响应式系统的核心实现
- 对象属性访问的日志记录与审计
- 数据校验与约束（类型检查、范围限制）
- 缓存代理（懒加载、计算属性缓存）
- 私有属性拦截（以 `_` 开头的属性禁止外部访问）
- 观察者模式 / 发布订阅模式
- API 废弃警告（访问旧 API 时提示迁移）
- 不可变数据结构的实现

**对比替代方案：**

| 维度 | Proxy | Object.defineProperty |
|------|-------|----------------------|
| 拦截能力 | 13 种操作（读写、删除、枚举、函数调用等） | 仅 get/set |
| 新增属性 | 自动监听 | 需要手动 `$set` |
| 数组变化 | 直接监听索引和 length | 需要重写 push/splice 等方法 |
| 原型链属性 | 可拦截 | 无法拦截 |
| 删除属性 | 可拦截 deleteProperty | 无法拦截 |
| 函数调用 | 可拦截 apply | 不适用 |
| 构造函数 | 可拦截 construct | 不适用 |
| 性能 | 初始化快，访问时有拦截开销 | 初始化需递归遍历所有属性 |
| 兼容性 | IE 不支持 | IE9+ 支持 |
| 使用方式 | 整体代理，非侵入式 | 逐属性定义，侵入式 |

**优缺点：**

- ✅ 优点：
  - 拦截能力全面，13 种操作全覆盖
  - 非侵入式：不修改原对象，返回新代理对象
  - 可监听新增属性、数组变化、删除操作
  - 可撤销代理（`Proxy.revocable`）
  - 实现响应式系统的理想选择（Vue 3 的选择）
- ❌ 缺点：
  - IE 全版本不支持，无法 polyfill
  - 拦截带来性能开销（热路径慎用）
  - `this` 指向问题（代理后 `this` 可能改变）
  - 部分内置对象（`Map`/`Set`/`Date`等）内部插槽导致代理异常
  - 调试困难，代理对象在控制台展示不如原对象直观

## How — 怎么用

### 快速上手

```javascript
// 基础 Proxy：拦截读写
const user = { name: '张三', age: 25 };

const proxy = new Proxy(user, {
  get(target, key) {
    console.log(`读取属性: ${key}`);
    return Reflect.get(target, key);
  },
  set(target, key, value) {
    console.log(`设置属性: ${key} = ${value}`);
    return Reflect.set(target, key, value); // 返回布尔值
  }
});

proxy.name;          // 读取属性: name → '张三'
proxy.age = 26;      // 设置属性: age = 26
user.age;            // 26（原对象也被修改了）
```

### 代码示例

**1. get/set 拦截 — 数据校验**

```javascript
function validatedObject(initial) {
  const rules = {};

  return new Proxy(initial, {
    get(target, key) {
      return Reflect.get(target, key);
    },
    set(target, key, value) {
      // 年龄校验
      if (key === 'age' && (typeof value !== 'number' || value < 0 || value > 150)) {
        throw new RangeError(`age 必须是 0~150 的数字，收到: ${value}`);
      }
      // 邮箱校验
      if (key === 'email' && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
        throw new TypeError(`email 格式不合法: ${value}`);
      }
      return Reflect.set(target, key, value);
    }
  });
}

const person = validatedObject({ name: '张三', age: 25 });
person.age = 30;        // ✅ 正常
person.age = -1;        // ❌ RangeError
person.email = 'a@b.c'; // ✅ 正常
person.email = 'abc';   // ❌ TypeError
```

**2. get/set 拦截 — 访问日志**

```javascript
function withLogger(obj, name = 'Object') {
  const logs = [];

  const proxy = new Proxy(obj, {
    get(target, key) {
      const value = Reflect.get(target, key);
      logs.push({ time: Date.now(), type: 'GET', key, value });
      return value;
    },
    set(target, key, value) {
      const oldValue = Reflect.get(target, key);
      logs.push({ time: Date.now(), type: 'SET', key, oldValue, newValue: value });
      return Reflect.set(target, key, value);
    }
  });

  // 挂载日志查看方法
  proxy.__logs__ = logs;
  return proxy;
}

const config = withLogger({ theme: 'light', lang: 'zh' }, 'Config');
config.theme;           // GET theme
config.theme = 'dark';  // SET theme: light → dark
console.log(config.__logs__);
// [{ type: 'GET', key: 'theme', value: 'light' }, { type: 'SET', key: 'theme', ... }]
```

**3. get 拦截 — 计算属性缓存**

```javascript
function withCache(obj) {
  const cache = new Map();

  return new Proxy(obj, {
    get(target, key) {
      // 缓存命中
      if (cache.has(key)) {
        console.log(`[缓存命中] ${key}`);
        return cache.get(key);
      }

      const value = Reflect.get(target, key);

      // 如果是函数，缓存其计算结果
      if (typeof value === 'function' && value._cacheable) {
        const result = value.call(target);
        cache.set(key, result);
        return result;
      }

      return value;
    }
  });
}

const data = withCache({
  get expensiveCalc() {
    console.log('[计算中...]');
    let sum = 0;
    for (let i = 0; i < 1e7; i++) sum += i;
    return sum;
  }
});

data.expensiveCalc; // [计算中...]
data.expensiveCalc; // [缓存命中] （不再重复计算）
```

**4. has 拦截 — in 操作符控制**

```javascript
// 隐藏私有属性，使其不被 in 检测到
function withPrivate(obj) {
  return new Proxy(obj, {
    has(target, key) {
      if (key.startsWith('_')) {
        return false; // 私有属性对 in 不可见
      }
      return Reflect.has(target, key);
    },
    get(target, key) {
      if (key.startsWith('_')) {
        throw new Error(`无法访问私有属性: ${key}`);
      }
      return Reflect.get(target, key);
    }
  });
}

const user = withPrivate({ name: '张三', _password: '123456', _id: 1 });

'name' in user;       // true
'_password' in user;  // false（隐藏了）
user.name;            // '张三'
user._password;       // Error: 无法访问私有属性: _password
```

**5. deleteProperty 拦截**

```javascript
function withDeleteProtection(obj) {
  const protectedKeys = new Set();

  return new Proxy(obj, {
    deleteProperty(target, key) {
      if (protectedKeys.has(key)) {
        throw new Error(`属性 "${key}" 受保护，不可删除`);
      }
      const result = Reflect.deleteProperty(target, key);
      console.log(`删除属性: ${key}，结果: ${result}`);
      return result;
    },
    // 标记保护属性的方法
    get(target, key) {
      if (key === 'protect') {
        return (k) => { protectedKeys.add(k); return proxy; };
      }
      return Reflect.get(target, key);
    }
  });

  // 注意：proxy 在声明后赋值
  const proxy = new Proxy(obj, {
    deleteProperty(target, key) {
      if (protectedKeys.has(key)) {
        throw new Error(`属性 "${key}" 受保护，不可删除`);
      }
      return Reflect.deleteProperty(target, key);
    }
  });
  return proxy;
}

// 简化实现
function protectDeletion(obj, immutableKeys = []) {
  const protectedSet = new Set(immutableKeys);
  return new Proxy(obj, {
    deleteProperty(target, key) {
      if (protectedSet.has(key)) {
        console.warn(`属性 "${key}" 不可删除`);
        return false;
      }
      return Reflect.deleteProperty(target, key);
    }
  });
}

const config = protectDeletion(
  { host: 'localhost', port: 3000, timeout: 5000 },
  ['host', 'port']
);

delete config.timeout; // true
delete config.host;    // false + 警告
```

**6. apply / construct 拦截 — 函数和构造函数**

```javascript
// apply：拦截函数调用
function withCallCount(fn) {
  let count = 0;
  const proxy = new Proxy(fn, {
    apply(target, thisArg, args) {
      count++;
      console.log(`函数被调用第 ${count} 次，参数:`, args);
      return Reflect.apply(target, thisArg, args);
    }
  });
  proxy.callCount = () => count;
  return proxy;
}

const add = withCallCount(function(a, b) { return a + b; });
add(1, 2);           // 函数被调用第 1 次，参数: [1, 2]
add(3, 4);           // 函数被调用第 2 次，参数: [3, 4]
add.callCount();     // 2

// construct：拦截 new 操作
const ValidatedPerson = new Proxy(function(name, age) {
  this.name = name;
  this.age = age;
}, {
  construct(target, args) {
    const [name, age] = args;
    if (typeof name !== 'string') throw new TypeError('name 必须是字符串');
    if (age < 0 || age > 150) throw new RangeError('age 不合法');
    return Reflect.construct(target, args);
  }
});

const p = new ValidatedPerson('张三', 25);  // ✅
new ValidatedPerson(123, 25);               // ❌ TypeError
new ValidatedPerson('李四', -1);            // ❌ RangeError
```

**7. Proxy.revocable — 可撤销代理**

```javascript
const sensitiveData = { apiKey: 'sk-xxxx', secret: 'top-secret' };

const { proxy, revoke } = Proxy.revocable(sensitiveData, {
  get(target, key) {
    console.log(`访问敏感字段: ${key}`);
    return Reflect.get(target, key);
  }
});

proxy.apiKey;  // 访问敏感字段: apiKey → 'sk-xxxx'
proxy.secret;  // 访问敏感字段: secret → 'top-secret'

revoke();      // 撤销代理

proxy.apiKey;  // ❌ TypeError: Cannot perform 'get' on a proxy that has been revoked

// 实际应用：限时访问令牌
function createTimedAccess(obj, ms) {
  const { proxy, revoke } = Proxy.revocable(obj, {
    get(target, key) {
      return Reflect.get(target, key);
    }
  });
  setTimeout(revoke, ms);
  return proxy;
}

const tempAccess = createTimedAccess({ data: '重要数据' }, 5000);
tempAccess.data;  // '重要数据'（5秒内可访问）
// 5秒后 → TypeError: proxy has been revoked
```

**8. Reflect 完整用法**

```javascript
const obj = { x: 1, y: 2 };

// Reflect.get(target, key, receiver?)
Reflect.get(obj, 'x');                              // 1
Reflect.get({ x: 1 }, 'x', { x: 100 });            // 100（receiver 影响 this）

// Reflect.set(target, key, value, receiver?)
Reflect.set(obj, 'z', 3);                           // true
Reflect.set(obj, 'x', 10);                          // true

// Reflect.has(target, key)
Reflect.has(obj, 'x');                               // true（等价于 'x' in obj）

// Reflect.deleteProperty(target, key)
Reflect.deleteProperty(obj, 'y');                    // true（等价于 delete obj.y）

// Reflect.ownKeys(target)
Reflect.ownKeys({ a: 1, [Symbol('b')]: 2 });        // ['a', Symbol(b)]

// Reflect.getOwnPropertyDescriptor(target, key)
Reflect.getOwnPropertyDescriptor(obj, 'x');          // { value: 10, writable: true, ... }

// Reflect.defineProperty(target, key, descriptor)
Reflect.defineProperty(obj, 'readOnly', { value: 99, writable: false }); // true

// Reflect.getPrototypeOf / setPrototypeOf
Reflect.getPrototypeOf(obj);                         // Object.prototype
Reflect.setPrototypeOf(obj, null);                   // true

// Reflect.isExtensible / preventExtensions
Reflect.isExtensible(obj);                           // true
Reflect.preventExtensions(obj);                      // true

// Reflect.apply(target, thisArg, args)
Reflect.apply(Math.max, null, [1, 5, 3]);            // 5

// Reflect.construct(target, args, newTarget?)
function Point(x, y) { this.x = x; this.y = y; }
Reflect.construct(Point, [10, 20]);                  // Point { x: 10, y: 20 }

// Reflect 对比 Object 的优势
try {
  Object.defineProperty({}, 'a', { get() { return 1; } });
  Object.defineProperty({}, 'a', { value: 2 }); // ❌ TypeError（严格模式抛异常）
} catch (e) {}

const ok = Reflect.defineProperty({}, 'a', { value: 2 }); // 返回 false，不抛异常
// 布尔返回值更适合在 Proxy trap 中使用
```

**9. Proxy + Reflect 实现响应式数据（简化版 Vue3 reactive）**

```javascript
// 简化版 Vue3 响应式系统
const targetMap = new WeakMap();  // target → depsMap
let activeEffect = null;

function effect(fn) {
  activeEffect = fn;
  fn();                          // 立即执行一次，触发 get 收集依赖
  activeEffect = null;
}

function track(target, key) {
  if (!activeEffect) return;
  let depsMap = targetMap.get(target);
  if (!depsMap) {
    depsMap = new Map();
    targetMap.set(target, depsMap);
  }
  let dep = depsMap.get(key);
  if (!dep) {
    dep = new Set();
    depsMap.set(key, dep);
  }
  dep.add(activeEffect);
}

function trigger(target, key) {
  const depsMap = targetMap.get(target);
  if (!depsMap) return;
  const dep = depsMap.get(key);
  if (dep) {
    dep.forEach(effect => effect());
  }
}

function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      const result = Reflect.get(target, key, receiver);
      track(target, key);                    // 收集依赖
      // 深层响应式：如果值是对象，递归代理
      if (result !== null && typeof result === 'object') {
        return reactive(result);
      }
      return result;
    },
    set(target, key, value, receiver) {
      const oldValue = Reflect.get(target, key, receiver);
      const result = Reflect.set(target, key, value, receiver);
      if (oldValue !== value) {
        trigger(target, key);                // 触发更新
      }
      return result;
    },
    deleteProperty(target, key) {
      const hadKey = Reflect.has(target, key);
      const result = Reflect.deleteProperty(target, key);
      if (hadKey && result) {
        trigger(target, key);
      }
      return result;
    }
  });
}

// 使用示例
const state = reactive({ count: 0, user: { name: '张三' } });

effect(() => {
  console.log(`count 变为: ${state.count}`);
  // count 变为: 0（首次执行）
});

state.count = 1;    // count 变为: 1（自动触发）
state.count = 2;    // count 变为: 2
state.user.name;    // 深层代理生效
delete state.count; // 触发更新
```

**10. Proxy 嵌套 / 多层代理**

```javascript
// 多层代理：在已有代理上再加代理
const raw = { value: 42 };

// 第一层：日志
const logged = new Proxy(raw, {
  get(target, key) {
    console.log(`[LOG] 读取 ${String(key)}`);
    return Reflect.get(target, key);
  }
});

// 第二层：校验
const validated = new Proxy(logged, {
  set(target, key, value) {
    if (key === 'value' && value < 0) {
      throw new RangeError('value 不能为负数');
    }
    return Reflect.set(target, key, value);
  }
});

validated.value;       // [LOG] 读取 value → 42
validated.value = 100; // [LOG] 读取 value （set 内部触发了 get）+ 校验通过
validated.value = -1;  // ❌ RangeError

// 代理链模式：像中间件一样叠加功能
function createProxyChain(target, ...middlewares) {
  return middlewares.reduce((current, mw) => mw(current), target);
}

const withLog = (obj) => new Proxy(obj, {
  get(t, k) { console.log(`[GET] ${String(k)}`); return Reflect.get(t, k); }
});
const withFreeze = (obj) => new Proxy(obj, {
  set(t, k) { console.warn(`[FROZEN] 不可修改`); return true; }
});

const chained = createProxyChain({ x: 1 }, withLog, withFreeze);
chained.x;       // [GET] x → 1
chained.x = 2;   // [FROZEN] 不可修改
```

**11. 观察者模式实现**

```javascript
// 用 Proxy 实现观察者模式
function observable(target) {
  const observers = new Set();

  const proxy = new Proxy(target, {
    set(target, key, value) {
      const oldValue = Reflect.get(target, key);
      const result = Reflect.set(target, key, value);
      if (oldValue !== value) {
        notify({ key, oldValue, newValue: value });
      }
      return result;
    },
    deleteProperty(target, key) {
      const hadKey = Reflect.has(target, key);
      const oldValue = Reflect.get(target, key);
      const result = Reflect.deleteProperty(target, key);
      if (hadKey) {
        notify({ key, oldValue, newValue: undefined, type: 'delete' });
      }
      return result;
    }
  });

  function notify(change) {
    observers.forEach(fn => fn(change));
  }

  proxy.subscribe = function(fn) {
    observers.add(fn);
    return () => observers.delete(fn); // 返回取消订阅函数
  };

  return proxy;
}

// 使用
const store = observable({ count: 0, name: 'App' });

const unsubscribe = store.subscribe(({ key, oldValue, newValue }) => {
  console.log(`属性变更: ${key}, ${oldValue} → ${newValue}`);
});

store.count = 1;      // 属性变更: count, 0 → 1
store.name = 'Demo';  // 属性变更: name, App → Demo
delete store.name;     // 属性变更: name, App → undefined

unsubscribe();         // 取消订阅
store.count = 2;       // 无输出
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Proxy 后 `this` 指向错误 | Proxy 的 `get` trap 返回的方法中 `this` 指向 proxy 而非 target | 使用 `Reflect.get(target, key, receiver)`，receiver 保持 proxy |
| 对象内部方法调用自身属性未触发拦截 | 方法内 `this` 指向 target 而非 proxy | 构造时将 proxy 传入，或用 receiver 参数 |
| 代理 `Map`/`Set` 报错 | 内部插槽（internal slot）检查 `this` 是否为原始对象 | 使用 `bind` 绑定 target，或重写方法将 `this` 指回 target |
| 代理 `Date` 对象报错 | `Date` 方法内部要求 `this` 是真正的 Date 实例 | 将 Date 方法绑定到 target，或自定义 get trap 转发 |
| 双层代理性能陷阱 | 每次访问穿过多层 proxy，开销叠加 | 避免不必要的代理嵌套，合并拦截逻辑 |
| `JSON.stringify` 代理对象可能触发无限循环 | 如果 proxy 的 get trap 返回代理对象自身 | 在 get 中对 `toJSON` 方法特殊处理 |
| 代理对象 `===` 比较为 false | `proxy !== target`，它们是不同的对象 | 用 `WeakMap` 建立 proxy → target 的映射 |
| 性能敏感场景卡顿 | 拦截函数在热路径上频繁调用 | 只在需要拦截的属性上使用，热路径直接访问 target |

**Map/Set 代理的正确做法：**

```javascript
// ❌ 直接代理 Map 会报错
const map = new Map();
const proxy = new Proxy(map, {});
proxy.set('key', 'value'); // TypeError: Method Map.prototype.set called on incompatible receiver

// ✅ 正确做法：重写方法，绑定 target
function proxyMap(map) {
  return new Proxy(map, {
    get(target, key) {
      const value = Reflect.get(target, key);
      // 如果是 Map 自身方法，绑定到原始 target
      if (typeof value === 'function' && value === Map.prototype[key]) {
        return value.bind(target);
      }
      return value;
    }
  });
}

const pMap = proxyMap(new Map());
pMap.set('a', 1);     // ✅ 正常
pMap.get('a');         // 1
pMap.size;             // 1
```

**this 指向问题的详细示例：**

```javascript
// ❌ this 指向问题
const obj = {
  name: '张三',
  greet() {
    return `Hello, ${this.name}`;
  }
};

const proxy = new Proxy(obj, {
  get(target, key) {
    console.log(`读取: ${String(key)}`);
    return Reflect.get(target, key); // this 指向 target
  }
});

proxy.greet(); // 读取: greet → "Hello, 张三"（看起来正常，因为 greet 内部 this 没有再走 proxy）

// 但如果方法内部访问的属性也被拦截，且 this 不是 proxy，就不会触发拦截
// ✅ 使用 receiver 保证 this 链
const proxy2 = new Proxy(obj, {
  get(target, key, receiver) {
    console.log(`读取: ${String(key)}`);
    return Reflect.get(target, key, receiver); // receiver = proxy2
  }
});

proxy2.greet(); // 读取: greet, 读取: name → "Hello, 张三"
```

### 最佳实践

- **Proxy + Reflect 配对使用**：在 trap 中始终用 `Reflect` 对应方法执行默认行为，保证 receiver 正确传递
- **保持 trap 的不变式**：`get` 返回值应与 `target` 的属性描述一致（configurable/writable）；`set` 返回布尔值；`defineProperty` 返回布尔值
- **避免在热路径使用 Proxy**：高频调用的核心逻辑（如渲染循环）不要加代理
- **深层响应式按需代理**：只在属性被访问时才递归创建子代理（惰性代理），避免初始化时遍历整个对象树
- **使用 WeakMap 管理代理缓存**：同一对象返回同一代理，避免重复代理
- **可撤销代理用于临时授权**：权限控制、限时访问等场景用 `Proxy.revocable`
- **Map/Set/Date 等内置对象需特殊处理**：在 get trap 中将方法 `bind` 回 target
- **代理对象要实现 `toJSON`**：否则 `JSON.stringify` 可能触发意外拦截或丢失数据

## 面试题

**Q1: Proxy 的基本用法是什么？handler 中有哪些常见的陷阱函数？**

> Proxy 通过 `new Proxy(target, handler)` 创建代理对象。handler 是包含陷阱函数的对象，常见陷阱：`get`（拦截属性读取）、`set`（拦截属性设置）、`has`（拦截 `in` 操作符）、`deleteProperty`（拦截 `delete`）、`apply`（拦截函数调用）、`construct`（拦截 `new`）、`ownKeys`（拦截键遍历）。共 13 种陷阱函数，覆盖对象的所有基本操作。

**Q2: Proxy 和 Object.defineProperty 的区别是什么？Vue3 为什么用 Proxy 替换 Object.defineProperty？**

> 核心区别：(1) Proxy 拦截 13 种操作，defineProperty 只能拦截 get/set；(2) Proxy 能监听新增属性（无需 `$set`），defineProperty 需要递归遍历；(3) Proxy 能直接监听数组索引和 length 变化，defineProperty 需要重写数组方法；(4) Proxy 是非侵入式的整体代理，defineProperty 是逐属性侵入式定义。Vue3 选择 Proxy 是因为可以一行代码代理整个对象，自动覆盖新增属性和数组变化，消除 Vue2 中 `Vue.set`、数组变异方法等补丁方案，代码更简洁、功能更完整。

**Q3: Reflect 存在的意义是什么？为什么 Proxy 的 trap 中推荐用 Reflect？**

> Reflect 存在的意义：(1) 将 Object 上的内部方法统一为函数式 API（如 `Reflect.deleteProperty(obj, key)` 代替 `delete obj[key]`）；(2) 返回值更合理——`Reflect.defineProperty` 返回布尔值而非抛异常，适合在 Proxy trap 中使用；(3) 与 Proxy 的 13 个 trap 一一对应，`Reflect.get/set/has/...` 正好是各 trap 的默认行为实现；(4) 提供 `receiver` 参数，保证 getter/setter 中 `this` 正确指向代理对象而非原始对象。在 trap 中用 `Reflect.get(target, key, receiver)` 是最佳实践。

**Q4: 如何用 Proxy 实现对象的私有属性？**

> 在 `get` trap 中拦截以 `_` 开头的属性名，抛出错误或返回 `undefined`；在 `set` trap 中同样拦截，阻止赋值；在 `has` trap 中返回 `false`，使 `in` 操作符检测不到；在 `ownKeys` trap 中过滤掉私有属性，使其不出现在 `Object.keys()` 中。这样就实现了全面的私有属性保护。但注意这不是真正的私有，直接访问 `target._prop` 仍然可以，Proxy 只拦截代理对象的操作。

**Q5: Proxy.revocable 是什么？有什么应用场景？**

> `Proxy.revocable(target, handler)` 创建可撤销的代理，返回 `{ proxy, revoke }`。调用 `revoke()` 后，proxy 对象的任何操作都会抛出 TypeError。应用场景：(1) 限时访问——设定定时器到期后 revoke；(2) 权限控制——用户权限变更后立即撤销数据访问代理；(3) 模块 API 封装——暴露代理对象，需要时 revoke 切断外部访问；(4) 安全隔离——将敏感数据通过代理有限暴露，用完即焚。

**Q6: Vue3 的 reactive 为什么选择 Proxy？实现响应式时有哪些技术细节？**

> Vue3 选择 Proxy 的原因：能监听新增/删除属性、数组变化、无需递归遍历初始化。技术细节：(1) 使用 `WeakMap` 缓存已代理对象，避免重复代理；(2) 深层响应式采用惰性代理——只在 `get` trap 中发现属性值是对象时才递归创建子代理，而非初始化时递归整个对象树；(3) `get` 中收集依赖（track），`set` 中触发更新（trigger），`deleteProperty` 同样触发更新；(4) 使用 `Reflect.get(target, key, receiver)` 保证 this 指向正确；(5) 对 Map/Set/Date 等内置对象需要特殊处理（在 get 中 bind 回 target）；(6) 用 `raw === target` 判断避免代理对象的递归代理。

**Q7: Proxy 能否代理 Map/Set？为什么直接代理会报错？**

> 直接代理 Map/Set 会报错，因为 Map/Set 的方法（如 `get`、`set`、`has`）内部通过内部插槽（internal slot）检查 `this` 是否为真正的 Map/Set 实例，而代理对象的 `this` 不是原始实例，导致 `TypeError: Method called on incompatible receiver`。解决方案：在 `get` trap 中检测到 Map/Set 原型方法时，用 `value.bind(target)` 将方法绑定回原始对象，使内部 `this` 指向真正的实例。Vue3 的 reactive 对 Map/Set 做了类似的特殊处理。

**Q8: Proxy 的性能考量是什么？在生产环境中应该注意什么？**

> 性能考量：(1) 每次 Proxy 操作都经过 JS 引擎的拦截调用，比直接属性访问慢（约 2~10 倍）；(2) 多层代理嵌套使开销线性叠加；(3) `get` trap 在频繁读取时开销最大。生产环境建议：(1) 热路径（渲染循环、高频计算）避免使用 Proxy，直接访问原始对象；(2) 惰性代理——深层对象只在访问时才代理，而非初始化时遍历；(3) 合并拦截逻辑，减少代理层数；(4) 用 `WeakMap` 缓存代理，避免重复创建；(5) Vue3 的响应式只在开发环境收集依赖信息，生产环境精简 trap 逻辑；(6) 不需要响应式的大数据（如列表渲染的只读数据）不要用 reactive 包装。

---

**相关链接：**
- [[ES6+核心特性]]
- [[Vue3响应式原理]]
- [[设计模式]]
