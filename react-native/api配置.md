## 安装 axios、moment、chalk、qs 库

```ts
npm install axios moment chalk qs
```

## 在根目录文件下新建 config 文件夹

#### 存放 API 的基础 URL、超时时间等

```ts
/*  config.json */
{
  "apiUrl":"",
  "timeout":12000
}

/* env.ts */
{
  import {IEnv} from './env.type';
  export const Env: IEnv = require('./config.json');
}
```

## 根目录文件新建 service 文件夹

```tsx
/*  api.ts  */

import Qs from "qs";
import axios, { AxiosInstance, AxiosRequestConfig } from "axios";
import { Env } from "../../config/env";
import { requestLogger, responseLogger } from "../axios-logger";
import moment from "moment-timezone";

// 取消令牌源，终止响应
export let cancelTokenSource = axios.CancelToken.source();

// 创建Axios实例'api'
export const api: AxiosInstance = axios.create({
  baseURL: Env.apiUrl,
  headers: {
    Accept: "application/json",
  },
  paramsSerializer: (params) => Qs.stringify(params, { arrayFormat: "repeat" }), // 用于序列化请求参数
  timeout: 120000,
  cancelToken: cancelTokenSource.token, // 意味着每个请求都将使用这个取消令牌源
});

api.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    // 获取当前时区的名称，服务器有可能需要知道请求来自哪个时区，得知客户端的时区可以进行时间转换和处理，比如应用需要展示时间相关的事件
    config.headers.Timezone = moment.tz.guess();

    cancelTokenSource = axios.CancelToken.source();
    config.cancelToken = cancelTokenSource.token;

    return config;
  },
  function (error) {
    return Promise.reject(error);
  }
);

// 开发环境中启用请求和响应的日志记录
// 生产环境中关闭，避免造成不必要的性能和安全问题
if (process.env.NODE_ENV === "development") {
  api.interceptors.request.use(requestLogger);
  api.interceptors.response.use(responseLogger);
}

api.interceptors.response.use(
  (response) => {
    return response.data;
  },
  (error) => {
    // ...处理不同类型的错误...
  }
);

export const setApiToken = (token: string) => {
  api.defaults.headers.common.Authorization = `${token}`;
};

export const setUserAgent = (userAgent: string) => {
  api.defaults.headers.common["User-Agent"] = userAgent;
};
```

#### 其中，记录请求和相应的日志记录配置：

###### 在 service 下新建 axios-logger，新建以下文件

###### log-builder.ts

```tsx
import chalk from "chalk";
import moment from "moment";

class LogBuilder {
  private printQueue: Array<string>;

  constructor() {
    this.printQueue = [];
  }

  // 日志类型：‘API Request’或‘API Response’
  makeLogTypeWithPrefix(logType: string) {
    const prefix = `[${logType}]`;
    this.printQueue.push(chalk.green(prefix));

    return this;
  }

  // 日期时间格式化，使用‘moment’库将日期转换为ISO格式
  makeDateFormat(date: Date) {
    const dateFormat = moment(date).toISOString();
    this.printQueue.push(dateFormat);

    return this;
  }

  // 添加请求的url
  makeUrl(url?: string) {
    url && this.printQueue.push(url);

    return this;
  }

  // 添加请求的方法‘get’或‘post’等，并将方法名称标记为黄色
  makeMethod(method?: string) {
    method && this.printQueue.push(chalk.yellow(method.toUpperCase()));

    return this;
  }

  // 添加请求的数据，并将数据对象转换为JSON字符串
  makeData(data?: object) {
    data && this.printQueue.push(JSON.stringify(data));

    return this;
  }

  // 添加响应的数据，将数据对象转换为JSON字符串
  makeStatus(status?: number, statusText?: string) {
    if (status && statusText) {
      this.printQueue.push(`${status}:${statusText}`);
    } else if (status) {
      this.printQueue.push(`${status}`);
    } else if (statusText) {
      this.printQueue.push(statusText);
    }

    return this;
  }

  // 添加执行时间，用于记录请求的处理时间
  makeExecuteTime(time: number) {
    const timeFormat = `+${time}ms`;
    this.printQueue.push(chalk.yellow(timeFormat));

    return this;
  }

  // 将所添加到队列的日志部分组合成最终的日志字符串
  build() {
    return this.printQueue.join(" ");
  }
}

export default LogBuilder;
```

###### request.ts

```tsx
/* request.ts */

import { AxiosRequestConfig } from "axios";
import LogBuilder from "./log-builder";

const requestLogger = (request: AxiosRequestConfig) => {
  const { url, method, data } = request;

  const logBuilder = new LogBuilder();
  const log = logBuilder
    .makeLogTypeWithPrefix("API Request")
    .makeDateFormat(new Date())
    .makeMethod(method)
    .makeUrl(url)
    .makeData(data)
    .build();

  // 打印请求日志记录
  console.log(log);

  return request;
};

export default requestLogger;
```

###### response.ts

```tsx
import { AxiosResponse } from "axios";
import LogBuilder from "./log-builder";

const responseLogger = (response: AxiosResponse) => {
  const {
    config: { url, method, params },
    status,
    statusText,
  } = response;

  // @ts-ignore
  const time = new Date().getTime() - response.config.meta.requestStartedAt;

  const logBuilder = new LogBuilder();
  const log = logBuilder
    .makeLogTypeWithPrefix("API Response")
    .makeDateFormat(new Date())
    .makeMethod(method)
    .makeUrl(url)
    .makeData(params)
    .makeStatus(status, statusText)
    .makeExecuteTime(time)
    .build();

  console.log(log, response);

  return response;
};

export default responseLogger;
```
