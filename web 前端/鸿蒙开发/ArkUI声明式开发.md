---
tags:
  - Web前端
  - 鸿蒙
  - ArkUI
  - 声明式UI
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# ArkUI声明式开发

## What — 是什么

> ArkUI 是鸿蒙的声明式 UI 开发框架，类似 SwiftUI/Jetpack Compose/Flutter，通过 ArkTS 声明式语法描述 UI 结构，框架自动处理数据变化到 UI 更新的映射。ArkUI 采用"UI = f(State)"的声明式范式，开发者只需关注"UI 应该是什么样"，而非"如何把 UI 变成目标样子"。

**核心概念：**

- **@Component**：自定义组件装饰器，所有 UI 组件必须标注
- **@Entry**：页面入口装饰器，标记为路由页面
- **@Builder**：轻量级 UI 复用函数，类似 React 的 render 函数
- **声明式范式**：描述 UI 应该是什么样，而非命令式操作 DOM

**核心架构：**

```
┌──────────────────────────────────────────────┐
│                 ArkUI 框架                     │
├──────────────────────────────────────────────┤
│  声明式语法层 (ArkTS)                          │
│  @Component / @Builder / @Extend / @Styles   │
├──────────────────────────────────────────────┤
│  组件层                                       │
│  基础组件: Text/Button/Image/TextInput        │
│  容器组件: Column/Row/Stack/Flex/List/Grid    │
│  画布组件: Canvas/CustomPaint                  │
├──────────────────────────────────────────────┤
│  渲染引擎层                                    │
│  组件树 → 渲染树 → 布局计算 → 绘制合成         │
│  状态变化 → 差量更新 → 局部刷新                │
└──────────────────────────────────────────────┘
```

**与 React/Vue/Flutter 声明式 UI 对比：**

| 维度 | ArkUI | React | Vue 3 | Flutter |
|------|-------|-------|-------|---------|
| 语言 | ArkTS | JS/TS | JS/TS | Dart |
| 组件定义 | @Component struct | Function/Class | SFC setup | StatelessWidget |
| 状态 | @State/@Prop/@Link | useState/Context | ref/reactive | State/Provider |
| 条件渲染 | if/else | && / 三元 | v-if | if/else |
| 列表渲染 | ForEach/LazyForEach | map() | v-for | ListView.builder |
| 样式 | 链式属性 | CSS/CSS-in-JS | CSS | Widget 属性 |
| 事件 | .onClick() | onClick | @click | onPressed |

## Why — 为什么

**适用场景：**

- 鸿蒙应用 UI 开发（唯一官方 UI 框架）
- 多设备适配（手机/平板/手表/车机）
- 高性能列表和复杂交互页面

**优缺点：**

- ✅ 优点：
  - 声明式范式，代码简洁直观
  - 声明式状态管理，自动驱动 UI 更新
  - 链式调用，样式设置流畅
  - 内置组件丰富，开箱即用
  - 方舟引擎 AOT 编译，渲染性能优秀
- ❌ 缺点：
  - 生态不如 React/Vue/Flutter 成熟
  - 调试工具链仍在完善
  - 部分高级功能文档不完善
  - 样式系统不如 CSS 灵活

## How — 怎么用

### 1. @Component 与 @Entry

```typescript
// @Entry 标记页面入口，@Component 标记自定义组件
@Entry
@Component
struct HomePage {
  @State message: string = 'Hello ArkUI'

  build() {
    Column() {
      Text(this.message)
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // 使用自定义组件
      MyButton(text: 'Click', onClick: () => {
        this.message = 'Clicked!'
      })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}

// 自定义组件
@Component
struct MyButton {
  @Prop text: string              // 父到子单向传递
  onClick: () => void = () => {}  // 回调函数

  build() {
    Button(this.text)
      .width(120)
      .height(40)
      .onClick(() => this.onClick())
  }
}
```

