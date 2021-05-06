---
title: 迁移到webpack5
order: 1
toc: menu
nav:
  title: 迁移指南
  path: /migrate
---

改动还好，优化明显 [https://github.com/webpack/changelog-v5/blob/master/MIGRATION%20GUIDE.md](https://github.com/webpack/changelog-v5/blob/master/MIGRATION%2520GUIDE.md)

@querycap-dev/webpack-\* 相关升级的时候，注意 node 的废弃，并用 alias 和 ProviderPlugin 替代。

## package.json

```json
{
  // remove "@morlay/webpack-browser-sync": "",
  // remove "@types/webpack": "",
  "@querycap-dev/webpack-browser-sync": "^0.1.0",
  "@querycap-dev/webpack-preset": "^0.7.0",
  "@querycap-dev/webpack-preset-assets": "^0.7.0",
  "@querycap-dev/webpack-preset-html": "^0.8.0",
  "@querycap-dev/webpack-preset-ts": "^0.8.0",
  "webpack": "5.0.0"
}
```

## node-lib 和 全局变量的替代

这个改动是让开发者明确的清楚 node 环境和浏览器环境的不同

根据报错信息，添加前端对应版本

```typescript
// node lib 需要定义 alias
c.resolve!.alias = {
  path: "path-browserify", // yarn add -D path-browserify
  stream: "stream-browserify", // yarn add -D stream-browserify
  ...
};

// 对于 process, Buffer 全局声明，需要这样替代，已在 @querycap-dev/webpack-preset-ts 中处理
c.plugins.push(
   new ProvidePlugin({
      Buffer: ["buffer", "Buffer"],
      process: "process",
   }),
)
```

### Module Federation（伪 Snowpack）

webpack 5 的新特性，可在此基础探索微前端，以实现跨应用的模块共享。
在开发模式下，通过拆分三方依赖，项目内公用模块，应用模块等，减少需要 pack 的量，以加速 re-pack。

另，为了让这一机制更好生效，严格：

1. `package.json`  正则需要在浏览器中使用的依赖, 务必移到 `dependencies`
1. 注册为 `shared`  的目录，需要有 `index.ts` 和 `package.json`，调用方按模块引入。
   1. 当然也可以精确到文件，只是需要再每次文件创建，改名或删除时，重启 webpack

```typescript
// @ts-ignore
import pkg from './package.json';
// @ts-ignore
import ModuleFederationPlugin from 'webpack/lib/container/ModuleFederationPlugin';

(c, state) => {
  if (state.flags.production) {
    // todo 需要验证生产环境下，对 tree shaking 等的影响
    // http2 模式 only
    return;
  }

  const deps = Object.keys(pkg.dependencies);
  const canaries = globby.sync(
    [
      // canary pkg
      '@querycap-canary/*',
      // clients
      'src-clients/*',
      // 应用模块
      'src-app/*/*',
    ],
    {
      cwd: state.cwd,
      onlyDirectories: true,
    },
  );

  c.plugins?.push(
    // eslint-disable-next-line @typescript-eslint/no-unsafe-call
    new ModuleFederationPlugin({
      shared: [...deps, ...canaries],
    }),
  );
};
// withTsPreset 之前插入
```

## Tips

升级后记得删除 cache `rm -rf node_modules/.cache`
