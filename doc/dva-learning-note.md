# DVA Learning Notes

## Why Dva

### Start from react

#### react没有解决的问题

- react 的优势
  - 运行效率高：react是一个DOM的抽象层，通过组件构建虚拟DOM,运行效率高
  - 组件化：可以累积业务组件，也可以使用开源组件
- react没有解决的问题
  - 数据流问题
    - 组件之间的通信（通过redux解决）
    - 实现异步逻辑（通过redux-thunk/redux-promise/redux-saga来解决）
  - 路由（react-router）

>> 结论1：光使用react本身是无法编写大型复杂的应用的；redux的适用场景是：多交互、多数据源。

#### when use redux(An example)

|Stage| 需求 | 是否需要|
|----|--------------|----------|
|1| 父组件把自身状态作为属性传递给子组件，单向数据流 | 不需要|
|2| 非父子组件需要共享一些状态，状态提升 |可以不需要|
|3| 随着组件和功能的增多，程序流程变更加复杂| 开始需要 |

#### 流行的react实践方案

- 数据流方案： redux
- 异步操作： redux-sage
- 路由：react-router

一个简单的例子 react + redux + react-router

```js
// view(page)
class CoursesPage extends React.Component {
    ...
    render() {...};
}
const mapStateToProps = (state, ownProps) =>({...});
const mapDispatchToProps = dispatch = > ({...});
export default connect(
    mapStateToProps,
    mapDispatchToProps
)(CoursesPage);

// redux
// create action
export const action1 = state1 => ({ type: type1, ...state1 });
export const action2 = () => dispatch => beginCall().then(state2 => dispatch(action2(state2))).catch(() =>{});

// create reducer
const reducer1 = (state = initialState, action) => newState;
const rootReducer =  combineReducers({
    state1: reducer1,
    state2: reducer2,
});

// create store
const store = createStore(rootReducer, [initialState], applyMiddleware(...middleware));

// router
const App = () => (
    <Router>
        <Route path="/" exact component={HomePage} />
        <Route path="/courses" component={CoursesPage} />
    </Router>
)
```

**痛点**

1. 文件组织结构复杂
   - 引入redux后，需要写action\reducer，还需要在component中connect
   - 文件分离，需要来回切换，编辑（增删改）成本高