### 2. 基础组件

```typescript
// ===== Text 文本 =====
Text('Hello HarmonyOS')
  .fontSize(20)
  .fontColor(Color.Orange)
  .fontWeight(FontWeight.Bold)
  .textAlign(TextAlign.Center)
  .maxLines(2)
  .textOverflow({ overflow: TextOverflow.Ellipsis })
  .decoration({ type: TextDecorationType.Underline, color: Color.Blue })

// Span 富文本
Text() {
  Span('Hello ')
    .fontSize(16)
    .fontWeight(FontWeight.Bold)
  Span('ArkUI')
    .fontSize(16)
    .fontColor(Color.Blue)
    .decoration({ type: TextDecorationType.Underline })
}

// ===== Button 按钮 =====
Button('Primary', { type: ButtonType.Capsule })
  .width('80%')
  .height(48)
  .fontSize(16)
  .backgroundColor('#007DFF')
  .onClick(() => console.log('Clicked'))

// 自定义按钮
Button() {
  Row() {
    Image($r('app.media.icon')).width(20).height(20)
    Text('With Icon').fontSize(14).fontColor(Color.White)
  }.justifyContent(FlexAlign.Center)
}
.width(160).height(40)

// ===== Image 图片 =====
Image($r('app.media.banner'))      // 资源引用
  .width('100%')
  .height(200)
  .objectFit(ImageFit.Cover)       // 裁剪模式
  .interpolation(ImageInterpolation.High)
  .alt($r('app.media.placeholder')) // 占位图

Image('https://example.com/photo.jpg')  // 网络图片
  .width(100).height(100)
  .onComplete(() => console.log('Loaded'))
  .onError(() => console.log('Error'))

// ===== TextInput 输入框 =====
TextInput({ placeholder: '请输入用户名' })
  .width('80%')
  .height(48)
  .fontSize(16)
  .type(InputType.Normal)
  .caretColor(Color.Blue)
  .onChange((value) => {
    this.username = value
  })
  .onSubmit(() => {
    console.log('Submitted: ' + this.username)
  })

// 密码输入
TextInput({ placeholder: '密码' })
  .type(InputType.Password)
  .showPasswordIcon(true)

// ===== TextArea 多行输入 =====
TextArea({ placeholder: '请输入内容' })
  .width('80%')
  .height(120)
  .onChange((value) => this.content = value)
```

### 3. 容器组件

```typescript
// ===== Column 垂直布局 =====
Column() {
  Text('Item 1')
  Text('Item 2')
  Text('Item 3')
}
.width('100%')
.height('100%')
.justifyContent(FlexAlign.Center)     // 主轴对齐
.alignItems(HorizontalAlign.Center)   // 交叉轴对齐
.space(10)                            // 子元素间距

// ===== Row 水平布局 =====
Row() {
  Text('Left')
  Text('Center')
  Text('Right')
}
.width('100%')
.justifyContent(FlexAlign.SpaceBetween)
.alignItems(VerticalAlign.Center)

// ===== Stack 层叠布局 =====
Stack() {
  Image($r('app.media.bg')).width('100%').height(200)
  Text('Overlay Text')
    .fontColor(Color.White)
    .fontSize(20)
}
.width('100%').height(200)
.alignContent(Alignment.BottomCenter)  // 子元素默认对齐

// ===== Flex 弹性布局 =====
Flex({ direction: FlexDirection.Row, wrap: FlexWrap.Wrap, justifyContent: FlexAlign.Start }) {
  Text('1').width('30%').height(60).backgroundColor(Color.Orange)
  Text('2').width('30%').height(60).backgroundColor(Color.Green)
  Text('3').width('30%').height(60).backgroundColor(Color.Blue)
}

// ===== RelativeContainer 相对布局 =====
RelativeContainer() {
  Text('Center')
    .id('center')
    .alignRules({ center: { anchor: '__container__', align: VerticalAlign.Center } })
}
.width('100%').height(200)
```

