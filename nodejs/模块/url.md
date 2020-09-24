# url

## 1. 引入

```js
var url =require('url');
```

## 2. parse方法

	- 案例

```js
path = 'https://space.bilibili.com/411553390/favlist?fid=755609890&ftype=create'
```

- url.parse(path)

```js
Url {
  protocol: 'https:',
  slashes: true,
  auth: null,
  host: 'space.bilibili.com',
  port: null,
  hostname: 'space.bilibili.com',
  hash: null,
  search: '?fid=755609890&ftype=create',
  query: 'fid=755609890&ftype=create',
  pathname: '/411553390/favlist',
  path: '/411553390/favlist?fid=755609890&ftype=create',
  href: 'https://space.bilibili.com/411553390/favlist?fid=755609890&ftype=create'
```

- url.parse(path,true)

  ```js
  Url {
    protocol: 'https:',
    slashes: true,
    auth: null,
    host: 'space.bilibili.com',
    port: null,
    hostname: 'space.bilibili.com',
    hash: null,
    search: '?fid=755609890&ftype=create',
    query: 'fid=755609890&ftype=create',
    pathname: '/411553390/favlist',
    path: '/411553390/favlist?fid=755609890&ftype=create',
    href: 'https://space.bilibili.com/411553390/favlist?fid=755609890&ftype=create'
  ```

  

