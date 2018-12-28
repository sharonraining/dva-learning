# Building-Applications-with-React-and-Redux-in-ES6

## Introduce

### why redux(comprehension)

1. one store: easy to understand.
2. reduced boilerplate
3. Isomorphic/Universal Friendly
4. Immutable Store
5. Hot Reloading
6. Time-travel debugging
7. Small

## Environment Setup

1. the version use:
    - react:15.0.2
    - redux 3.5.2
    - react/router 2.4.0
    - webpack 1.13
    - babel 6.*

2. hot reloading
    - 更强大的babel plugin： React Transform
        - 以前reloading缺点：每个component的修改都会丢失状态
        - Babel-plugin-react-transform
        - 是基本的架子，可以通过它完成各种transform，如果想实现Hot Module Replacement (页面不刷新，直接替换修改的Component)，再安装一个transform. （react-transform-hmr）
        - 在页面上直接显示catch到的错误的transform （react-transform-catch-errors）
    - 让新建的两个transform生效,只需再安装一个present： babel-preset-react-hmre, 添加到.babelrc中
        ```js
        "env": {
        "development": {
            "presets": ["react-hmre"]
        }
        ```

3. npm 和gulp 对比
    - npm比gulp的优势
        - gulp存在对插件的依赖，新的版本发布时候，还要依赖新的gulp插件的更新
            - 相比之下，npm直接使用新工具，没有额外的抽象
        - 由于集成失败，和额外的抽象层，一个错误对gulp存在更多原因，调试艰难
        - 脱节文档，需要在插件和抽象工具之间上下文切换
            - npm脚本中读取和实现一个简短的命令行调用更清晰，更低的摩擦，更容易调试
    - npm被忽略的原因
        - 认为npm的脚本需要强大的命令行技能（grep，sed，awk和pipe是值得学习的技能）
        - 认为npm本省不够强大（npm的强大依赖于pre and post hooks）
        - gulp的streams是快速构建所必需的(通过pipe(|)和redirect（>）实现stream，通过&实现两个命令同时执行)
        - npm脚本不能跨平台运行（许多项目都与特定的操作系统相关联，因此跨平台问题并不重要。）
    - npm的痛点
        - JSON规范不支持注释（下面为解决方案）
            - 编写名称小，命名良好的单一用途脚本
            - 调用单独的.js文件

4. webpack configuration
   - webpack 核心概念
     - 入口
        - 通过entry配置，表明webpack应该用哪些模块来构建内部依赖图的开始
     - 出口
        - 表明 webpack 在哪里输出它所创建的 bundles 以及如何命名这些文件
     - loader
        - webpack 自身只支持 JavaScript, loader 能够让 webpack 处理那些非 JavaScript 文件
            - 转换为有效模块=>添加到依赖图中=>提供给应用程序使用
            - loader 能够 import 导入任何类型的模块
        - loader 有两个特征
            - test 属性，用于标识出应该被对应的 loader 进行转换的某个或某些文件
            - use 属性，表示进行转换时，应该使用哪个 loader。
     - 插件
        - 用于执行范围更广的任务，插件的范围包括：打包优化、资源管理和注入环境变量。

5. editor configuration
   - 通过项目中.editorconfig 的文件定义该项目的编码规范
     - charset: 文件编码，可选latin1,utf-8(usually use), utf-16be, utf-16le
     - indent_style: 缩进类型, 可选space, tab
     - indent_size: 缩进数量, 可选整数。一般设置 2 或 4。tab
     - insert_final_newline： 是否在文件的最后插入一个空行。 true/false
     - end_of_line：换行符格式,可选： lf/crlf/cr
     - trim_trailing_whitespace: 是否删除行尾的空格。true/false
     - [know more](https://github.com/editorconfig/editorconfig/wiki/EditorConfig-Properties)
   - 安装与编辑器对应的 EditorConfig 插件。

6. express
    - 小规模的灵活的 Node.js Web 应用程序开发框架
    - [Doc](http://www.expressjs.com.cn/)

## React component

1. Stateless functional components: 9 benefits
    - No class needed
    - Avoid this keyword
    - Enforced best practices
    - High signal-to-noise ration
    - Enhanced code completion/intellisense
    - Bloated components are obvious
    - Easy to understand
    - Easy to test
    - Performance

2. when should I use class/stateless components

   | Class component | Stateless Components |
   | -------- | -------|
   | State | Everywhere else |
   | Refs | |
   | Lifecycle methods | |
   | Child functions(for performance) | |
   | State | |

3. container vs presentation components(most components)

   | Container component | Presentation Components |
   | -------- | -------|
   | Little to no markup | Nearly all markup |
   | Pass data and actions down |Receive data and actions via pros |
   | Knows about Redux | Doesn't know about Redux |
   | Often stateful | Typically functional components|

   | Container component | Presentation Components |
   | -------- | -------|
   | Container | Presentational |
   | Smart |Dumb |
   | Stateful |Stateless |
   | Controller View| View|

   when you notice that some compoennts don't use props they receive but merely forward them down...it's a good time to introduce some container components

## Introduce to Redux

1. when do I need redux?
   - Complex data flows
   - Inter-component communication
   - Non-heirarchical data
   - May actions
   - Same data used in multiple places

2. 3 Redux Principles
   - One immutable store
   - Actions trigger changes
   - Reducers update state

3. Flux vs Redux
   - Similarities
     - Unidirectional Flow: Data down & action up
     - Actions
     - Stores(Flux allow multiple store while redux have one)
   - Difference
     - Redux
        - Reducers are pure functions
        - Contianers
        - Immutability
   - Flux
     - Action->Dispatcher->Store->React->Action

   | Flux | Redux |
   | -------- | -------|
   | Stores contain state and change logic | Store and change logic are separate |
   | Multiple stores | One store |
   | Flat and disconnected stores | Single store with hierarchical reducers |
   | Singleton dispatcher| No dispatcher|
   | React components subscribe to stores |Container components utilize connect |
   | State is mutated| State is immutated|

4. Immutable
    - when changing the state, it will return a new  object
    - Immutable already in js: Number,String,Boolean, Undefined, NUll
    - Mutable: Objects, Array, Functions
    - **Redux depend on immutable state to improve performance**
        - Clarity(Immutability = Clarity)
        - Performance(don't need to check in one by one)
        - Awesome Sauce
    - Enforce immutability
        - redux-immutable-state-invariant(only in debug, since influence performance)
        - libary: immutable.js

5. Actions, Stores, and Reducers
    - Actions: Represent user intent and must have a type
    - Store: dispatch/subsbribe/getState
    - Immutability: just return a new copy
    - Reducers: Must be pure(Mutate/side effects/no-pure function are forbidden)/Multiple per app/Slice of state

## Test

1. Mocha: highly configurable and have ecosystem
2. Jasmine: similar to mocha
3. Jest: A wrapper from Jasmine

Asserts:

1. Chai: wrapper based api
2. Expect

Helper library:

1. React testing library(Facebook)
    - shallowRender
        - Render single component
        - No DOM required
        - Fast and simple
    - Render Into Document
        - Render component and children
        - DOM required
        - Supports simulating interactions
    - DOM interactions
        - findRenderedDOMComponentWithTag
        - scryRenderedDOMComponentsWithTag
    - Simulate
        - Clicks/Keypresses etc.

2. Enzyme
    - React Test Utils
    - jSDOM(In-memory DOM)
    - Cheerio(Fast Jquery style selectors) 