### 4. 列表与网格

```typescript
// ===== List 列表 =====
List() {
  ForEach(this.items, (item: Item) => {
    ListItem() {
      Row() {
        Text(item.name).fontSize(16).fontWeight(FontWeight.Medium)
        Text(item.desc).fontSize(12).fontColor('#999999')
      }
      .width('100%')
      .padding(16)
    }
    .onClick(() => this.navigateToDetail(item.id))
  }, (item: Item) => item.id)
}
.width('100%')
.height('100%')
.divider({ strokeWidth: 1, color: '#E5E5E5' })
.edgeEffect(EdgeEffect.Spring)    // 弹性滚动
.onReachEnd(() => this.loadMore())

// ===== Grid 网格 =====
Grid() {
  ForEach(this.products, (product: Product) => {
    GridItem() {
      Column() {
        Image(product.image).width('100%').height(120).objectFit(ImageFit.Cover)
        Text(product.name).fontSize(14).margin({ top: 8 })
        Text(`¥${product.price}`).fontSize(14).fontColor(Color.Orange)
      }
      .padding(8)
    }
  }, (product: Product) => product.id)
}
.columnsTemplate('1fr 1fr')       // 两列
.rowsGap(8)
.columnsGap(8)
.width('100%')
.height('100%')

// ===== LazyForEach 懒加载 =====
// 适合大数据量列表，只渲染可见项
List() {
  LazyForEach(this.dataSource, (item: Item) => {
    ListItem() {
      ItemCard({ item: item })
    }
  }, (item: Item) => item.id)
}
.cachedCount(5)                   // 缓存项数
```

### 5. @Builder 复用

```typescript
// ===== 全局 @Builder =====
@Builder function ItemBuilder(text: string, onClick: () => void) {
  Row() {
    Text(text).fontSize(16)
    Blank()
    Image($r('sys.media.ohos_ic_public_arrow_right'))
      .width(16).height(16)
  }
  .width('100%')
  .padding({ left: 16, right: 16, top: 12, bottom: 12 })
  .onClick(onClick)
}

// 使用
Column() {
  ItemBuilder('设置', () => router.pushUrl({ url: 'pages/Settings' }))
  ItemBuilder('关于', () => router.pushUrl({ url: 'pages/About' }))
}

// ===== 组件内 @Builder（可访问 this）=====
@Component
struct MyList {
  @State items: string[] = ['A', 'B', 'C']

  @Builder ItemBuilder(item: string, index: number) {
    Row() {
      Text(`${index + 1}. ${item}`).fontSize(16)
    }
    .padding(12)
  }

  build() {
    List() {
      ForEach(this.items, (item: string, index: number) => {
        ListItem() {
          this.ItemBuilder(item, index)
        }
      })
    }
  }
}
```

### 6. @Extend 和 @Styles 样式复用

```typescript
// ===== @Extend — 扩展组件样式（可传参）=====
@Extend(Text) function titleText(size: number = 20, color: ResourceColor = Color.Black) {
  .fontSize(size)
  .fontColor(color)
  .fontWeight(FontWeight.Bold)
  .maxLines(1)
  .textOverflow({ overflow: TextOverflow.Ellipsis })
}

// 使用
Text('Title').titleText()
Text('Subtitle').titleText(16, '#666666')

// ===== @Styles — 通用样式（不可传参，只能写通用属性）=====
@Styles function cardStyle() {
  .width('100%')
  .padding(16)
  .backgroundColor(Color.White)
  .borderRadius(12)
  .shadow({ radius: 4, color: '#1A000000', offsetX: 0, offsetY: 2 })
}

// 使用
Column() {
  Text('Card 1')
}.cardStyle()

Row() {
  Text('Card 2')
}.cardStyle()
```

### 7. 条件渲染与循环渲染

