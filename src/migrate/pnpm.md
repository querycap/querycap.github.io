---
title: 迁移到pnpm
order: 2
toc: menu
nav:
  title: 迁移指南
  path: /migrate
  order: 3
---

代码仓库（的依赖）在膨胀，yarn 的机制已经使得部分项目无法在 ci 环境愉快的安装依赖

因此，考虑迁移 [https://pnpm.js.org/](https://pnpm.js.org/)

##

# How to?

首先安装命令行

```shell
npm install -g pnpm
```

在项目目录中清理干净 yarn 相关的文件

```shell
rm -rf .yarn
rm .yarnrc.yml
rm yarn.lock
```

升级依赖

```shell
pnpm add @querycap-dev/devkit@0.9.0
```

批量替换 `package.json`  中 `yarn`  为 `npm`

尝试运行项目 `npx devkit dev xxx`

和 `yarn` 不同， `pnpm` 没有选择改变 `npm`  太多，而是通过软连接的方式挂载全局缓存的 pkg 到本地。
也没有拍平安装，因此，当前项目会有报错提示 xxx 找不到， `pnpm add xxx`  添加即可。

实在要想和 yarn 的行为一致，也可配置 [https://pnpm.js.org/npmrc#shamefully-hoist](https://pnpm.js.org/npmrc#shamefully-hoist)，shame.. [Doge]

# Troubleshootings

## 重复依赖，影响单例包 (React 等)

在日常项目中可以通过配置 [https://pnpm.js.org/package_json#pnpmoverrides](https://pnpm.js.org/package_json#pnpmoverrides) 解决 （和 yarn 和 npm 的 dedupe 不一样，需要明确声明，大部分情况下只要版本兼容，不会有这个情况）

在 monorepo 或使用 workspace 时，会存在多个 node_modules 的情况，依然会有重复依赖

```
packages/
   local-a/
      node_modules/
        a/
node_modules/
   a/
```

可配置 Webpack 的 `resolve.modules`  解决

```typescript
 c.resolve.modules: [
   process.cwd(),
   // root node_modules first
   path.join(process.cwd(), "node_modules"),
   // then related node_modules
   "node_modules"
 ]
```

Node 环境需要自定 module resolver

## Workspace

详见 [https://pnpm.js.org/workspaces](https://pnpm.js.org/workspaces)
