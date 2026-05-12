---
tags:
  - Web前端
  - JavaScript
  - this
  - 绑定
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# this指向与绑定

## What — 是什么

> `this` 是 JavaScript 函数执行时的上下文对象，其值取决于函数的调用方式，而非定义位置（箭头函数除外）。

### 四种绑定规则完整详解

#### 1. 默认绑定

当函数独立调用（没有任何调用对象前缀）时，`this` 适用默认绑定规则：

```javascript
// 非严格模式
function foo() {
    console.log(this); // window
}
foo(); // 独立调用，this → window

// 严格模式
'use strict';
function bar() {
    console.log(this); // undefined
}
bar(); // 严格模式下独立调用，this → undefined
```

默认绑定的关键点：
- 函数名后面没有点号（`.`）调用，即无调用者对象
- 非严格模式下，默认绑定到全局对象（浏览器 `window`，Node.js `global`）
- 严格模式下，默认绑定到 `undefined`
- 即使在非严格模式的函数中调用严格模式的函数，严格模式函数内的 `this` 仍为 `undefined`

```javascript
function nonStrict() {
    console.log(this); // window
}

function strict() {
    'use strict';
    console.log(this); // undefined
}

nonStrict();  // window
strict();     // undefined

// 在非严格模式函数中调用严格模式函数
function outer() {
    console.log(this);        // window
    strict();                 // undefined（严格模式由函数自身决定）
}
```

#### 2. 隐式绑定

当函数通过对象属性调用（`obj.fn()`）时，`this` 指向调用该函数的对象：

```javascript
const obj = {
    name: 'Alice',
    greet() {
        console.log(this.name);
    },
};

obj.greet(); // "Alice" — this 指向 obj
```

隐式绑定的关键规则：
- **只看最后一层调用对象**：`a.b.c.fn()` 中 `this` 指向 `c`，不是 `a` 也不是 `b`
- 对象引用链只有最后一层影响 `this`

```javascript
const a = {
    name: 'a',
    b: {
        name: 'b',
        fn() {
            console.log(this.name);
        },
    },
};

a.b.fn();       // "b" — this 指向 a.b
const fn = a.b.fn;
fn();           // undefined — 隐式丢失，默认绑定
```

#### 3. 显式绑定

通过 `call`、`apply`、`bind` 方法显式指定 `this`：

```javascript
function greet(greeting, punctuation) {
    console.log(`${greeting}, ${this.name}${punctuation}`);
}

const alice = { name: 'Alice' };
const bob = { name: 'Bob' };

greet.call(alice, 'Hello', '!');    // "Hello, Alice!"
greet.apply(bob, ['Hi', '?']);      // "Hi, Bob?"

const boundGreet = greet.bind(alice);
boundGreet('Hey', '~');             // "Hey, Alice~"
```

显式绑定的关键特性：
- `call`：逐个传参，立即执行
- `apply`：传数组参数，立即执行
- `bind`：返回新函数，不立即执行，绑定是永久的
- 传入原始值（字符串、数字、布尔值）时，会被包装为对应对象（`new String()`、`new Number()`、`new Boolean()`）
- 传入 `null` 或 `undefined` 时，在非严格模式下会被替换为全局对象

```javascript
function foo() {
    console.log(this);
}

foo.call(null);      // window（非严格模式，null 被替换为全局对象）
foo.call(undefined); // window（非严格模式）
foo.bind(null)();    // window

// 严格模式下
'use strict';
foo.call(null);      // null（严格模式不替换）
foo.call(undefined); // undefined
```

**硬绑定（hardBind）**：`bind` 返回的函数无法再次被 `call`/`apply` 改变 `this`：

```javascript
function foo() {
    console.log(this.name);
}

const alice = { name: 'Alice' };
const bob = { name: 'Bob' };

const boundFoo = foo.bind(alice);
boundFoo();               // "Alice"
boundFoo.call(bob);       // "Alice" — bind 绑定不可覆盖
boundFoo.apply(bob);      // "Alice"
```

#### 4. new 绑定

