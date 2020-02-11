# 元部件开发文档


>  文档将按以下顺序讲解：
>
>  - 概览
>    - 开发流程
>    - 注意事项
>    - 简明 API
>  - 例子
>  - 详细内容
>
>  	 1. 目录结构
>  	 2. 元部件依赖库
>    3. `@Widget()` 装饰器
>    4. `WidgetBase` 元部件基类
>    5. 生命周期回调
>    6. 数据源接入
>    7.  可配置属性与 `@property`
>    8. 图片、字体等静态资源使用方式
>    9. 元部件配置文件
>    10. 打包及发布
>
>  - Widget devkit 详细 API
>



## 概览



### 开发流程

1. 执行 `npm i @gspwidget/widget-cli -g` 全局安装元部件脚手架 Widget Cli；
2. 执行 `widget init` 初始化元部件总项目，执行 `widget add` 新增元部件；
3. 新增的元部件根组件为 `<widgets-project>/project/<widget-name>/src/lib/<widget-name>.component.ts`；用 `ng serve` 启动开发服务器，一边开发它一边预览；
4. 开发完成，在 `widget.config.json` 中按需更改配置项，用 `widget build` 打包；
5. 打包结果都在 `<widget-name>/release`，打成同名压缩包后上传至 Dashboard。



### 注意事项

- 元部件的根组件需继承 `WidgetBase` 类，并加入 `@Widget({ name: 'xxx' })` 指定当前元部件名称；
- 打包前记得写好元部件配置文件 `widget.config.json` 的必填字段 `name` 和 `displayName`；其中，`name` 的值必需与 `@Widget()` 中的 `name` 相同；
- 元部件使用的图片等静态资源要放在 `<元部件根项目>/src/app/assets/<元部件名>/` ，引用路径为 `this.assetsBaseUrl + 'xxx.png'`



### 简明 API (Widget-devkit)

- Decorator `Widget`: 元部件根组件定义前加上 `@Widget({name: 'xxx'})` 以指定此根组件及元部件英文名称。
- Decorator `Property`: 元部件根组件的属性定义前加上 `@Property()` 可将其声明为可配置属性。
- Class `WidgetBase`: 元部件根组件需要继承的基类。
  - Property `assetsBaseUrl: string`: 静态资源的 base 路径。
  - Callback `onPropertyChange(propName: string, value)`: 可配置属性的值改变事件回调。
  - Callback `onResized(width: number)`: 元部件容器大小变化事件回调。
  - Callback `onGetData(data: any[])`: 数据获取完成事件回调。
- Service `WidgetDevkitService`: Dashboard 提供的接口方法的集合。
- Component `<widget-title-bar>`: 元部件公共标题栏组件。



## 例子

1. 全局**安装 widget-cli 命令行工具**：

    ```shell
    $ npm i @gspwidget/widget-cli -g
    ```

    

2. 安装完成后进入合适的目录执行以下命令，在此目录内**创建一个元部件总项目**（之后的元部件都在这个总项目中）：

    ```bash
    $ widget init
    # 出现询问输入，填写项目信息
    ```

    假设在 `D:/` 目录下执行命令，填写的总项目名为 `widget-project`，则生成元部件项目 `D:/widget-project`。
    
    创建完成后进入总项目目录，安装元部件项目依赖：
    
    ```shell
    $ npm i
    ```
    
3. 进入元部件项目，执行以下命令，**新增一个元部件**：

    ```shell
    $ widget add
    ```

    假设填写元部件名称为 `widget-weather`，则元部件存放目录为 `D:/widget-project/projects/widget-weather`，开发元部件就是编写元部件目录内的 .ts，.html，.css 文件。

    之后每当要新建一个元部件，就在总项目目录内运行 `widget add` 命令。
    
    

4. 可开启开发服务器，代码变化后自动刷新页面：
   
   ```shell
$ ng serve
   ```
   
   此时已有一个完整的元部件示例正在运行，可在其之上开发并测试。
> 元部件项目就是一个加入了特殊配置的 Angular 项目，利用 widget cli 可以省去手动增改配置的麻烦、元部件打包配置的麻烦。



5. 开发完成后，根据需要修改元部件配置文件 `widget.config.json`，完成后即可打包:

   ```shell
   $ widget build
   ```

   将打包结果 `<元部件名>/release/` 目录整个压缩，提交至 Dashboard，结束。



