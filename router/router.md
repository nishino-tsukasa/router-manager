# router

## 导入方式

`import router from 'router'`

## 属性

### route

- 类型: Route

  当前页面对应的路由对象

- route.id

  - 类型: string

    当前页面的 id（唯一）

- route.url

  - 类型: string

    当前页面 url，带查询参数（只带数字类型和字符串类型的参数数据）

- route.page

  - 类型: string

    当前页面名

- route.data

  - 类型: Object

    当前页面的完整查询参数数据

- route.meta

  - 类型: string

    当前页面的非查询参数的其它数据

- route.routes

  - 类型: [Route, Route, ..., Route(当前路由)]

    当前页面的路由栈

- route.lastRoute

  - 类型: Route

    路由栈中当前路由的上一个路由，没有则为 null

- route.referrerRoute

  - 类型: Route

    前向路由，访问轨迹的当前页面的上一个页面的路由，没有则为 null

    例如：A -> B -> C，再返回到 B，当前路由为 B，lastRoute 为 A，而 referrerRoute 为 C

- route.hasVisited

  - 类型: null

    页面内是否通过 this.$route 已经访问过当前路由对象

### homePage

- **类型**: string

  首页的页面名

### tabPageMap

- **类型**: Object

  tab 页配置表，因为 tab 页跳转的 api 不一样

  ```js
  const switchRouteMap = {
    BoxList: true,
    CourseList: true,
    MyBooking: true,
    My: true
  }
  
  ```

### routeMap

- **类型**: Object

  [页面路由配置表](/路由管理器.mdrouteMap）)

## 方法/导航

页面导航在 wx api 的基础上封装

### navigateTo

保留当前页面，跳转到应用内的某个页面

- **形式**：navigateTo(object, encode)

- **参数**：

  - `{Object} object`
    - `{string} page`页面名
    - `{Object} data`页面参数
    - `{function} success`接口调用成功的回调函数
    - `{function} fail`接口调用失败的回调函数
    - `{function} complete`接口调用结束的回调函数（调用成功、失败都会执行）
  - `{bool} encode`是否对参数进行编码

- **返回值** promise，根据调用成功/失败设置状态

- **示例**

  ```js
  import router from 'router/router'
  router.navigateTo({ page: 'BadgeDetail', data: { badgeId, sk } })
  ```

### redirectTo

关闭当前页面，跳转到应用内的某个页面

- **形式**：redirectTo(object, encode)

- **参数**：

  - `{Object} object`
    - `{string} page`页面名
    - `{Object} data`页面参数
    - `{function} success`接口调用成功的回调函数
    - `{function} fail`接口调用失败的回调函数
    - `{function} complete`接口调用结束的回调函数（调用成功、失败都会执行）
  - `{bool} encode`是否对参数进行编码

- **返回值** promise，根据调用成功/失败设置状态

- **示例**

  ```js
  import router from 'router'
  router.redirectTo({ page: 'BadgeDetail', data: { badgeId, sk } })
  ```

### switchTab

跳转到 tabBar 页面

- **形式**：switchTab(object, encode)

- **参数**：

  - `{Object} object`
    - `{string} page`页面名
    - `{Object} data`页面参数
    - `{function} success`接口调用成功的回调函数
    - `{function} fail`接口调用失败的回调函数
    - `{function} complete`接口调用结束的回调函数（调用成功、失败都会执行）
  - `{bool} encode`是否对参数进行编码

- **返回值** promise，根据调用成功/失败设置状态

- **示例**

  ```js
  import router from 'router'
  router.switchTab({ page: 'ClassList', data: { scheduleId, sk } })
  ```

### navigateBack

关闭当前页面，返回上一页面或多级页面

- **形式**：navigateBack(object, encode)

- **参数**：

  - `{Object} object`
    - `{number} delta`返回的页面数，如果 delta 大于现有页面数，则返回到首页。默认：1
    - `{function} success`接口调用成功的回调函数
    - `{function} fail`接口调用失败的回调函数
    - `{function} complete`接口调用结束的回调函数（调用成功、失败都会执行）

- **返回值** promise，根据调用成功/失败设置状态

- **示例**

  ```js
  import router from 'router'
  router.navigateBack({ delta: 1 })
  ```

### reLaunch

关闭所有页面，打开到应用内的某个页面

- **形式**：reLaunch(object, encode)

- **参数**：

  - `{Object} object`
    - `{string} page`页面名
    - `{Object} data`页面参数
    - `{function} success`接口调用成功的回调函数
    - `{function} fail`接口调用失败的回调函数
    - `{function} complete`接口调用结束的回调函数（调用成功、失败都会执行）
  - `{bool} encode`是否对参数进行编码

- **返回值** promise，根据调用成功/失败设置状态

- **示例**

  ```js
  import router from 'router'
  router.reLaunch({ page: 'ClassList', data: { scheduleId, sk } })
  ```

>  page 会根据 routeMap 找到真实页面路径（path）
>
> data 会转换成查询参数（'key=value&key2=value2'），如果 encode 为 true，则会使用 encodeURIComponent 进行编码，由于转换后是字符串，因此 data 里如果有非 string 和 number 将会被过滤掉，可以从 route.data 里拿到完整的 data

## 方法/全局守卫

### beforeEach

[参考](#beforeEach)

### beforeResolve

[参考](#beforeResolve)

### afterEach

[参考](#afterEach)

## 方法/工具

### getRealPageInfo

- **形式**：getRealPageInfo({ page, data })

- **参数**：

  - `{string} page`页面名
  - `{Object} data`页面参数

- **返回值**：返回物理页面相关信息 { pagePath, pageData, pageMeta }

  - `{string} pagePath`页面完整路径（不带参数）
  - `{Object} pageData`页面参数
  - `{Object} pageMeta`页面不属于 data 的额外信息

- **示例**

  ```js
  import router from 'router'
  const { pagePath, pageData, pageMeta } = router.getRealPageInfo({ page, data })
  ```