使用 `new` 关键字调用构造函数时，`this` 指向新创建的实例对象：

```javascript
function Person(name, age) {
    this.name = name;
    this.age = age;
    // new 调用时：
    // 1. 创建空对象 {}
    // 2. 将空对象的 [[Prototype]] 指向 Person.prototype
    // 3. this 指向该空对象
    // 4. 执行函数体（给 this 赋属性）
    // 5. 自动返回 this（除非函数显式返回对象）
}

const alice = new Person('Alice', 25);
console.log(alice.name); // "Alice"
console.log(alice.age);  // 25
```

`new` 绑定的特殊情况 —— 构造函数返回对象时：

```javascript
function Foo() {
    this.value = 1;
    return { value: 2 }; // 显式返回对象 → new 结果是该对象
}
const foo = new Foo();
console.log(foo.value); // 2（不是 1）

function Bar() {
    this.value = 1;
    return 42; // 返回原始值 → 被忽略，仍然返回 this
}
const bar = new Bar();
console.log(bar.value); // 1
```

### 绑定规则的优先级

优先级从高到低：**new > 显式 > 隐式 > 默认**

```javascript
// 验证：new 优先级高于显式绑定
function Foo(name) {
    this.name = name;
}

const obj = { name: 'obj' };
const boundFoo = Foo.bind(obj);

// bind 绑定了 obj，但 new 仍然创建新实例
const instance = new boundFoo('instance');
console.log(instance.name);    // "instance"（不是 "obj"）
console.log(obj.name);         // "obj"（未被修改）
```

```javascript
// 验证：显式绑定优先级高于隐式绑定
function foo() {
    console.log(this.name);
}

const obj1 = { name: 'obj1', foo };
const obj2 = { name: 'obj2' };

obj1.foo();              // "obj1" — 隐式绑定
obj1.foo.call(obj2);     // "obj2" — 显式绑定覆盖隐式绑定
```

```javascript
// 验证：隐式绑定优先级高于默认绑定
const obj = {
    name: 'obj',
    foo() { console.log(this.name); },
};

obj.foo(); // "obj" — 隐式绑定（优先于默认绑定）
```

优先级判断流程图：

```
函数被 new 调用？
  ├─ 是 → this = 新创建的实例（new 绑定）
  └─ 否 → 函数被 call/apply/bind 调用？
              ├─ 是 → this = 指定的对象（显式绑定）
              └─ 否 → 函数有调用对象（obj.fn()）？
                          ├─ 是 → this = 调用对象（隐式绑定）
                          └─ 否 → this = 全局对象/undefined（默认绑定）
```

### 箭头函数的 this

箭头函数没有自己的 `this`，它在**定义时**继承外层词法作用域的 `this`（称为"词法 this"）：

```javascript
const obj = {
    name: 'Alice',
    // 普通函数：this 在运行时确定
    greetRegular: function () {
        console.log(this.name);
    },
    // 箭头函数：this 在定义时继承外层（此处外层是 obj 的方法上下文）
    greetArrow: () => {
        console.log(this.name); // this → 外层作用域（全局/模块）
    },
};

obj.greetRegular(); // "Alice"
obj.greetArrow();   // undefined（箭头函数的 this 不是 obj）
```

箭头函数 `this` 的关键特性：

1. **定义时确定**：箭头函数的 `this` 在创建时就被"固化"，无法改变
2. **不可修改**：`call`/`apply`/`bind` 无法改变箭头函数的 `this`
3. **不绑定 arguments**：箭头函数也没有自己的 `arguments` 对象
4. **不可用作构造函数**：不能使用 `new` 调用

```javascript
const arrow = () => {
    console.log(this);
};

const obj = { name: 'obj' };
arrow.call(obj);     // window — call 无法改变箭头函数的 this
arrow.apply(obj);    // window
arrow.bind(obj)();   // window
new arrow();         // TypeError: arrow is not a constructor
```

箭头函数 `this` 的查找机制 —— 沿着作用域链向外查找第一个包含 `this` 的普通函数：

