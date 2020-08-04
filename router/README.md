## 说明

对微信路由进行了封装和扩展（设计思路主要参考 vue router）

- 路由配置表
- api 支持 promise
- 任意类型的路由传参
- 导航守卫

## 创建一个路由管理器

```js
const config = { homePage, tabPageMap, routeMap }

const router = createSmRouter(config)
```

[查看 homePage、tabPageMap 定义](/router.md#homePage)

## 路由配置表（routeMap）

routeMap 是一个对象（key / value），所有页面的路由都在此配置。其中 key 为页面名（page），value 为路由配置信息（config），例如：

```js
routeMap = {
  CourseList: '/pages/tabBar/courseList/CourseList',
  ClassList: { page: 'CourseList', data: { courseType: 'class' } },
  BookList: { path: '/pages/BookList', data: { info: { name: 'roma' } }, meta: 234 },
  CityList: { handler: ({ data }) => ({ path: '/pages/CityList', data }) },
  Courses: 'CourseList'
}
```

config 支持三种数据类型来配置一个路由：Object、function、string

### config（Object）

两种结构：

#### 第一种

| 字段名      | 类型     | 必填 | 说明                                                         |
| :---------- | -------- | ---- | ------------------------------------------------------------ |
| path        | string   | 否   | 返回自 1970-1-1 00:00:00  UTC（世界标准时间）至今所经过的毫秒数 |
| page        | string   | 否   | 页面名                                                       |
| data        | Object   | 否   | 路由传参参数                                                 |
| meta        | any      | 否   | 不属于 data 的额外信息                                       |
| beforeEnter | function | 否   | 查看导航守卫章节                                             |

> 注：`path、page 二选一，page 应该与页面文件名一致`

#### 第二种

| 字段名      | 类型     | 必填 | 说明                                                       |
| :---------- | -------- | ---- | ---------------------------------------------------------- |
| handler     | function | 是.  | ({ data }) => { path, page, data, meta }，具体含义参考上面 |
| beforeEnter | function | 否   | 查看导航守卫章节                                           |

### config（function）

```
({ data }) => ({ path | page, data(可选), meta(可选) } | string)
```

### config（string）

```
path 或者 page，有 / 的被认为是 path
```

## 导航守卫

导航守卫参考 vue-router 的导航守卫，基本保持接口对齐。但有以下区别：

- 支持 promise，可让函数返回一个 promise 代替 next。
- to、from 的具体内容不同
- 解析流程有部分区别

### beforeRouteLeave

- 组件守卫，离开页面时触发

- **形式**：beforeRouteLeave(to, from, next)

- **参数**：

  - `{Object} to` 目标路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
    - jumpMethodName 跳转方式: 'navigateBack', 'navigateTo', 'redirectTo', 'reLaunch', 'switchTab'
    - encode: 是否编码
  - `{Object} from` 原路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
  - `{function} next`
    - **`next()`**: 进入下一个守卫
    - **`next(false)`**: 中断当前的导航
    - **`next({ page, data, meta, jumpMethodName, encode })`**: 跳转到一个不同的页面。当前导航被中断
    - **`next(error)`**: 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调

- **this**：指向离开页面的实例

- **返回值**：promise，无论 rejected 还是 fulfilled ，其值的处理逻辑相同

  - **`true`**: 进入下一个守卫
  - **`false`**: 中断当前的导航
  - **`{ page, data, meta, jumpMethodName, encode }`**: 跳转到一个不同的页面。当前导航被中断
  - **`error`**: 返回一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调

  > `注意`：next 和 返回值 二选一，不然跳转会卡主

- **用法**

  在页面里定义

- **示例**

  ```js
  <script>
    wepy.page({
      beforeRouteLeave(to, from, next) {
        next()
      }
      // 或者
      async beforeRouteLeave(to, from) {
        return true
      }
    })
  </script>
  ```

### beforeEach

- 全局守卫，离开页面时触发

- **形式**：beforeEach(to, from, next)

- **参数**：

  - `{Object} to` 目标路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
    - jumpMethodName 跳转方式: 'navigateBack', 'navigateTo', 'redirectTo', 'reLaunch', 'switchTab'
    - encode: 是否编码
  - `{Object} from` 原路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
  - `{function} next`
    - **`next()`**: 进入下一个守卫
    - **`next(false)`**: 中断当前的导航
    - **`next({ page, data, meta, jumpMethodName, encode })`**: 跳转到一个不同的页面。当前导航被中断
    - **`next(error)`**: 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调

- **this**：指向离开页面的实例

- **返回值**：

  - promise，无论 rejected 还是 fulfilled ，其值的处理逻辑相同

    - **`true`**: 进入下一个守卫
    - **`false`**: 中断当前的导航
    - **`{ page, data, meta, jumpMethodName, encode }`**: 跳转到一个不同的页面。当前导航被中断
    - **`error`**: 返回一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调

    > `注意`：next 和 返回值 二选一，不然跳转会卡主

- **用法**

  全局定义

- **示例**

  ```js
  <script>
  import router from 'router'
  router.beforeEach(function beforeEach(to, from, next) {
    next()
  })
  // 或者
  router.beforeEach(function beforeEach(to, from) {
    return true
  })
  </script>
  ```

### beforeEnter

- 路由守卫

- **形式**：beforeEnter(to, from, next)

- **参数**：

  - `{Object} to` 目标路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
    - jumpMethodName 跳转方式: 'navigateBack', 'navigateTo', 'redirectTo', 'reLaunch', 'switchTab'
    - encode: 是否编码
  - `{Object} from` 原路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
  - `{function} next`
    - **`next()`**: 进入下一个守卫
    - **`next(false)`**: 中断当前的导航
    - **`next({ page, data, meta, jumpMethodName, encode })`**: 跳转到一个不同的页面。当前导航被中断
    - **`next(error)`**: 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调

- **this**：指向离开页面的实例

- **返回值**：

  - promise，无论 rejected 还是 fulfilled ，其值的处理逻辑相同

    - **`true`**: 进入下一个守卫
    - **`false`**: 中断当前的导航
    - **`{ page, data, meta, jumpMethodName, encode }`**: 跳转到一个不同的页面。当前导航被中断
    - **`error`**: 返回一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调

    > `注意`：next 和 返回值 二选一，不然跳转会卡主

- **用法**

  routerConfig 里配置

- **示例**

  ```js
  routerConfig = {
    PersonalDetail: {
      path: '/pagesSubPackage/personal/pages/PersonalDetail',
      beforeEnter: (to, from, next) => next()
    }
  }
  ```

### beforeRouteEnter

- 组件守卫，进入页面时触发

- **形式**：beforeRouteEnter(to, from, next)

- **参数**：

  - `{Object} to` 目标路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
    - jumpMethodName 跳转方式: 'navigateBack', 'navigateTo', 'redirectTo', 'reLaunch', 'switchTab'
    - encode: 是否编码
  - `{Object} from` 原路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
  - `{function} next`
    - **`next()`**: 进入下一个守卫
    - **`next(false)`**: 中断当前的导航
    - **`next({ page, data, meta, jumpMethodName, encode })`**: 跳转到一个不同的页面。当前导航被中断
    - **`next(error)`**: 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调
    - **`next(vm => {})`**：vm 为进入页面的实例

- **this**：由于进入页面还未实例化，因此不能使用，由 vm 代替

- **返回值**：

  - promise，无论 rejected 还是 fulfilled ，其值的处理逻辑相同

    - **`true`**: 进入下一个守卫
    - **`false`**: 中断当前的导航
    - **`{ page, data, meta, jumpMethodName, encode }`**: 跳转到一个不同的页面。当前导航被中断
    - **`error`**: 返回一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调
    - **`vm => {}`**： vm 为进入页面的实例

    > `注意`：next 和 返回值 二选一，不然跳转会卡主

- **用法**

  页面里定义

- **示例**

  ```js
  <script>
    wepy.page({
      beforeRouteEnter(to, from, next) {
        next(false)
      }
      // 或者
      async beforeRouteEnter(to, from) {
        return vm => {}
      }
    })
  </script>
  ```

### beforeResolve

- 全局守卫，以上守卫执行后执行，但在页面跳转前

- **形式**：beforeResolve(to, from, next)

- **参数**：

  - `{Object} to` 目标路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
    - jumpMethodName 跳转方式: 'navigateBack', 'navigateTo', 'redirectTo', 'reLaunch', 'switchTab'
    - encode: 是否编码
  - `{Object} from` 原路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
  - `{function} next`
    - **`next()`**: 进入下一个守卫
    - **`next(false)`**: 中断当前的导航
    - **`next({ page, data, meta, jumpMethodName, encode })`**: 跳转到一个不同的页面。当前导航被中断
    - **`next(error)`**: 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调

- **this**：无 this

- **返回值**：

  - promise，无论 rejected 还是 fulfilled ，其值的处理逻辑相同

    - **`true`**: 进入下一个守卫
    - **`false`**: 中断当前的导航
    - **`{ page, data, meta, jumpMethodName, encode }`**: 跳转到一个不同的页面。当前导航被中断
    - **`error`**: 返回一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调

    > `注意`：next 和 返回值 二选一，不然跳转会卡主

- **用法**

  全局定义

- **示例**

  ```js
  <script>
  import router from 'router'
  router.beforeResolve((to, from, next) => next())
  // 或者
  router.beforeResolve((to, from) => true)
  </script>
  ```

### afterEach

- 全局守卫，进入页面后触发

- **形式**：afterEach(to, from)

- **参数**：

  - `{Object} to` 目标路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息
  - `{Object} from` 原路由
    - page 页面名
    - data 跳转页面携带数据
    - meta 携带额外信息

- **this**：指向进入页面的实例

- **返回值**：

  - 无

- **用法**

  全局定义

- **示例**

  ```js
  <script>
  import router from 'router'
  router.afterEach((to, from) => {})
  </script>
  ```

### beforeRouteUpdate

- 组件守卫，页面内跳转

- **形式**：beforeRouteUpdate(to, from, next)

- **参数**：

  - `{Object} to` 目标路由
    - data 跳转页面携带数据
    - meta 携带额外信息
  - `{Object} from` 原路由
    - data 跳转页面携带数据
    - meta 携带额外信息
  - `{function} next`
    - **`next()`**: 进入下一个守卫
    - **`next(false)`**: 中断当前的导航
    - **`next({ page, data, meta, jumpMethodName, encode })`**: 跳转到一个不同的页面。当前导航被中断
    - **`next(error)`**: 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调

- **this**：当前页面的实例

- **返回值**：promise，无论 rejected 还是 fulfilled ，其值的处理逻辑相同

  - **`true`**: 进入下一个守卫
  - **`false`**: 中断当前的导航
  - **`{ page, data, meta, jumpMethodName, encode }`**: 跳转到一个不同的页面。当前导航被中断
  - **`error`**: 返回一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调

  > `注意`：next 和 返回值 二选一，不然跳转会卡主
  >
  > **跳转成功以后，route 对象会更新**

- **用法**

  在页面里定义（当跳转页面为当前页面时，触发此导航）

- **示例**

  ```js
  <script>
    wepy.page({
      beforeRouteUpdate(to, from, next) {
        next()
      }
      // 或者
      async beforeRouteLeave(to, from) {
        return true
      }
    })
  </script>
  ```

### 完整页面导航解析流程

#### 不同页面

1. 从一个页面跳转到另一页面
2. 在离开页面里调用 `beforeRouteLeave`
3. 调用全局的 `beforeEach` 
4. 调用路由配置里的 `beforeEnter`
5. 调用即将进入页面的 `beforeRouteEnter`
6. 调用全局的 `beforeResolve` 
7. 页面进入后
8. 调用全局的 `afterEach` 
9. 调用 `beforeRouteEnter` 中传给 `next` 的回调函数（如果定义了）
10. 如果某个守卫改变了页面跳转，则中断以上流程并且重新开始导航解析

#### 同一个页面

1. 从一个页面跳转到另一页面
2. 离开页和进入页为同一个页面
3. 调用当前页的`beforeRouteUpdate`

## 实战建议

具体使用场景可以根据业务自行处理，下面给出几个典型场景供开发者参考

### 复杂情况下的页面数据交互

- 将获取数据抽象成一个服务

  ```js
  // 代金券选择组件
  wepy.component({
    /** 页面/组件名 */
    name: 'BookingConfirmRowSelectTicket',
    methods: {
      async goSelectTicket() {
        this.ticketId = await this._getTicketId() || ''
      }
    },
    _getTicketId() {
      return new Promise((resolve, reject) => {
        this.$router.navigateTo({
          page: 'UseTicket',
          data: {
            ...params,
            resolve,
            reject
          }
        })
      })
    }
  })
  
  // 选择代金券页面
  wepy.page({
    /** 页面/组件名 */
    name: 'UseTicket',
    methods: {
      onSelectTicket({ ticketId }) {
        this.ticketId = ticketId
        this.$router.navigateBack({ delta: 1 })
      }
    },
    onUnload() {
      this.$route.data.resolve(this.ticketId)
    }
  })
  ```

### 非正常流程页面跳转

- 要求符合条件 A 的用户无论点击哪个按钮都要跳到某个指定页面，这时可以使用守卫 `beforeRouteLeave`处理

  ```js
  <script>
    wepy.page({
      beforeRouteLeave(to, from, next) {
        // 如果不加 to.page !== 'xxxPage'，会进入死循环
      	if (this.condition && to.page !== 'xxxPage') {
      	  next({ page: 'xxxPage', jumpMethodName: 'navigateTo' })
      	} else {
      	  next()
      	}
      }
    })
  </script>
  ```

### 限制进入某个页面

- 进入某个页面需要符合某个条件，这时可以使用守卫 `beforeEnter`处理

  ```js
  routerConfig = {
    xxxPage: {
      path: '/xxxPath',
      beforeEnter: (to, from, next) => {
        if (condition) {
          next()
        } else {
          this.$showToast('需要 xxx 条件方可进入哦')
          next(false)
        }
      }
    }
  }
  ```

### 预加载数据

- 提升页面加载速度，减少用户等待时间，这时可以使用守卫 `beforeRouteEnter`处理

  ```js
  <script>
    wepy.page({
      _pData: null,
      beforeRouteEnter(to, from, next) {
        const pData = api.getXXX()
        next(vm => {
          vm._pData = pData
        })
      },
      onLoad() {
        this._fetchData()
      },
      async _fetchData() {
        try {
          if (this._pData) {
            this._renderingData = await this._pData
          } else {
            this._renderingData = await api.getXXX()
          }
        } catch(e) {
          // 错误处理
        } finally {
          this._pData = null
        }
      }
    })
  </script>
  ```

### 页面内不同标签页之间跳转

- 某个页面由不同业务组成，分成不同标签页，在定义路由时可根据标签页定义不同的逻辑页面（映射到同一个物理页面），页面内跳转时，可以使用`beforeRouteUpdate`守卫处理。（可以参考启动页、课表页）

  ```js
  routerConfig = {
    xxxPage: '/xxxPath',
    APage: { page: 'xxxPage', data: { type: 'A' } },
    BPage: { page: 'xxxPage', data: { type: 'B' } }
  }
  
  <script>
    wepy.page({
      data: {
        type: 'A'
      },
      beforeRouteUpdate(to, from, next) {
        if (to.data.type === 'B') {
          // 处理 B 业务
        }
  
        next()
      }
    })
  </script>
  ```

## 注意事项

- 路由管理器基于微信提供的 api，但用户某个路由跳转行为不是基于 api，因此无法处理，开发时需要注意。例如：手势滑动返回 或者 点击 tab 或者从外部跳转进入小程序
- 对于首次分包加载， `beforeRouteEnter`守卫无法终止路由跳转，因为导航守卫是在运行时处理的，首次分包加载前无跳转页面代码，因此拿不到此守卫

## 路由管理器对象

路由管理器对象[参考](/router.md)