```typescript
// ===== if/else 条件渲染 =====
@Component
struct StatusView {
  @State isLoading: boolean = true
  @State hasError: boolean = false
  @State data: string = ''

  build() {
    Column() {
      if (this.isLoading) {
        LoadingProgress().width(40).height(40).color(Color.Blue)
      } else if (this.hasError) {
        Text('加载失败').fontColor(Color.Red)
        Button('重试').onClick(() => this.retry())
      } else {
        Text(this.data).fontSize(16)
      }
    }
    .width('100%')
    .height(200)
    .justifyContent(FlexAlign.Center)
  }
}

// ===== ForEach 循环渲染 =====
// 基本用法
ForEach(this.items, (item: Item) => {
  Text(item.name)
}, (item: Item) => item.id)    // 第三个参数是 keyGenerator

// ===== LazyForEach 懒加载（大数据量）=====
// 需要实现 IDataSource 接口
class MyDataSource implements IDataSource {
  private dataList: Item[] = []

  totalCount(): number { return this.dataList.length }
  getData(index: number): Item { return this.dataList[index] }

  registerDataChangeListener(listener: DataChangeListener): void {}
  unregisterDataChangeListener(listener: DataChangeListener): void {}
}
```

### 8. 组件生命周期

```typescript
@Component
struct LifecycleDemo {
  @State count: number = 0

  // 组件即将出现（在 build 之前）
  aboutToAppear() {
    console.log('aboutToAppear')
    // 初始化数据、启动定时器
  }

  // 组件即将销毁
  aboutToDisappear() {
    console.log('aboutToDisappear')
    // 清理资源、取消定时器
  }

  // 页面显示时（@Entry 组件）
  onPageShow() {
    console.log('onPageShow')
  }

  // 页面隐藏时
  onPageHide() {
    console.log('onPageHide')
  }

  // 返回键按下时
  onBackPress(): boolean {
    console.log('onBackPress')
    return false  // true 表示拦截返回键
  }

  build() {
    Column() {
      Text(`Count: ${this.count}`)
      Button('Increment').onClick(() => this.count++)
    }
  }
}
```

### 9. 事件处理

```typescript
// ===== 点击事件 =====
Button('Click')
  .onClick((event: ClickEvent) => {
    console.log(`Clicked at: ${event.x}, ${event.y}`)
  })

// ===== 触摸事件 =====
Text('Touch Me')
  .onTouch((event: TouchEvent) => {
    if (event.type === TouchType.Down) {
      console.log('Touch Down')
    } else if (event.type === TouchType.Up) {
      console.log('Touch Up')
    }
  })

// ===== 手势 =====
Text('Swipe Me')
  .gesture(
    SwipeGesture({ direction: SwipeDirection.Horizontal })
      .onAction((event: SwipeGestureEvent) => {
        console.log(`Swipe angle: ${event.angle}`)
      })
  )

// 长按手势
Text('Long Press')
  .gesture(
    LongPressGesture({ repeatInterval: 500 })
      .onAction((event: GestureEvent) => {
        console.log('Long pressed')
      })
  )

// 捏合手势
Image($r('app.media.photo'))
  .gesture(
    PinchGesture()
      .onActionStart(() => console.log('Pinch start'))
      .onActionUpdate((event: GestureEvent) => {
        console.log(`Scale: ${event.scale}`)
      })
  )

// 拖拽
Text('Drag Me')
  .gesture(
    PanGesture()
      .onActionStart(() => console.log('Pan start'))
      .onActionUpdate((event: GestureEvent) => {
        console.log(`Offset: ${event.offsetX}, ${event.offsetY}`)
      })
  )
```

### 10. 动画