```javascript
const obj = {
    name: 'Alice',
    foo() {
        // 普通函数，this 由调用方式决定
        const inner = () => {
            // 箭头函数，继承 foo 的 this
            console.log(this.name);
        };
        inner();
    },
};

obj.foo(); // "Alice" — inner 继承 foo 的 this（即 obj）
```

### 箭头函数 vs 普通函数的 this 对比

| 维度 | 普通函数 | 箭头函数 |
|------|---------|---------|
| `this` 确定时机 | 运行时（根据调用方式） | 定义时（继承外层词法作用域） |
| `call`/`apply`/`bind` | 可以改变 `this` | 无法改变 `this` |
| `new` 调用 | 可以（`this` 指向新实例） | 不可以（没有 `[[Construct]]`） |
| `arguments` | 有自己的 `arguments` 对象 | 没有，可使用剩余参数 `...args` |
| `prototype` | 有 `prototype` 属性 | 没有 `prototype` 属性 |
| 适用场景 | 需要动态 `this`（构造函数、方法） | 回调函数、保持外层 `this` |
| 隐式丢失 | 会丢失 | 不会丢失（`this` 已固化） |

### this 在不同场景中的值

| 场景 | `this` 值 | 示例 |
|------|----------|------|
| 全局作用域 | 全局对象（严格模式 `undefined`） | `console.log(this)` → `window` |
| 独立函数调用 | 全局对象（严格模式 `undefined`） | `fn()` |
| 对象方法调用 | 调用对象 | `obj.fn()` → `obj` |
| 构造函数（new） | 新创建的实例 | `new Fn()` → 实例 |
| 事件处理函数 | 触发事件的 DOM 元素 | `btn.onclick = function() {}` → `btn` |
| 类方法 | 调用对象（可能丢失） | 需要绑定或箭头函数 |
| 回调函数 | 全局对象或 `undefined` | `setTimeout(fn, 0)` |
| 箭头函数 | 定义时外层 `this` | 继承上层词法作用域 |

### 严格模式对 this 的影响

```javascript
// 非严格模式
function sloppy() {
    console.log(this);
}
sloppy(); // window

// 严格模式
function strict() {
    'use strict';
    console.log(this);
}
strict(); // undefined

// 严格模式对方法调用无影响
const obj = {
    name: 'Alice',
    greet() {
        'use strict';
        console.log(this.name);
    },
};
obj.greet(); // "Alice" — 隐式绑定不受严格模式影响
```

| 调用方式 | 非严格模式 | 严格模式 |
|---------|----------|---------|
| 独立调用 `fn()` | `window` | `undefined` |
| `fn.call(null)` | `window`（null 被替换） | `null` |
| `fn.call(undefined)` | `window`（undefined 被替换） | `undefined` |
| `obj.fn()` | `obj` | `obj` |
| `new Fn()` | 新实例 | 新实例 |

## Why — 为什么

### 为什么需要 this

`this` 提供了一种优雅的方式来隐式传递上下文对象，使得函数可以在不同对象上复用：

```javascript
// 没有 this —— 每个对象需要独立的方法
const alice = {
    name: 'Alice',
    greet: function () { return 'Hello, ' + alice.name; }, // 硬编码 alice
};
const bob = {
    name: 'Bob',
    greet: function () { return 'Hello, ' + bob.name; },   // 硬编码 bob
};

// 有 this —— 同一方法可复用
function greet() {
    return 'Hello, ' + this.name;
}
const alice2 = { name: 'Alice', greet };
const bob2 = { name: 'Bob', greet };
alice2.greet(); // "Hello, Alice"
bob2.greet();   // "Hello, Bob"
```

`this` 存在的核心价值：

1. **方法复用**：同一函数可以在不同对象上调用，自动获取正确的上下文
2. **隐式传递上下文**：无需显式传参，代码更简洁
3. **支持面向对象模式**：构造函数、原型继承都依赖 `this`
4. **回调中保持上下文**：配合箭头函数，可以方便地保持外层 `this`

