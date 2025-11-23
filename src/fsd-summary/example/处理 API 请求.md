# 处理 API 请求

## 共享 API 请求

将通用API请求放在 `shared/api` 中

- 📂 shared
  - 📂 api
    - 📄 client.ts
    - 📄 index.ts
    - 📂 endpoints
      - 📄 login.ts



`client.ts` 是 HTTP 常用请求方法封装，以 axios 为例

```ts
// Example using axios
import axios from 'axios';

export const client = axios.create({
  baseURL: 'https://your-api-domain.com/api/',
  timeout: 5000,
  headers: { 'X-Custom-Header': 'my-custom-value' }
});
```

在 `shared/api/endpoints` 封装具体业务功能请求

```ts
import { client } from '../client';

export interface LoginCredentials {
  email: string;
  password: string;
}

export function login(credentials: LoginCredentials) {
  return client.post('/login', credentials);
}
```

## 特定 Slice 的 API 请求

如果一个请求方法只会在特定 分片(Slice) 中出现，并且不会被重复使用，请将其放在当前切片的 api 中，如下：

- 📂 pages
  - 📂 login
    - 📄 index.ts
    - 📂 api
      - 📄 login.ts
    - 📂 ui
      - 📄 LoginPage.tsx

## 其他

如果你的后端有遵循 OpenAPI 规范，也可以使用 [orval](https://orval.dev/) 或 [openapi-typescript](https://openapi-ts.dev/) 这样的工具生成 API 请求类型和函数。最后将代码放在 例如  `shared/api/openapi` 中这样的文件夹中