```typescript
// ===== 属性动画（隐式）=====
@Component
struct AnimateDemo {
  @State scale: number = 1
  @State opacity: number = 1
  @State rotate: number = 0

  build() {
    Column() {
      Text('Animate')
        .fontSize(24)
        .scale({ x: this.scale, y: this.scale })
        .opacity(this.opacity)
        .rotate({ angle: this.rotate })
        .animation({                          // 属性动画配置
          duration: 300,
          curve: Curve.EaseInOut,
          delay: 0,
          iterations: 1,
        })

      Button('Scale').onClick(() => this.scale = this.scale === 1 ? 1.5 : 1)
      Button('Rotate').onClick(() => this.rotate += 90)
      Button('Fade').onClick(() => this.opacity = this.opacity === 1 ? 0.3 : 1)
    }
  }
}

// ===== animateTo 显式动画 =====
Button('Animate')
  .onClick(() => {
    animateTo({ duration: 500, curve: Curve.Spring }, () => {
      this.scale = 1.5
      this.opacity = 0.5
    })
  })

// ===== 转场动画 =====
// 页面转场
@Entry
@Component
struct PageTransitionDemo {
  @State show: boolean = true

  build() {
    Column() {
      if (this.show) {
        Text('Content')
          .transition({ type: TransitionType.Insertion, opacity: 0 })
          .transition({ type: TransitionType.Deletion, opacity: 0 })
      }
      Button('Toggle').onClick(() => {
        animateTo({ duration: 300 }, () => {
          this.show = !this.show
        })
      })
    }
  }
}

// 共享元素转场
// 页面 A
Image($r('app.media.photo'))
  .sharedTransition('photo_transition', { duration: 300, curve: Curve.EaseInOut })

// 页面 B（相同 sharedTransition id）
Image($r('app.media.photo'))
  .sharedTransition('photo_transition', { duration: 300, curve: Curve.EaseInOut })
```

### 11. 自定义绘制（Canvas）

```typescript
@Entry
@Component
struct CanvasDemo {
  private settings: RenderingContextSettings = new RenderingContextSettings(true)
  private context: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings)

  build() {
    Column() {
      Canvas(this.context)
        .width('100%')
        .height(300)
        .onReady(() => {
          this.drawProgress()
        })

      Button('Redraw').onClick(() => this.drawProgress())
    }
  }

  private drawProgress() {
    const ctx = this.context
    const width = 300
    const height = 300
    const centerX = width / 2
    const centerY = height / 2
    const radius = 100

    ctx.clearRect(0, 0, width, height)

    // 背景圆
    ctx.beginPath()
    ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI)
    ctx.strokeStyle = '#E5E5E5'
    ctx.lineWidth = 12
    ctx.stroke()

    // 进度圆
    ctx.beginPath()
    ctx.arc(centerX, centerY, radius, -Math.PI / 2, -Math.PI / 2 + Math.PI * 1.5)
    ctx.strokeStyle = '#007DFF'
    ctx.lineWidth = 12
    ctx.lineCap = 'round'
    ctx.stroke()

    // 中心文字
    ctx.font = '32px sans-serif'
    ctx.fillStyle = '#333333'
    ctx.textAlign = 'center'
    ctx.fillText('75%', centerX, centerY + 10)
  }
}
```

### 12. 布局与适配