```javascript
// 隐式传递 vs 显式传递
// 显式传递（繁琐）
function greet(context) {
    console.log('Hello, ' + context.name);
}
greet(alice);

// 隐式传递（简洁）
function greet() {
    console.log('Hello, ' + this.name);
}
alice.greet(); // this 自动指向 alice
```

### this 设计的历史原因

JavaScript 的 `this` 设计受到以下因素影响：

1. **借鉴 Scheme 的词法作用域 + Self 的原型继承**：Brendan Eich 在设计 JS 时，融合了 Scheme 的函数式特性和 Self 的基于原型的面向对象特性。`this` 就是为了支持原型继承而引入的。

2. **最初仅为构造函数设计**：早期 JavaScript 中，`this` 主要用在构造函数里，为实例赋属性。默认绑定到全局对象是一个"便利设计"，让独立调用的函数也能工作。

3. **动态绑定是面向对象的必然选择**：如果 `this` 是静态绑定（词法作用域），那么方法就无法在不同对象间复用，面向对象编程将难以实现。

4. **严格模式的修正**：ES5 引入严格模式，将默认绑定的 `this` 改为 `undefined`，是对原始设计的修正，避免全局变量污染。

### 箭头函数为什么没有自己的 this

ES6 引入箭头函数时，故意取消了 `this` 绑定，原因如下：

1. **解决回调中 `this` 丢失的痛点**：在箭头函数出现之前，回调中保持 `this` 需要用 `var self = this` 或 `.bind(this)`，是高频 bug 源。

2. **词法 `this` 更符合直觉**：大多数情况下，开发者希望回调中的 `this` 与外层一致，而非运行时重新绑定。

3. **简化代码**：消除了 `var that = this` 这样的临时变量模式。

```javascript
// 箭头函数出现之前的写法
class Timer {
    constructor() {
        this.seconds = 0;
    }
    start() {
        var self = this; // 旧方案：保存 this 引用
        setInterval(function () {
            self.seconds++;
        }, 1000);
    }
}

// 箭头函数的写法
class Timer {
    constructor() {
        this.seconds = 0;
    }
    start() {
        setInterval(() => {
            this.seconds++; // 自然而然地继承 this
        }, 1000);
    }
}
```

4. **设计取舍**：箭头函数定位为"轻量级回调函数"，不需要 `this`、`arguments`、`prototype` 等构造函数特性，因此取消这些绑定让引擎可以优化。

## How — 怎么用

### 示例1：四种绑定规则完整代码

```javascript
// 1. 默认绑定
function defaultBind() {
    console.log('默认绑定:', this);
}
defaultBind(); // window（非严格模式）

// 2. 隐式绑定
const person = {
    name: 'Alice',
    greet() {
        console.log('隐式绑定:', this.name);
    },
};
person.greet(); // "隐式绑定: Alice"

// 3. 显式绑定
function explicitBind() {
    console.log('显式绑定:', this.name);
}
const bob = { name: 'Bob' };
explicitBind.call(bob); // "显式绑定: Bob"

// 4. new 绑定
function NewBind(name) {
    this.name = name;
    console.log('new 绑定:', this.name);
}
const instance = new NewBind('Charlie'); // "new 绑定: Charlie"
```

### 示例2：隐式丢失问题及修复

**解构赋值导致隐式丢失：**

```javascript
const obj = {
    name: 'Alice',
    greet() {
        console.log(this.name);
    },
};

// 隐式丢失
const { greet } = obj;
greet(); // undefined — 等价于独立调用

// 修复方案1：直接调用
obj.greet(); // "Alice"

// 修复方案2：bind
const boundGreet = obj.greet.bind(obj);
boundGreet(); // "Alice"

// 修复方案3：箭头函数包装
const arrowGreet = () => obj.greet();
arrowGreet(); // "Alice"
```

**回调传参导致隐式丢失：**