2. 不便于组织业务模型
   - 目前的action\reducer按照视图结构进行组织的
   - 不好进行动态扩展
   - [Mobile opportunity example](https://msasg.visualstudio.com/Bing_Ads/_git/BingAdsMobile?path=%2Fapp%2Fredux%2Freducers%2Fopt-app%2Fopt-reducer.js&version=GBmaster)
3. 对大项目，难有动态加载方案

**继续讨论关于异步操作的问题**

1. what's redux-sage
   - 通过ES6 的 [Generator](http://es6.ruanyifeng.com/#docs/generator) 功能，让异步的流程更易于读取，写入和测试
   - 类似redux-thunk， 不过redux-thunk和redux-promise都相当于改造了action，但是redux-sage是把所有的业务逻辑都放到 saga 里，这样可以让 reducer, action 和 component 都很纯粹
   - An example

        ```js
            // this is a worker, will handle the action
            function *userFetch() {
                try {
                    yield put({ type: USERDATA_REQUEST });
                    let { data } = yield call(request.get, `/users/${uid}`);
                    yield put({ type: USERDATA_SUCCESS, data });
                } catch(e) {
                    yield put({ type: USERDATA_ERROR, error });
                }
            }
            // this is a watcher, will watch for dispatch actions
            function *userFetchWatcher() {
                takeEvery('user/fetch', userFetch);
            }
            // a task is like a process running in background
            function *rootSaga() {
                yield fork(userFetchWatcher);
            }
        ```
2. sage优势(优雅且强大)
   - 可以把所有的业务逻辑都放到sage中，让reducer,action和component都很纯粹
   - sage可以更好的支持长流程的业务逻辑
   - redux-sage可以更好的支持组合、取消、多任务调度等复杂操作。
3. sage缺点
   - 监听一个 action 需要走 fork -> watcher -> worker 的流程
   - 每个action都书写一遍，会显得很冗余

>> 结论2： react的最佳实践在项目中使用仍旧存在一些痛点和问题。

## What Dva

### dva & umi & antd

umi(工具+路由) + dva(数据) + antd（视图）

- dva: 数据流解决方案。
- umi: 前端底层框架开发方案。
- antd: UI库，前端中台。

dva

### dva basic

- Model 可认为是领域模型，用于把数据相关的逻辑聚合到一起
  - namespace/state/reducers/effects/subscriptions
- State 表示应用的数据层，由 model 的 state 组成全局的 state
- Reducer 是唯一可以修改 state 的地方，接收 state 和 action，返回新的 state
- Effect 用于处理异步逻辑，基于 redux-saga 实现,将异步转成同步写法，从而将effects转为纯函数。
- Subscription 表示订阅，用于订阅一个数据源，然后按需 dispatch action. 数据源可以是当前的时间、服务器的 websocket 连接、keyboard 输入、geolocation 变化、history 路由变化等等.
- Router 路由配置信息

>> 结论3： dva没有引入新的概念，只是对数据相关逻辑做了一个聚合，产生了dva的model，通过sage将异步的reducer拆分出来作为effct，并且增加了订阅，enhance了路由

### what dva

1. dva-core = redux + redux-saga 数据流方案
   - dva-core 主要解决了 model 的问题，包括 state 管理、数据的异步加载、订阅-发布模式的实现，可以作为数据层在各处使用。
2. dva = dva-core + react-router
   - dva传递了一些初始化数据到 dva-core 所实现的 model 层，在dva-core的基础上，实现了 dva 的 view 层。
   - 并提供了一些dva中常用的方法函数。
      - dynamic 动态加载
      - 以下部分， dva只是搬运工
          - fetch 请求方法
          - saga(数据层处理异步的方法)。
          - react-router-dom和react-router-redux
          - react-redux的connect
3. dva数据流图
   ![dva数据流图](https://raw.githubusercontent.com/sharonrain/Learning-book/master/doc/front/react/assets/ESPP.JPG)

>> 结论4： dva是很薄的一层，主要逻辑在dva-core中，dva-core也可以作为一个独立的数据流解决方案。具体细节可以参考源码解读。

### an dva example

A real dva model

```js
export default {
    namespace: 'authors',
    state: {
        list: [],
    },
    reducers: {
        save(state, { payload: {data: list } }) {
            return { ...state, list };
        },
    },
    effects: {
        *fetch({ payload: { page = 1 } }, { call, put }) {
            const { data, headers } = yield call(authorsService.fetch, { page });
            yield put({
              type: 'save',
              payload: {
                data,
              },
            });
        },
        *create({ payload: values }, { call, put }) {
            yield call(authorsService.create, values);
            yield put({ type: 'reload' });
        },
        *reload(action, { put, select }) {
            const page = yield select(state => state.authors.page) || 1;
            yield put({ type: 'fetch', payload: { page } });
        },
    },
    subscriptions: {
        setup({ dispatch, history }) {
          return history.listen(({ pathname, query }) => {
            if (pathname === '/authors') {
              dispatch({ type: 'fetch', payload: query || { } });
            }
          });
        },
    },
};
```

A dva example

```js
import dva from'dva';

const app = dva(); //创建 dva 实例
app.use(require('dva‐loading')()); // optional装载插件
app.model(require('./models/authors'));// 注册 Model
app.router(require('./router'));// 配置路由
app.start(); //启动dva
```

### Use in Bingads

1. dva core是一个独立的数据解决方案，广泛应用于阿里内部
   bundle size: 24.4kb
   download time: 488ms(50kb/s)
2. 目前CampaignUI对redux的使用情况：
   - 大部分feature都基于一些已经存在的component，但不是react
   - AdsAppsSharedComponent中的模块多是拆分的小模块，偏向于view的实现，只使用react，没有用到redux+react
   - AdsAppsCampaign中有一个feature(brand-management/business-portal-content)用到了react+redux的解决方案
3. 个人观点：
   - 目前dva在Bingads中暂时用不起来，因为暂时没有这样的需求，但dva对于一个有复杂数据流的start up项目确实是个很不错的选择，我们可以在开发新工具的时候使用,用起来体验感也方便
   - 对于Bingads中的新的feature，多数据流且使用到redux的时候可以考虑使用dva-core，实践效果优于redux
   - 对于new ui，new ui都是基于react，如果我们将来会将数据和视图分离，使用redux，可以考虑使用dva，dva对model进行了封装且兼具扩展性，优化了大家的开发体验。

>> 结论5： redux不是必要的，使用redux的场景是：多交互，多数据源。但是如果确定使用了redux，dva-core是个非常更好的选择。

## 源码解读

### what dva-core done

dva-core主入口：
create dva

- Create dva with plugin: 详细见#3

start dva

- GetSaga： 详细见#1
- GetReducer： 详细见#2
- Create store: 详细见#5
- Run subscriptions: 详细见#4
- Set up app.model after app start: 详细见#7

1. 简化了saga书写:getSaga文件将统一对用户输入的 effects 部分的键值对函数进行管理。
   - 统一书写了fork -> watcher -> worker 的流程
   - 允许通过onEffect封装执行。比如 dva-loading 基于此实现了自动处理 loading 状态
   - 统一处理了每个sage的try .. catch
   - 使用时只需要简单配置对应的effect操作即可

        ```js
        {
            effects: {
                *fetch({ uid }, { call, put }) {
                    yield put({ type: USERDATA_REQUEST });
                    let { data } = yield call(request.get, `/users/${uid}`);
                    yield put({ type: USERDATA_SUCCESS, data });
                },
            },
        }
        ```

2. 省略了action的书写：getReducer和handleActions将dva的reducer转化为redux接受的reducer
   - 默认的handlerAction会自动将reducer的namespace + key作为action的type触发相应的reducer
   - 允许自定义handlerActions,例如dva-immer通过改写handlerActions实现了immer reducer
   - model中reducer的配置

        ```js
        {
            save(state, { data: list, total, page }) {
                return { ...state, list, total, page, authors };
            },
        }
        ```
3. 通过Plugin中不同种类的钩子事件来丰富数据流方案
   - 目前支持的hooks类型主要如下：

        ```js
        const hooks = [
            // effect 执行错误或 subscription 通过 done 主动抛错时触发，可用于管理全局出错状态。
            'onError',
            // state 改变时触发，可用于同步 state 到 localStorage，服务器端等
            'onStateChange',
            // 在 action 被 dispatch 时触发，用于注册 redux 中间件。支持函数或函数数组格式。
            'onAction',
            // 热替换相关，目前用于 babel-plugin-dva-hmr
            'onHmr',
            // 封装 reducer 执行。比如借助 redux-undo
            'onReducer',
            // 封装 effect 执行。比如 dva-loading 基于此实现了自动处理 loading 状态
            'onEffect',
            // 指定额外的 reducer，比如 redux-form 需要指定额外的 form reducer
            'extraReducers',
            // 指定额外的 StoreEnhancer ，比如结合 redux-persist 的使用
            'extraEnhancers',
            // 定义/转化reducer
            '_handleActions',
        ];
        ```
4. subscription
   - 订阅一个数据源，然后按需 dispatch action
   - 包含取消订阅函数
5. createstore
   - 创建带中间件cratePromiseMiddleware, sagaMiddleware, extraMiddlewares(onAction)的store
6. cratePromiseMiddleware: 自定义的 redux 插件,用于支持effect, 方便在视图层 dispatch action 并处理回调

   ```js
   dispatch({ type: 'count/addAsync' })
    .then(() => {
        console.log('done');
    });
   ```

7. app start后，可以通过app.model来inject 新的model, 通过app.unmodel来unregister model, app.replaceModel来替换model
   - inject的过程和index中start的过程相似，getreducer => replace store reducer => run sage => run subscription
   - unmodel的过程， deleteReducer => cancel effect => unlisten subscriptions
   - replace old, delete old model => inject new model

### what dva done

index 文件

1. 使用 call 给 dva-core 实例化的 app(这个时候还只有数据层) 的 start 方法增加了一些新功能（或者说，通过代理模式给 model 层增加了 view 层）
2. 使用 react-redux的Provider为react提供了store context, 从而使得被代理的组件实现了从 context 中获得 store 的方法。
3. 添加了 redux 的中间件 react-redux-router，强化了 history 对象的listen功能。

dynamic文件

1. 动态load model和component，load完成后resolve返回AsyncComponent即需要加载的Component

    ```js
    import dynamic from 'dva/dynamic';

    const UserPageComponent = dynamic({
    app, // router所挂载的实例
    models: () => [
        import('./models/users'),
    ],
    component: () => import('./routes/UserPage'),
    });
    ```
2. react-router-4动态加载的实现, 参考[react-router-4 bundle loader](https://juejin.im/entry/58fdc74aa0bb9f0065c8ad6b)