```typescript
// ===== 尺寸单位 =====
// vp — 虚拟像素，密度无关（推荐）
// fp — 字体像素，跟随系统字体缩放
// lpx — 逻辑像素，按设计稿比例换算
// px — 物理像素

// ===== 响应式断点 =====
// sm: 0-600vp (手机)
// md: 600-840vp (平板竖屏)
// lg: 840+vp (平板横屏/PC)

@Component
struct ResponsiveLayout {
  @State currentBreakpoint: string = 'sm'

  build() {
    Column() {
      if (this.currentBreakpoint === 'sm') {
        // 手机布局：单列
        Column() {
          Text('Content 1')
          Text('Content 2')
        }
      } else {
        // 平板/PC布局：双列
        Row() {
          Text('Content 1').layoutWeight(1)
          Text('Content 2').layoutWeight(1)
        }
      }
    }
    .width('100%')
    .height('100%')
    .onAreaChange((oldValue: Area, newValue: Area) => {
      const width = newValue.width as number
      this.currentBreakpoint = width < 600 ? 'sm' : (width < 840 ? 'md' : 'lg')
    })
  }
}

// ===== 百分比布局 =====
Row() {
  Column() {
    Text('30%')
  }.width('30%').backgroundColor(Color.Orange)

  Column() {
    Text('70%')
  }.width('70%').backgroundColor(Color.Green)
}.width('100%')

// ===== layoutWeight 弹性权重 =====
Row() {
  Text('1/4').layoutWeight(1)
  Text('3/4').layoutWeight(3)
}.width('100%')
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 组件不更新 | 状态变量未用装饰器 | 确保 @State/@Prop/@Link 标注 |
| ForEach 闪烁 | key 不稳定 | 使用唯一 id 作为 key |
| 动画卡顿 | 属性动画配置不当 | 减少动画属性，使用硬件加速 |
| 列表滑动卡顿 | LazyForEach 使用不当 | 合理设置 cachedCount |
| Canvas 不显示 | onReady 未触发 | 确保在 onReady 回调中绘制 |
| 布局溢出 | 未设约束 | 使用 .constraintSize 限制 |
| 样式不生效 | @Extend 参数类型不匹配 | 检查参数类型和默认值 |

### 最佳实践

- 组件拆分到单一职责，每个 @Component 只做一件事
- 用 @Builder 提取重复 UI 片段
- 用 @Extend 统一组件样式
- 列表用 LazyForEach + cachedCount 优化性能
- 状态管理选最小权限装饰器（能用 @Prop 不用 @Link）
- 动画用 animateTo 而非属性动画（控制更精确）
- 资源用 $r() 引用，不硬编码
- 响应式布局用断点系统适配多设备

## 面试题

**Q1: ArkUI 的声明式 UI 和传统命令式 UI 有什么区别？**
> 声明式 UI 描述"UI 应该是什么样"，框架负责将 UI 更新到目标状态。开发者只关注状态和 UI 的映射关系（UI = f(State)），状态变化时框架自动重新执行 build 生成新的组件树并差量更新。命令式 UI 需要手动操作 DOM/视图（findViewById → setText → setVisibility），代码冗余且容易状态不一致。ArkUI 的声明式优势：①状态驱动 UI，无需手动同步；②代码简洁，声明即渲染；③框架自动优化差量更新；④易于理解和维护。

**Q2: @Builder、@Extend、@Styles 三者有什么区别？**
> @Builder 是 UI 复用函数，可以在函数体内写完整的组件声明（包含子组件和事件），类似 React 的 render 函数，支持传参。@Extend 是组件样式扩展，只能为特定组件添加属性方法，支持传参，但不能包含子组件。@Styles 是通用样式提取，只能包含通用属性（width/height/padding/margin 等通用属性），不能传参，不能包含子组件。选择：需要复用 UI 片段（含子组件）用 @Builder；需要为特定组件扩展样式用 @Extend；需要提取通用样式用 @Styles。

**Q3: ForEach 和 LazyForEach 有什么区别？什么时候用哪个？**
> ForEach 会一次性创建所有列表项组件，适合数据量小（<100项）的列表。LazyForEach 懒加载，只创建可见区域和缓存区的列表项组件，滚动时动态创建和销毁，适合大数据量列表。LazyForEach 需要实现 IDataSource 接口（提供 totalCount/getData/registerDataChangeListener 等方法），而 ForEach 直接用数组。性能差异：ForEach 1000项全部创建，LazyForEach 只创建可见的 10-20 项。选择：数据量 < 100 用 ForEach 简单直接；数据量 >= 100 或列表项复杂用 LazyForEach。

**Q4: ArkUI 的组件生命周期有哪些？和 Flutter/React 有什么区别？**
> ArkUI 组件生命周期：aboutToAppear（组件即将出现，build 之前）→ build → aboutToDisappear（组件即将销毁）。页面级还有 onPageShow/onPageHide/onBackPress。对比 Flutter：initState → build → dispose，类似但命名不同。对比 React：constructor → render → componentWillUnmount。关键区别：ArkUI 的 aboutToAppear 在 build 之前执行，适合预加载；Flutter 的 initState 也是 build 之前。ArkUI 的 onPageShow/onPageHide 是页面级生命周期，在 @Entry 组件中有效，类似 Flutter 的 WidgetsBindingObserver。

**Q5: ArkUI 的动画体系是怎样的？animateTo 和属性动画有什么区别？**
> ArkUI 动画分三类：①属性动画（.animation()）— 在组件属性变化时自动执行动画，声明式配置 duration/curve 等，适合简单的属性变化动画；②animateTo 显式动画 — 在回调中修改状态，框架自动对变化属性执行动画，适合需要精确控制时机的场景；③转场动画 — 组件出现/消失时的过渡效果（transition）和共享元素转场（sharedTransition）。animateTo vs 属性动画：animateTo 可以在一次调用中同时修改多个状态并统一动画，属性动画是对单个组件的隐式动画配置。animateTo 适合按钮点击触发的复杂动画，属性动画适合持续响应状态变化的动画。

**Q6: ArkUI 如何实现响应式布局和多设备适配？**
> 三层适配策略：①断点系统（Breakpoint）— 根据屏幕宽度分为 sm(<600vp)/md(600-840vp)/lg(>840vp)，不同断点加载不同布局；②百分比/flex 布局 — 使用 layoutWeight 弹性权重和百分比宽度实现自适应；③资源限定词 — 在 resources 目录下创建不同设备类型的资源目录（如 base/phone/tablet），系统自动选择匹配的资源。最佳实践：布局用 Flex + layoutWeight，断点用 Grid 的 columnsTemplate，资源用资源限定词分发，组件用多态组件（PolymorphicComponent）按设备类型显示不同实现。

**Q7: ArkUI 的手势系统是怎样的？如何处理手势冲突？**
> ArkUI 手势分三类：①绑定手势（.gesture()）— 单指手势，如点击、长按、拖拽、捏合、旋转；②组合手势（GestureGroup）— 串行（Sequence）、并行（Parallel）、互斥（Exclusive）；③原生手势组件 — 如 SwipeGesture/PinchGesture。手势冲突处理：优先级从高到低为 .priorityGesture() > .parallelGesture() > .gesture()。当父子组件手势冲突时，子组件优先；同组件多个手势可用 GestureGroup 的 Exclusive 模式互斥选择。常用模式：下拉刷新用 PanGesture，图片缩放用 PinchGesture，卡片滑动用 SwipeGesture。

**Q8: Canvas 自定义绘制的流程是什么？和 Web Canvas 有什么异同？**
> 流程：创建 CanvasRenderingContext2D → Canvas 组件绑定 context → 在 onReady 回调中绘制。API 与 Web Canvas 高度相似（beginPath/moveTo/lineTo/arc/fill/stroke 等），但差异：①ArkUI Canvas 的 onReady 是绘制时机（确保 Canvas 尺寸已确定），Web Canvas 直接获取 context 即可；②ArkUI Canvas 使用 RenderingContextSettings 开启抗锯齿；③ArkUI 不支持 requestAnimationFrame，动画用 animateTo 或定时器；④ArkUI Canvas 的坐标系统默认左上角为原点。相同点：绑定 API 一致，绘制逻辑可复用，2D 绘图能力基本对齐。

---

**相关链接：**
- [[ArkTS语言核心]]
- [[鸿蒙系统架构与开发入门]]
- [[Flutter核心与Widget体系]]
- [[Vue核心]]