```javascript
const obj = {
    name: 'Alice',
    greet() {
        console.log(this.name);
    },
};

// 隐式丢失
function runCallback(cb) {
    cb(); // 独立调用，this 丢失
}
runCallback(obj.greet); // undefined

// 修复方案1：传箭头函数
runCallback(() => obj.greet()); // "Alice"

// 修复方案2：bind
runCallback(obj.greet.bind(obj)); // "Alice"
```

**事件监听导致隐式丢失：**

```javascript
class Button {
    constructor() {
        this.count = 0;
    }

    // ❌ 事件监听中 this 丢失
    setupWrong() {
        document.addEventListener('click', function () {
            this.count++; // this → 触发事件的 DOM 元素
            console.log(this.count);
        });
    }

    // ✅ 箭头函数
    setup() {
        document.addEventListener('click', () => {
            this.count++; // this → Button 实例
            console.log(this.count);
        });
    }

    // ✅ bind
    setupBind() {
        document.addEventListener('click', function () {
            this.count++;
        }.bind(this));
    }
}
```

### 示例3：call/apply/bind 详解与对比

```javascript
function introduce(greeting, punctuation) {
    console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const alice = { name: 'Alice' };
const bob = { name: 'Bob' };

// call — 逐个传参
introduce.call(alice, 'Hello', '!');  // "Hello, I'm Alice!"
introduce.call(bob, 'Hi', '?');      // "Hi, I'm Bob?"

// apply — 数组传参
introduce.apply(alice, ['Hey', '~']); // "Hey, I'm Alice~"

// bind — 返回新函数
const aliceIntro = introduce.bind(alice);
aliceIntro('Hello', '!');             // "Hello, I'm Alice!"

// bind 也可以预设部分参数（偏函数）
const aliceHello = introduce.bind(alice, 'Hello');
aliceHello('!');                      // "Hello, I'm Alice!"
aliceHello('?');                      // "Hello, I'm Alice?"
```

| 方法 | 执行时机 | 参数形式 | 返回值 | 是否可覆盖 |
|------|---------|---------|--------|----------|
| `call` | 立即执行 | 逐个传参 `fn.call(obj, a, b)` | 函数返回值 | - |
| `apply` | 立即执行 | 数组传参 `fn.apply(obj, [a, b])` | 函数返回值 | - |
| `bind` | 返回新函数 | 逐个传参 `fn.bind(obj, a)` | 新函数 | 不可覆盖（`new` 除外） |

### 示例4：bind 的柯里化用法

```javascript
// 基础柯里化：预设部分参数
function multiply(a, b) {
    return a * b;
}

const double = multiply.bind(null, 2);
console.log(double(5));  // 10
console.log(double(10)); // 20

const triple = multiply.bind(null, 3);
console.log(triple(5));  // 15

// 实际应用：日志函数
function log(level, timestamp, message) {
    console.log(`[${level}] ${timestamp}: ${message}`);
}

const info = log.bind(null, 'INFO');
const error = log.bind(null, 'ERROR');

info('2026-05-12', 'Server started');   // [INFO] 2026-05-12: Server started
error('2026-05-12', 'Connection lost'); // [ERROR] 2026-05-12: Connection lost

// 实际应用：事件处理器绑定
class List {
    constructor() {
        this.items = [];
    }

    addItem(item) {
        this.items.push(item);
        console.log('Added:', item);
    }
}

const list = new List();
// 将 addItem 绑定为事件回调
const boundAdd = list.addItem.bind(list);
boundAdd('item1'); // "Added: item1"
```

### 示例5：箭头函数在各种场景中的正确使用