## 详细内容

### 1. 目录结构

元部件就是一个 [Angular Library](https://angular.cn/guide/libraries)，所以目录结构、开发方式与标准  Angular Library 基本一致。

```typescript
<元部件根项目>/
	projects/
		<元部件名1>
			src
				lib
					<元部件名1>.component.ts
					<元部件名1>.component.html
					<元部件名1>.component.css
				public-api.ts
			...
		<元部件名2>
		...
	package.json
	...
```

注意所有组件、service、directive 等必需在 `<元部件名>/public-api.ts` 中导出：

```typescript
// public-api.ts
export * from './lib/widget-weather.service';
export * from './lib/widget-weather.component';
export * from './lib/widget-weather-another.component';
export * from './lib/widget-weather.module';
```


另外，`<元部件名>/` 目录下还应额外准备如下文件，打包时会一起打入结果目录：

```
<元部件名>/
    logo.png			（logo图片）
    preview.png			（预览图）
    widget.config.json	（元部件的配置文件）
    README.md			（元部件的说明文档）
```



### 2. 元部件依赖库

元部件依赖库照常用 npm 安装即可开发调试：

```bash
npm install change-case
```

默认情况下，开发完成后用 `widget build` 打包时，会将依赖库打包到元部件脚本文件 `main.js` 中，在外部框架内运行时不会缺少依赖。

但如果多个元部件都引用了相同的库（比如 `@angular/core`），把这些公共库打包进每个 `main.js` 会造成不小的空间浪费。把公共库加入到 `<根项目目录>/rollup.config.json` 的 `external` 数组中，则不会被打包到 `main.js`：

```json
{
    external: [
        '@angular/core',
        "@angular/common",
        "@angular/common/http",
        '@angular/forms',
        '@angular/platform-browser',
        '@angular/platform-browser/animations',
        '@angular/animations',
        "@angular/animations/browser",
        'ngx-bootstrap/modal',
        'ngx-bootstrap/accordion',
        "ngx-bootstrap/collapse",
        '@gspwidget/widget-devkit',
        'echarts'
    ]
}
```

并在把公共库及对应版本注册到外部框架（如Dashboard）中，让外部框架加载此依赖。




### 3. @Widget() 装饰器

> 外部框架运行时通常会同时存在多个元部件，每个元部件也可能包含多个组件。

为了让外部框架能找到一个元部件对应的根组件，给根组件加上 `@Widget({name: 'xxx'})`。框架就能根据元部件的 `widget.config.json` 的 `name` 值，在茫茫一堆组件中找到它并渲染：

```js
// widget-example.component.ts
@Widget({ name: 'widget-example' })	// <--
@Component()
export class WidgetExampleComponent extends WidgetBase {
}

// widget.config.json
{
    "name": 'widget-example',		// <--
    "displayName": "示例元部件"
}
```

因此 `@Widget()` 参数中的 `name` ，应与 `widget.config.json` 的 `name` 保持一致。



### 4. `WidgetBase` 元部件基类

元部件组件应继承 `WidgetBase` 基类，提供后面介绍的事件回调等诸多功能。

```js
// widget-example.component.ts
import { Widget, WidgetBase } from '@gspwidget/widget-devkit';
// ...
export class WidgetExampleComponent extends WidgetBase { // <-- 继承 WidgetBase
    // ...
}
```



### 5. 生命周期回调

元部件根组件可使用全部 Angular 生命周期函数:

- `ngOnChanges()`: Angular（重新）设置数据绑定输入属性时响应。
- `ngOnInit()`: Angular 第一次显示数据绑定和设置指令/组件的输入属性之后，初始化指令/组件。
- `ngDoCheck()`: 检测，并在发生 Angular 无法或不愿意自己检测的变化时作出反应。
- [`ngAfterContentInit()`](https://angular.cn/api/router/RouterLinkActive#ngAfterContentInit): 当 Angular 把外部内容投影进组件/指令的视图之后调用。
- `ngAfterContentChecked()`: 每当 Angular 完成被投影组件内容的变更检测之后调用。
- [`ngAfterViewInit()`](https://angular.cn/api/forms/NgForm#ngAfterViewInit): 当 Angular 初始化完组件视图及其子视图之后调用。
- `ngAfterViewChecked()`: 每当 Angular 做完组件视图和子视图的变更检测之后调用。
- `ngOnDestroy()`: 每当 Angular 每次销毁指令/组件之前调用并清扫。 在这儿反订阅可观察对象和分离事件处理器，以防内存泄漏。

详细说明见 [Angular 文档](https://angular.cn/guide/lifecycle-hooks)。
除此之外还可使用 WidgetBase 提供的元部件特有的回调函数：

- `onResized()`: 当元部件容器尺寸变化时调用。可用于触发 `echarts.resize()` 等需要手动触发的操作；参数传入了容器的当前宽度（单位 `px`），所以可实现响应式样式的判断等：

    ```js
    onResized (size: 'sm' | 'md' | 'lg', width: number) {
        // 当宽度大于 300px 时显示头像
        if(width > 300) {
            this.showHead = true
        }
        // 当宽度变化时触发 echart 重新渲染合适大小
        this.chart.resize();
    }
    ```



### 6. 数据源接入



#### 方法一（已废弃，将在 widget-core@0.2.0, widget-devkit@0.7.0 移除）

> 元部件可以亲自在代码中处理数据加载。
>
> 但对于公共的展示类元部件（如柱状图等图表元部件），应将复杂的数据源配置、请求、处理等交由外部框架处理，元部件本身仅负责当获取到新数据时更新渲染。

要使用框架提供的外部数据源，元部件仅需实现数据加载完成的回调函数 `onGetData(data)`  —— 则当数据配置发生变化，外框架会自动加载数据，并在完成后将数据结果通过此函数发送给元部件。举例：

```js
// 当获取到外部数据
onGetData (data: any[]) {
    this.data = data // 保存数据
    this.render() // 用新数据重新渲染元部件内容
}
```

每当获得新数据时，元部件都应保证渲染出正确的新数据内容。

外部数据源接入默认打开，若不需要，可在配置文件中配置 `widget.config.json` 的 `usePublicData` 为 `false` 来关闭。



#### 方法二（推荐，将在 widget-core@0.2.0, widget-devkit@0.7.0 启用）

> 元部件可以亲自在代码中处理数据加载，也可以获得在外部框架配置的数据，本节仅讲解后者。

要使用框架提供的外部数据源，元部件应在合适的初始化完成阶段（一般是 `ngOnInit()` 回调）调用  `WidgetBase.loadData()` 触发加载数据，并监听数据加载成功回调。举例：

```typescript
ngOnInit () {
    // 初始化时触发数据加载
    this.loadData().subscribe(data => {
        this.data = data
        this.render()
    }
}
```

`WidgetBase.loadData()` 返回一个“新数据加载完成”的 `EventEmitter`，每当外部框架加载到新数据时都将触发此事件，调用元部件内订阅的处理函数。当获得新数据时，元部件都应保证渲染出正确的新数据内容。

外部数据源接入默认打开，若不需要，可在配置文件中配置 `widget.config.json` 的 `usePublicData` 为 `false` 来关闭。



### 7. 可配置属性与 @property

给元部件根组件的属性加上 `@property()` 标签，即可将其声明为**可配置属性**。

```typescript
// widget-example.component.ts
@Property() slogan
```

```html
<!-- widget-example.component.html -->
<div>{{slogan}}</div>
```

外部框架会在必要时给元部件的可配置属性生成配置编辑器，用户可以通过编辑器界面，在元部件运行时更改这些属性的值。如上例的 `slogan`，元部件可实时更新显示用户更新的 `slogan` 值。

#### PropertyOption

`@Property(propertyOption?: PropertyOption)` 有一个可选参数 `propertyOption`，包含了丰富的配置项，如：

```typescript
// 给 smooth 属性生成 checkbox 开关表单项（值为 boolean 类型）；label 为“平滑”；在“样式”分组下：
@Property({
    type: PropertyTypes.Bool,
    displayName: '平滑',
    category: '样式'
}) smooth

// 给 icon 属性生成 select 下拉框（值为 string 类型），可选项为 [arrow_right, explore, favorite]：
@Property({
    category: '公共属性',
    displayName: '图标',
    type: PropertyTypes.Enum, // 下拉框
    enumOption: {
        default: 'explore', // 默认值为 explore
        items: [ // select 可选值
            { value: 'arrow_right', displayName: '右箭头' },
            { value: 'explore' },
            { value: 'favorite' }
        ]
    }
}) icon

// 不写 propertyOption 则默认生成 text 型 <input> 输入框（值为 string 类型）；label 为 “slogan”；在“其他”分组下：
@Property() slogan
```

所有可配置项及其可选值见“详细API”。

#### 数组

任何类型都可以被同时设置为数组，以接收不固定长度的、数组类型的值，仅需设置 `isArray` 为 `true`：

```typescript
// 给 colors 生成取色器数组（值为 string[] 类型）：
@Property({
    displayName: '色板'
    type: PropertyTypes.Color,
    isArray: true
}) colors

// 给 map 生成 Object 类型的数组，值为 { valueField: string, color: string }[] 类型。
// 在用户看来，Object 是一组编辑器，其中的每一项都由一个 propertyOption 配置。
@Property({
    displayName: 'Y 轴'
    type: PropertyTypes.Object,
    isArray: true,
    objectOption: {
        objPropertyOptions: [{ // 子 propertyOption，对应 map.valueField
            name: 'valueField',
            displayName: '对应字段',
            type: PropertyTypes.Text
        }, {
            name: 'color',
            displayName: '颜色', // 子 propertyOption，对应 map.color
            type: PropertyTypes.Color
        }]
    }
}) map
```

#### Property 改变事件

若要在 Typescript 中监听可配置属性的值变化，应使用 `onPropertyChange()` 事件回调函数：

```js
// widget-example.component.ts
onPropertyChange (propName: string, value) {
    if(propName == 'slogan' && value == 'Hello INSPUR') {
        console.log('Hello YOU')
    }
}
```

相比于 Angular 的 `ngOnChange()` 回调，`onPropertyChange()` 的优势在于：即使发生改变的属性是引用类型，且这个引用未发生变化（比如数组新增了一个项，或对象的属性值改变），`onPropertyChange()` 依然会被触发调用，而  `ngOnChange()` 不会。



### 8. 图片、字体等静态资源使用方式

Angular 项目中的静态资源文件需要放在 Angular Application 的 `/assets/` 目录下。而元部件是 Angular Library，若元部件需要使用静态资源，应放在 `<元部件根项目>/src/app/assets/<元部件名>/` ，引用路径使用 `this.assetsBaseUrl + 'xxx.png'`:

```html
<div>
	<img [src]="assetsBaseUrl + 'avatar.png'"/>
</div>
```

`this.assetsBaseUrl` 在不同的运行环境中会处理为不同的路径，所以应使用它而不应该手动写作 `<元部件根项目>/src/app/assets/<元部件名>/avatar.png`。



### 9. 元部件配置文件

元部件的注册信息配置文件为 `<元部件名>/widget.config.json`，在外部框架运行时起作用，打包前务必检查配置文件是否填写正确。最简状态的配置文件如下：

```json
/* 可仅保留必填项，未填的可选项会使用默认配置 */
{
	"name": "widget-example",
    "displayName": "实例元部件"
}
```

完整的配置项，含义及默认值如下：

```json
{
    /* 元部件名称，必需与元部件组件的 @Widget() 参数中的 name 一致 */
	"name": "widget-example",
    /* 元部件显示名称，必需 */
    "displayName": "实例元部件",
    /* 元部件描述，可选 */
    "desc": "",
    /* 版本号，可选，以下为默认值 */
    "version": "0.0.1",
    /* 元部件提供的资源文件路径，可选，以下为默认值 */
    "paths": {
        "main": "./main.js", 
        "logo": "./logo.png",
        "preview": "./preview.png"
    },
    /* 元部件尺寸范围，可选，以下为默认值 */
    "layout": {
        /* 栅格布局 */
        "grid": {
            "rows": 4, /* 理想尺寸 */
            "cols": 4,
            "minCols": 0, /* 最小尺寸 */
            "minRows": 0,
            "maxCols": 24, /* 最大尺寸 */
            "maxRows": 24
        },
        /* 其他布局 */
        "other": {
        }
    },
    /* 是否使用外部框架数据源，以下为默认值 */
    "usePublicData": true
}
```



### 10. 打包及发布

在元部件总项目下执行 `widget build [widgetName]` 打包。（不填写 `widgetName` 可在下一步的下拉列表中选择元部件。）

打包结果是 `<元部件名>/release` 目录，结构如下：

```
<元部件名>/
	release/
        main.js					-- 元部件脚本，对 library aot 打包生成
        assets/					-- 元部件的静态资源目录，来自<元部件根项目>/src/app/assets/<元部件名>
        logo.png				-- 来自<元部件名>/logo.png
        preview.png				-- 来自<元部件名>/preview.png
        widget.config.json		-- 来自<元部件名>/widget.config.json
        README.md				-- 来自<元部件名>/widget.config.json
        
```

打包完成后即可对 `release/` 目录打压缩包，将生成的 `release.zip` 上传。





## Widget devkit 详细 API

> 以下 API 全部来自 `@gspwidget/widget-devkit`。



### Widget `Decrator`

装饰器，用于把一个组件标记为元部件根组件，并指定元部件英文名称。

| 选项           | 说明                                             |
| -------------- | ------------------------------------------------ |
| `name: string` | 元部件英文名称，与元部件配置文件中的 `name` 对应 |



### Property `Decrator`

装饰器，用于把一个元部件根组件的属性标记为可配置属性。

| 选项                             | 说明                           |
| -------------------------------- | ------------------------------ |
| `propertyOption: PropertyOption` | 生成 `Property` 编辑器的配置项 |



### PropertyOption `interface`

`@Property()` 的参数类型，用于告诉外部框架如何生成的编辑器。

```typescript

export interface PropertyOption {
    name?: string, /** 当前属性名，不需要指定，会自动设置为 property 修饰的属性名 */
    displayName?: string, /** 当前属性的 label，用于展现给用户的名称 */
    type?: PropertyTypes, /** 指定当前属性的编辑器类型、传入值的类型 */
    isArray?: boolean, /** 是否是数组 */
    category?: string, /** 当前属性所在分组名 */
    tip?: string, /** 帮助提示，显示帮助信息 */
    /**
     * 控制当前 item 是否显示。
     * 注意：curPropValues 是当前 property 所在的直接父对象内的（**即与自身平级的**）所有 propValues，不包含爸爸的同辈。
     * 仅应当根据同辈的值控制显示与否，所以某儿子不应该根据爸爸的某同辈而变化。
     * */
    ifShow?: (curPropValues: { [propName: string]: any }) => boolean,
    /** text 类型的独占配置 */
    textOption?: {
        default?: string | string[], /* 默认值 */
        long?: boolean /* 是否是 textarea 长文本 */
    },
    /** bool 类型的独占配置 */
    boolOption?: {
        default?: boolean | boolean[] /* 默认值 */
    },
    /** number 类型的独占配置 */
    numberOption?: {
        default?: number | number[], /* 默认值 */
        min?: number, /* 最小值 */
        max?: number, /* 最大值 */
        step?: boolean /* number input 的步长 */
    },
    /** enum 类型的独占配置 */
    enumOption?: {
        default?: string | string[], /* 默认值 */
        /* enum 的可选项信息 */
        items: {
            value: string, /* 可选项的真实值 */
            displayName?: string /** 可选项的显示值，默认显示 value */
        }[]
    },
    /** color 类型的独占配置 */
    colorOption?: {
        default?: string | string[], /* 默认值 */
    },
    /** object 类型的独占配置 */
    objectOption?: {
        default?: any, /* 默认值 */
        /**
         * 包含的子属性的 propertyOptions，与外层稍有不同:
         * name: 必填，属性名，
         * category: 目录无效,
         * xxxOption.default: 默认值无效
         */
        objPropertyOptions: PropertyOption[]
    },
    /** help 类型的独占配置 */
    helpOption?: {
    },
    /** dataField 类型的独占配置 */
    dataFieldOption?: {
    }
}
```



### PropertyTypes `enum`

可配置属性的类型枚举，用作 `PropetyOption.type`  的值。

| 成员 | 说明     |
| ------------------------------ | ---- |
| `Text: 'textOption'`           | 文本框，产出值类型为 `string` |
| `Bool: 'boolOption'`           | 开关 / checkbox，产出值类型为 `boolean` |
| `Number: 'numberOption'`       | 数字文本框 / slider，产出值类型为 `number` |
| `Enum: 'enumOption'`           | 下拉框 / radio，产出值类型为 `string` |
| `Color: 'colorOption'`         | 取色器 / 色板，产出值类型为 `string` |
| `Object: 'objectOption'`       | 一组编辑器，产出值类型为 `object` |
| `Custom: 'customOption'` <开发中> | 自定义编辑器   |
| `Help: 'helpOption'` <开发中>     | 帮助      |
| `DataField: 'dataFieldOption'` | 接入数据源字段名选择器，产出值类型为 `string`，仅在 `usePublicData` 打开时起作用 |



### WidgetBase `class`

元部件基类。其中常用的有属性 `assetsBaseUrl`, 事件 `onResized()`, `onPropertyChange()`, `onGetData()`。

| 属性                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `@Property() title: string` | 可配置属性，部件名称                                         |
| `@Property() icon: string`  | 可配置属性，部件设置的 header 图标                           |
| `widgetName: any`           | 元部件名                                                     |
| `assetsBaseUrl: string`     | 静态资源路径 base。Typescript 中，必需在 `ngOnInit` 及之后使用。 |

| 事件                                          | 说明                           |
| --------------------------------------------- | ------------------------------ |
| `onResized(width: number)`                    | 元部件容器大小变化事件回调     |
| `onGetData(data: any[]): void` `[Deprecated]` | 获取数据完成事件回调。[已废弃] |

| 方法                              | 说明     |
| --------------------------------- | -------- |
| `loadData(): EventEmitter<any[]>` | 加载数据 |



### WidgetDevkitService `service`

包含了一堆外部框架方法接口。

#### 方法

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `openModal(content: any, config?: ModalOptions): BsModalRef` | 打开 modal 弹窗，其参数、使用与 [Ngx-bootstrap](https://github.com/valor-software/ngx-bootstrap/blob/development/docs/spec/index.md#modals) 一致。 |
| `openMenu(funcId: string, queryParamsArray?: string[][]): void` | 打开功能菜单。                                               |
| `openApp(appId: string): void`                               | 打开应用。                                                   |
| `openSuccessMsg(): void`                                     | 弹出“操作成功”提示。                                         |
| `openErrorMsg(err: string): void`                            | 弹出错误提示。                                               |
| `getData(): any`                                             | 主动调用外部框架获取数据。                                   |
| `jump(jumpProperty: JumpProperty, dataItem?: any): void`     | 根据参数联查跳转。                                           |



### JumpProperty `interface`

联查跳转所需的跳转配置属性对象。

| 属性                  | 说明 |
| --------------------- | ---- |
| `type: JumpTypes`      | 跳转类型 |
| `funcId: string`     | 目标菜单/联查 ID |
| `params: { key: string, valueType: string, fixedValue: string, fieldName: string }[]`  | 路由参数配置 |



### JumpTypes `enum`

跳转类型枚举，用于 `WidgetDevkitService.jump()` 参数中的 `jumpProperty.type`。

| 成员             | 说明                   |
| ---------------- | ---------------------- |
| None = '0',      | 无跳转                 |
| Func = '1',      | 跳到应用 / 联查 / 菜单 |
| ChartTable = '2' | 跳到左图右表页         |



### WidgetTitleBarComponent`component`

元部件的公共标题栏组件，提供了标题，标题旁数字，筛选下拉框，查看更多按钮，设置按钮。若无法满足需求可自行实现标题栏。

#### 选择器

`<widget-title-bar>`

#### 属性

| 属性                                   | 说明             |
| -------------------------------------- | ---------------- |
| `@Input() title: string = ''`          | 标题文字内容     |
| `@Input() icon: string = ''`           | 左侧图标名       |
| `@Input() number: number = 0`          | 标题右侧数字     |
| `@Input() moreUrl: string = ''`        | 联查跳转地址     |
| `@Input() showTitle: boolean = true`   | 是否显示标题文字 |
| `@Input() showNumber: boolean = false` | 是否显示左侧数字 |

#### 事件

| 事件                                                 | 说明                 |
| ---------------------------------------------------- | -------------------- |
| `@Output() onClickSetting = new EventEmitter<any>()` | 点击设置按钮事件回调 |

