# HTTP 接口规范

为了统一团队前后端接口标准，减少差异，制定 HTTP 接口规范，欢迎大家补充。

## 请求

- POST 请求中的参数，必须在 request body 中
- GET 请求中的参数，必须在 query 中

## 响应

**需要对入参进行校验，并友好提示**

### 统一结构

statusCode 可以都返回 200，或者返回按照 HTTP 规范返回，这个随意

```ts
type ResponseSuccessBody = {
  status: 'ok'
  message?: string
  data: any
}

type ResponseFailBody = {
  status: 'fail' | 'no-auth' | 'need-login'
  errors?: {
    [key: string]: string
  },
  message?: string
}
```

举例：

```json5
{
  status: 'ok',
  message: '登录成功'
}

{
  status: 'ok',
  data: {
    name: 'tom',
    id: 1
  }
}

{
  status: 'fail',
  errors: {
    name: '必填'
  }
}

{
  status: 'fail',
  message: 'name 字段必填'
}
```


### 业务场景

#### 业务 code 配置

业务中涉及到的配置，前端通过统一的 lookup code 进行查询获取，而不是前端编码在代码中。

如下业务配置，不应该前端单独进行维护：
```ts
enum COMPANY {
  baidu = 0
  ali = 1
  tencent = 2
}

enum BUSSINESS {
  电商 = 0
  直播 = 1
  社交 = 2
}
```

应该通过接口获取

```
GET /api/lookup?code=["COMPANY","BUSSINESS"]
```

响应：
```json5
{
  status: 'ok',
  data: {
    COMPANY: {
      baidu: 0,
      ali: 1,
      tercent: 2
    },
    BUSSINESS: {
      电商: 0,
      直播: 1,
      社交: 2
    }
  }
}
```



#### 分页

```json5
{
  status: 'ok',
  data: {
    page: {
      // 当前页码
      number: 1,
      // 全量数据数量
      total: 100,
      // 单页数据数量
      size: 10,
      // 是否后续还有数据，用于滚动异步加载
      hasMore: true
    },
    list: [...]
  }
}
```


## 其他通用

### 请求头

关于请求头，响应头中业务自定义的扩展，必须以 `x-` 开头，如 `x-user-id`

### long / bigInt 类型

由于 JS 存储数字精度问题，遇到 long / bigInt 类型 数字类型，需要后台输出转换为字符串类型，前端请求入参也应该传递为字符串。  
如 `213131232131232132` 需要传递 `'213131232131232132'`