```javascript
// ✅ 场景1：回调中保持 this
class Counter {
    constructor() {
        this.count = 0;
    }
    start() {
        setInterval(() => {
            this.count++;
            console.log(this.count);
        }, 1000);
    }
}

// ✅ 场景2：数组方法回调
class Team {
    constructor(members) {
        this.members = members;
    }
    getNames() {
        return this.members.map(member => `${this.prefix()}${member.name}`);
    }
    prefix() { return 'Member: '; }
}

// ✅ 场景3：Promise 链
class DataLoader {
    constructor() {
        this.data = [];
    }
    load() {
        fetch('/api/data')
            .then(res => res.json())
            .then(json => {
                this.data = json; // 箭头函数保持 this
            });
    }
}

// ❌ 场景4：不适合箭头函数 —— 对象方法
const obj = {
    name: 'Alice',
    greet: () => {
        console.log(this.name); // ❌ this 不是 obj
    },
};
obj.greet(); // undefined

// ✅ 正确做法
const obj2 = {
    name: 'Alice',
    greet() {
        console.log(this.name); // ✅ 隐式绑定
    },
};
obj2.greet(); // "Alice"

// ❌ 场景5：不适合箭头函数 —— 需要动态 this
const button = {
    clicked: false,
    click: () => {
        this.clicked = true; // ❌ this 不是 button
    },
};

// ✅ 正确做法
const button2 = {
    clicked: false,
    click() {
        this.clicked = true; // ✅
    },
};
```

### 示例6：类中 this 的陷阱与修复

```javascript
class ReactComponent {
    constructor() {
        this.state = { count: 0 };
    }

    // ❌ 问题：作为回调调用时 this 丢失
    handleClick() {
        this.setState({ count: this.state.count + 1 });
        // TypeError: this.setState is not a function
    }
}

// ===== 方案1：构造函数中 bind =====
class Component1 {
    constructor() {
        this.state = { count: 0 };
        this.handleClick = this.handleClick.bind(this); // 绑定 this
    }
    handleClick() {
        this.setState({ count: this.state.count + 1 }); // ✅
    }
}

// ===== 方案2：箭头函数类字段 =====
class Component2 {
    state = { count: 0 };

    handleClick = () => {
        this.setState({ count: this.state.count + 1 }); // ✅
    };
}

// ===== 方案3：内联箭头函数（JSX 中） =====
class Component3 {
    constructor() {
        this.state = { count: 0 };
    }
    handleClick() {
        this.setState({ count: this.state.count + 1 });
    }
    render() {
        // 每次渲染创建新函数，性能略差但简单
        return `<button onClick={() => this.handleClick()}>Click</button>`;
    }
}
```

三种方案对比：

| 方案 | 语法 | 性能 | 能否在子类中覆盖 | 推荐度 |
|------|------|------|----------------|--------|
| 构造函数 bind | `this.fn = this.fn.bind(this)` | 好（只绑定一次） | 能 | 高 |
| 箭头函数类字段 | `fn = () => {}` | 好（只创建一次） | 否（不在 prototype 上） | 高 |
| 内联箭头函数 | `onClick={() => this.fn()}` | 差（每次渲染新函数） | 能 | 中 |

### 示例7：手写 call/apply/bind

```javascript
// 手写 call
Function.prototype.myCall = function (context, ...args) {
    // 处理 context 为 null/undefined 的情况
    context = context ?? (function () { return this; })() ? globalThis : undefined;
    context = Object(context); // 原始值包装为对象

    // 用 Symbol 避免属性名冲突
    const fnKey = Symbol('fn');
    context[fnKey] = this; // this 就是调用 myCall 的函数

    const result = context[fnKey](...args); // 执行函数
    delete context[fnKey];                  // 删除临时属性

    return result;
};

// 手写 apply
Function.prototype.myApply = function (context, args) {
    context = context ?? globalThis;
    context = Object(context);

    const fnKey = Symbol('fn');
    context[fnKey] = this;

    const result = args ? context[fnKey](...args) : context[fnKey]();
    delete context[fnKey];

    return result;
};

// 手写 bind
Function.prototype.myBind = function (context, ...bindArgs) {
    const fn = this; // 保存原函数

    const boundFn = function (...callArgs) {
        // new 调用时，this 指向新实例，应使用新实例作为 this
        const isNewCall = this instanceof boundFn;
        const thisArg = isNewCall ? this : context;

        return fn.apply(thisArg, [...bindArgs, ...callArgs]);
    };

    // 维护原型链
    boundFn.prototype = Object.create(fn.prototype);
    return boundFn;
};

// 测试
function greet(greeting, punctuation) {
    console.log(`${greeting}, ${this.name}${punctuation}`);
}

const alice = { name: 'Alice' };
greet.myCall(alice, 'Hello', '!');      // "Hello, Alice!"
greet.myApply(alice, ['Hi', '?']);      // "Hi, Alice?"

const boundGreet = greet.myBind(alice, 'Hey');
boundGreet('~');                        // "Hey, Alice~"
```

