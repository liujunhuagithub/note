# 特点

- 支持 Promise API
- 拦截请求和响应，**自动转换 JSON 数据**
- 转换请求数据和响应数据，支持取消请求
- 客户端支持防御 XSRF

# 创建方式

#### 默认值创建

axios.defaults.XXX=

#### 新实例

```js
	const service = axios.create({
        baseURL: '/', // api的base_url      //传入config对象
        timeout: 12000 // request timeout
    })
```

# config选项

```js
{
    // url 是用于请求的服务器 URL
    url: '/user'
    method: 'get',    // default
    // 前缀，与url自动拼接
    baseURL: 'https://some-domain.com/api/',
    timeout: 1000,
        
    //发送前修改请求数据，只能用与 PUT、POST 、 PATCH
    // 后面数组中的函数必须返回一个字符串，或 ArrayBuffer，或 Stream
    transformRequest: [function (data, headers) {
        // 对 data 进行任意转换处理
        return data;
    }],
    // 响应后修改数据
    transformResponse: [function (data) {
        // 对 data 进行任意转换处理
        return data;
    }],

    // headers 是即将被发送的自定义请求头
    headers: {'X-Requested-With': 'XMLHttpRequest'},
    
    // Query参数
    // 必须是无格式对象 (plain object) 或 URLSearchParams 对象
    params: {
        ID: 12345
    },
     // data 是作为请求主体被发送的数据，自动json.stringfy，只适用于PUT、POST 、PATCH
    // string，plain object、URLSearchParams、FormData，File，Blob
    data: {
        firstName: 'Fred'
    },


    // withCredentials 表示跨域请求时是否需要使用凭证
    withCredentials: false,    // default
    // 这将设置Authorization 头，覆写自定义的Authorization
    auth: {
        username: 'janedoe',
        password: 's00pers3cret'
    },

    // 服务器响应的数据类型，可以是 blob、document、json、text、stream
    responseType: 'json',    // default
    // 响应的编码
    responseEncoding: 'utf-8',

    // xsrfCookieName 是用作 xsrf token 值的 cookie 名称
    xsrfCookieName: 'XSRF-TOKEN',    // default
    // xsrfHeaderName 是 xsrf token 值的 http 头名称
    xsrfHeaderName: 'X-XSRF-TOKEN',    // default
       
    // onUploadProgress 允许为上传处理进度事件
    onUploadProgress: function (progressEvent) {
        // ... ...
    },
    // onDownloadProgress 允许为下载处理进度事件
    onDownloadProgress: function (progressEvent) {
        // ... ...
    },
    // maxContentLength 定义允许的响应内容的最大尺寸
    maxContentLength: 2000,
}
```

## 拦截器

```js
axios.interceptors.request.use(
    function (config) {
        // 在发送请求之前做些什么
        return config;
    },
    function (error) {
        // 对请求错误做些什么
        return Promise.reject(error);
    }
);

// 添加响应拦截器
axios.interceptors.response.use(
    function (response) {
        // 对响应数据做点什么
        return response;
    },
    function (error) {
        // 对响应错误做点什么
        return Promise.reject(error);
    }
);
```

# 请求发送

- axios.get(url [, config])
- axios.delete(url [, config])
-  axios.post(url [, data[, config]])
- axios.put(url [, data[, config]])
- axios.patch(url [, data[, config]])

## 上传文件

需要用formData，设置请求头

```js
var configs = {
		headers:{'Content-Type':'multipart/form-data'}
	};
let forms=new FormData(formElement)//从已有表单创建
forms.append('k', 'v')    //设置表单字段

forms.append('file', file, file.name)    //设置文件

axios.post('url',forms,configs).then(res=>{})
```

### 上传表单

```js
const params = new URLSearchParams();
params.append('param1', 'value1');
params.append('param2', 'value2');
axios.post('/foo', params);
```

### params 和 data

params：url-query，一般用于get请求。
data：请求体（body）中，自动json序列化。 一般用于post请求，get/delete无效

# 响应格式

```js
{
    // data 由服务器提供的响应
    data: {},
    // status 来自服务器响应的 HTTP 状态码
    status: 200,
    // statusText 来自服务器响应的 HTTP 状态信息
    statusText: 'OK',
    // headers 服务器响应的头
    headers: {},

    // config 是为请求提供的配置信息
    config: {},
    // request 是生成当前响应的请求，XMLHttpRequest 实例
    request: {}
}
```

# 并行请求

- axios.all(  `[  数组  ]`  )：静态方法，==全部完成==才传递
- axios.spread( `callback`(    对应all的所有响应   )=>   )：静态方法，

```js
axios.all(axiosList).then(axios.spread((res1,res2) => {
console.log(res1,res2) // 分别是两个请求的返回值，
})
```

