## 路由配置

routeMap 是一个对象（key / value），所有页面的路由都在此配置。其中 key 为页面名（page），value 为路由配置信息（config），例如：

```js
homePage = 'SplashScreen'
tabPages = ['CourseList']
routeMap = {
  SplashScreen: '/pages/SplashScreen',
  CourseList: '/pages/tabBar/courseList/CourseList',
  ClassList: { page: 'CourseList', data: { courseType: 'class' } },
  BookList: { path: '/pages/BookList', data: { info: { name: 'roma' } }, meta: 2},
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