### 示例8：React 中 this 的常见问题

```javascript
// 问题1：事件处理函数 this 为 undefined
class Toggle extends React.Component {
    constructor(props) {
        super(props);
        this.state = { isToggleOn: true };
        // 必须在构造函数中绑定 this
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        this.setState(prevState => ({
            isToggleOn: !prevState.isToggleOn,
        }));
    }

    render() {
        return (
            <button onClick={this.handleClick}>
                {this.state.isToggleOn ? 'ON' : 'OFF'}
            </button>
        );
    }
}

// 问题2：列表渲染中传递参数
class ItemList extends React.Component {
    handleClick(id) {
        console.log('Clicked item:', id);
    }

    render() {
        return (
            <ul>
                {this.props.items.map(item => (
                    <li key={item.id} onClick={() => this.handleClick(item.id)}>
                        {item.name}
                    </li>
                ))}
            </ul>
        );
    }
}

// 问题3：函数组件中的 this（函数组件没有 this）
// 函数组件使用 Hooks，不需要关心 this
function FunctionComponent() {
    const [count, setCount] = React.useState(0);
    // 没有 this，直接使用闭包变量
    return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 回调中 this 丢失 | 函数作为参数传递后，隐式绑定丢失 | 用箭头函数或 `bind(this)` |
| 箭头函数不能用 new | 箭头函数没有 `[[Construct]]` | 需要构造函数时用普通函数 |
| React 事件 this 为 undefined | class 方法默认不会绑定 this | 箭头函数类属性或构造函数中 bind |
| 嵌套对象 this 指向 | `a.b.fn()` this 指向 `b` 不是 `a` | 只看最后一层调用对象 |
| 解构赋值 this 丢失 | `const { fn } = obj; fn()` → 默认绑定 | bind 或箭头函数包装 |
| 数组方法中 this 丢失 | `[].forEach(fn)` 中 fn 独立调用 | 传第二参数 `forEach(fn, this)` 或用箭头函数 |
| 类方法作为回调丢失 | 方法传递时丢失隐式绑定 | 构造函数 bind 或箭头函数类字段 |
| IIFE 中的 this | `(function(){ console.log(this) })()` → 默认绑定 | 箭头函数 IIFE 或 bind |
| 原型链方法 this | `instance.fn()` → this 指向 instance | 正常，但注意不要在原型上用箭头函数 |
| call/apply 传 null | 非严格模式被替换为全局对象 | 注意是否需要传 null，严格模式不会替换 |

### 最佳实践

- 回调函数优先用箭头函数（保持外层 `this`）
- 类方法用箭头函数属性（`handleClick = () => {}`）或构造函数 bind
- 需要动态 `this` 的用普通函数（构造函数、原型方法、事件委托）
- 理解四条规则优先级：new > 显式 > 隐式 > 默认
- 避免在对象字面量方法中使用箭头函数
- `call`/`apply` 传 `null` 时注意严格模式差异
- 函数组件 + Hooks 可以完全避免 `this` 问题
- 善用 `bind` 实现偏函数/柯里化

## 面试题

**Q1: `this` 的四种绑定规则是什么？优先级如何？**
> 四种绑定规则：1) 默认绑定：独立调用 `fn()`，非严格模式指向 `window`，严格模式为 `undefined`；2) 隐式绑定：`obj.fn()` 指向 `obj`；3) 显式绑定：`fn.call/apply/bind(obj)` 指向 `obj`；4) `new` 绑定：指向新创建的实例。优先级：new > 显式 > 隐式 > 默认。判断时按此优先级从高到低检查即可。

**Q2: 箭头函数的 `this` 和普通函数有什么不同？**
> 箭头函数没有自己的 `this`，它在定义时继承外层词法作用域的 `this`，且无法被 `call`/`apply`/`bind` 改变。普通函数的 `this` 在运行时根据调用方式确定。因此回调函数中优先用箭头函数保持外层 `this`，需要动态 `this` 时用普通函数。

**Q3: 箭头函数可以作为构造函数吗？为什么？**
> 不可以。箭头函数没有 `[[Construct]]` 内部方法，也没有 `prototype` 属性。当使用 `new` 调用箭头函数时，会抛出 `TypeError: xxx is not a constructor`。这是因为箭头函数的设计初衷是作为轻量级回调函数，不需要构造函数的相关特性（`this` 绑定、`arguments`、`prototype` 等），所以 ES 规范故意取消了这些能力。

**Q4: 以下代码输出什么？`var obj = { fn: () => console.log(this) }; obj.fn();`**
> 输出 `window`（浏览器环境）。虽然 `fn` 通过 `obj.fn()` 调用，看似隐式绑定，但箭头函数没有自己的 `this`，它在定义时继承外层词法作用域的 `this`。此处外层是全局作用域，所以 `this` 指向 `window`。箭头函数的 `this` 不受调用方式影响，始终是定义时继承的值。如果需要 `this` 指向 `obj`，应使用普通函数：`fn() { console.log(this) }`。

**Q5: `call`、`apply`、`bind` 有什么区别？**
> `call` 和 `apply` 立即执行函数，`call` 逐个传参（`fn.call(obj, a, b)`），`apply` 传数组参数（`fn.apply(obj, [a, b])`）。`bind` 返回新函数不立即执行，且绑定是永久的（`new` 调用除外）。`bind` 支持偏函数（预设部分参数），`call`/`apply` 不支持。`call`/`apply` 的绑定是一次性的，`bind` 的绑定是永久的。

**Q6: 如何在类方法中保证 this 指向正确？有哪些方案？**
> 三种主要方案：1) 构造函数中 `this.handleClick = this.handleClick.bind(this)` —— 性能好（只绑定一次），方法在 prototype 上可被继承；2) 箭头函数类字段 `handleClick = () => {}` —— 语法简洁，性能好，但方法在实例上而非 prototype，不可被子类覆盖；3) 内联箭头函数 `onClick={() => this.handleClick()}` —— 简单但每次渲染创建新函数，可能影响性能。推荐使用方案1或方案2。

**Q7: 隐式丢失是什么？举三种常见场景。**
> 隐式丢失是指函数虽然是对象的方法，但通过其他方式调用时 `this` 不再指向原对象。三种常见场景：1) 赋值丢失：`const fn = obj.fn; fn()` —— 函数引用被赋值给变量后独立调用；2) 回调传参丢失：`setTimeout(obj.fn, 0)` —— 函数作为回调传递后独立调用；3) 解构赋值丢失：`const { fn } = obj; fn()` —— 解构后独立调用。本质都是函数失去了调用对象前缀，变成了独立调用，触发默认绑定。修复方式：用 `bind` 绑定、箭头函数包装、或直接 `obj.fn()` 调用。

**Q8: 手写 `bind` 函数需要考虑哪些关键点？**
> 关键点：1) 返回新函数，不立即执行；2) 支持偏函数 —— 预设部分参数，调用时追加剩余参数；3) `new` 调用时，`this` 应指向新实例，而非 bind 绑定的对象（new 优先级高于 bind）；4) 维护原型链 —— 返回的函数的 `prototype` 应正确指向原函数的 `prototype`（通常用 `Object.create(fn.prototype)`）；5) 处理 `context` 为 `null`/`undefined` 的情况；6) 返回函数的 `length` 属性应反映剩余参数个数。

---

**相关链接：**
- [[作用域与闭包]]
- [[原型与继承]]
- [[ES6+核心特性]]
- [[类与面向对象]]
- [[函数高级用法]]
