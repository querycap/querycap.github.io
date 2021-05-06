---
title: 介绍
order: 1
toc: menu
nav:
  title: 指南
  path: /guide
  order: 1
---

[![Build Status](https://img.shields.io/travis/querycap/devkit.svg?style=flat-square)](https://travis-ci.org/querycap/devkit)
[![codecov](https://codecov.io/gh/querycap/devkit/branch/master/graph/badge.svg)](https://codecov.io/gh/querycap/devkit)

## 环境准备

- node：15.x （建议安装 nvm）
- yarn：2.x （升级指南）
- pnpm: 6 (升级指南)
- npm 源：https://registry.npmjs.org 或 https://registry.yarnpkg.com, 禁止使用淘宝源
- TypeScript Only
- No IE

## 初始化仓库

推荐直接 fork Gitlab idp/webapp-idp 这个仓库。

## 项目结构

- 支持多个应用并行开发，便于公用模块的复用
- 通过 tag 触发 CI/CD 流程，`feat/<APP_NAME>[--<FEATURE>][.<ENV>]`

```
src-app/
    app-one/
        index.ts    # WebApp 入口文件
        config.ts   # 项目配置，详见「创建项目」
        index.html  # (可选) index 模板
        icon.png    # 和 config 中的 APP_MANIFEST 共同触发构建为 PWA
    app-two/
        index.tsx
        config.ts
helmx.project.yml
webpack.config.ts
package.json
```

## Setup (fork 仓库跳过此步骤)

```shell
pnpm add -W @querycap-dev/devkit
pnpm run devkit init
```

### 配置 webpack.config.ts

```ts
import { withPresets } from '@querycap/webpack-preset';
import { withAssetsPreset } from '@querycap/webpack-preset-assets';
import { withHTMLPreset } from '@querycap/webpack-preset-html';
import { withTsPreset } from '@querycap/webpack-preset-ts';

export = withPresets(
  (c, state) => {
    console.log(state);
    // modify webpack configuration by yourself
  },
  withTsPreset(),
  withAssetsPreset(),
  withHTMLPreset(),
);
```

### 配置 package.json

```json
{
  "devkit": {
    "build": "webpack --config webpack.config.ts",
    "dev": "webpack-browser-sync --config webpack.config.ts --historyApiFallback --index=../index.html"
  }
}
```

### 创建项目

src-app 下每个文件夹为一个项目

#### 配置 config.ts

```ts
import { confLoader } from '@querycap/config';

// 部署环境列表，会处理为全小写
// 默认使用 第一个
export enum ENVS {
  STAGING = 'STAGING',
  TEST = 'TEST',
  DEMO = 'DEMO',
  ONLINE = 'ONLINE',
  LOCAL = 'LOCAL',
}

export const APP_MANIFEST = {
  name: '测试',
  background_color: '#19C7B1',
  crossorigin: 'use-credentials',
};

export const APP_CONFIG = {
  SRV_TEST: (env: string, feature: string) => {
    if (env === 'local') {
      return `//127.0.0.1:80`;
    }

    if (feature === 'demo') {
      return `//api.demo.com`;
    }

    return `//api.com`;
  },
};

// conf() 将返回从 meta 中读取的配置信息，也方便在容器中注入
// 另外，当应用以插件的形式使用的时候，也方便对应页面配置
// <meta name="devkit:app" content="appName=demo,env=demo,version=__PROJECT_VERSION__">
// <meta name="devkit:config" content="SRV_TEST=//demo.querycap.com">
export const conf = confLoader<keyof typeof APP_CONFIG>();
```

### 通过 app 和 env 进行项目切换

```
# 开发

pnpm run devkit dev <app[--feature]> [env]
or
pnpx devkit dev <app[--feature]> [env]

# 打包
pnpm run devkit build <app[--feature]> [env]

or

pnpx devkit build <app[--feature]> [env]

# 压缩打包
pnpm run devkit build --prod <app[--feature]> [env]

or

pnpx devkit build --prod <app[--feature]> [env]

# 发布
pnpm run devkit release <app[--feature]> [env]

or
pnpx devkit release <app[--feature]> [env]

# 生成clients文件

pnpm run c <app>

```

## 规范

### 目录结构规范

```bash
├── @querycap-canary // 公共组件
├── __mocks__ // 暂未使用
├── __types__ //ts类型
├── cmd // start后生成文件，用于CI
├── node_modules //依赖
├── public // build目录
├── src-app // 子项目文件夹
├── src-clients // API接口，自动生成
├── src-modules //业务模块组件
└── tools // 工具

单项目-以smart-power-plant为例

├── CSSPreset.tsx //全局CSS
├── README.md //项目说明
├── assets // 图标资源
├── bootstrap.tsx // 根组件，初始化store、axios、css preset、theme
├── config.ts //项目配置
├── data
├── health
├── index.html //html
├── index.ts // 入口文件
├── optimize
├── overview
├── platform
├── profile
├── routes.tsx //路由入口
└── support

// 其他目录为路由页面，按菜单或者功能模块划分不同文件夹，
//文件夹下routes.tsx文件为当前文件夹的路由组件。
//子菜单或者子模块组件或者页面位于同一文件夹，扁平结构便于查找。
```

### 公共模块

公共模块主要有以下几部分：

- @querycap-ui 或者三方组件库，统一安装于根目录下 node_modules。
- @querycap-canary,位于根目录下，所有项目共用，主要是 UI 组件、地图组件、utils，达到一定程度可抽离到@querycap-ui 中。
- src-modules，位于根目录下，所有项目共用，主要是业务级别的模块，如所有项目公用的 idp 相关页面级别模块，电厂项目的电厂端和环保院端的公共模块。
- src-clients，位于根目录下，所有项目共用，pnpm run c 命令生成的接口类型文件。

### config.ts 项目配置

```ts
import { confLoader } from '@querycap/config';

export const GROUP = 'custom-projects';

export const ENVS = {
  STAGING: 'staging',
  DEMO: 'demo',
  LOCAL: 'local',
  ONLINE: 'online',
};

export const APP_MANIFEST = {
  name: 'smart-power-plant',
  short_name: 'smart-power-plant',
  description: '烟气治理岛智慧运行管控平台',
  crossorigin: 'use-credentials',
};

export const APP_CONFIG = {
  appName: () => {
    return APP_MANIFEST.short_name;
  },
  ENV: (env: string) => {
    return env.toLowerCase();
  },
  TITLE: () => {
    return `${APP_MANIFEST.name}:${APP_MANIFEST.description}`;
  },
  SRV_BFF_SMART_POWER_PLANT: () => {
    return `https://srv-bff-smart-power-plant---custom-projects.hw-dev.rktl.xyz`;
  },
  SRV_IDP: (env: string) => {
    if (env === ENVS.LOCAL) {
      return '//localhost';
    }
    return `https://idp-api.rockontrol.com`;
  },
};

export const conf = confLoader<keyof typeof APP_CONFIG>();
```

GROUP 字段为当前项目所在 GROUP，由后端配置，会影响 CI 执行，会出现在发布后访问的 URL 上。
如果没有该 GROUP，需要找运维创建。

APP_CONFIG 中 SRV 开头的为后端不同服务对应的服务地址，pnpm run c 生成 client 也会读取该配置。

### 三方依赖

本项目主要用到的三方依赖有图表组件：@ant-design/charts。时间相关 utils 使用 date-fns，尽量减少三方依赖使用。

### 自定义 Icon 组件

主要使用@querycap-ui/icons 中的 createIcon 方法来实现自定义 Icon 组件，传入对应 svg 元素，删除掉不正确的属性，如果需要支持自定义 fill 需要将 path 上的 fill 属性删除。

```ts
import { createIcon } from '@querycap-ui/icons';

export const IconSupport1 = createIcon(
  <svg width="18px" height="18px" viewBox="0 0 18 18" version="1.1">
    <g id="电厂端" stroke="none">
      <g
        id="诊断详情"
        transform="translate(-680.000000, -16.000000)"
        fill="#FFFFFF"
      >
        <g id="3" transform="translate(665.000000, 0.000000)">
          <g id="远程支持中心icon" transform="translate(15.000000, 16.000000)">
            <path
              d="M16.3770395,8.60702771 C17.2726696,8.60702771 18,9.33493634 18,10.2300672 L18,10.2300672 L18,13.9187934 C18,14.8139243 17.2726696,15.5408492 16.3770395,15.5408492 L16.3770395,15.5408492 L14.2619745,15.5408492 L14.2619745,15.9854637 L15.1467784,15.9854637 C15.7028564,15.9854637 16.1546083,16.4369638 16.1546083,16.9927318 C16.1546083,17.5484999 15.7028564,18 15.1467784,18 L15.1467784,18 L11.4559949,18 C10.8989327,18 10.4471808,17.5484999 10.4471808,16.9927318 C10.4471808,16.4369638 10.8989327,15.9854637 11.4559949,15.9854637 L11.4559949,15.9854637 L12.2453305,15.9854637 L12.2453305,15.5408492 L10.2257338,15.5408492 C9.32911946,15.5408492 8.60080486,14.8139243 8.60080486,13.9187934 L8.60080486,13.9187934 L8.60080486,10.2300672 C8.60080486,9.33493634 9.32911946,8.60702771 10.2257338,8.60702771 L10.2257338,8.60702771 Z M2.25462578,9.8247008 C2.81070382,9.8247008 3.26343992,10.2762009 3.26343992,10.831969 C3.26343992,12.0241653 3.59511832,12.9586425 4.24961725,13.6078584 C5.37161541,14.7223455 7.15500197,14.7430023 7.17271773,14.7430023 C7.74749574,14.7449697 8.18448449,15.1748292 8.1884492,15.7433849 C8.19038975,16.0119241 8.08803202,16.2666922 7.8980797,16.4585059 C7.70911159,16.649336 7.4561699,16.756555 7.18649665,16.7585223 L7.18649665,16.7585223 L7.13531779,16.7585223 C6.70029745,16.753604 4.45236429,16.6454014 2.83629325,15.0469534 C1.78122129,14.0032898 1.24679585,12.5858353 1.24679585,10.831969 C1.24679585,10.2762009 1.69854775,9.8247008 2.25462578,9.8247008 Z M15.7619089,10.6225477 L10.8408644,10.6225477 C10.7188224,10.6225477 10.6184331,10.722881 10.6184331,10.8448549 L10.6184331,10.8448549 L10.6184331,13.3040057 C10.6184331,13.4259796 10.7188224,13.5263129 10.8408644,13.5263129 L10.8408644,13.5263129 L15.7619089,13.5263129 C15.8839508,13.5263129 15.9833559,13.4259796 15.9833559,13.3040057 L15.9833559,13.3040057 L15.9833559,10.8448549 C15.9833559,10.722881 15.8839508,10.6225477 15.7619089,10.6225477 L15.7619089,10.6225477 Z M7.77525043,0 C8.67088054,0 9.39919514,0.727908629 9.39919514,1.62303951 L9.39919514,1.62303951 L9.39919514,5.31176567 C9.39919514,6.20689655 8.67088054,6.93480518 7.77525043,6.93480518 L7.77525043,6.93480518 L5.66215389,6.93480518 L5.66215389,7.37745232 L6.54498928,7.37745232 C7.10106732,7.37745232 7.55380342,7.82993606 7.55380342,8.38570414 C7.55380342,8.94147221 7.10106732,9.39395595 6.54498928,9.39395595 L6.54498928,9.39395595 L2.85420585,9.39395595 C2.29812782,9.39395595 1.84539172,8.94147221 1.84539172,8.38570414 C1.84539172,7.82993606 2.29812782,7.37745232 2.85420585,7.37745232 L2.85420585,7.37745232 L3.64452561,7.37745232 L3.64452561,6.93480518 L1.62394471,6.93480518 C0.728314597,6.93480518 0,6.20689655 0,5.31176567 L0,5.31176567 L0,1.62303951 C0,0.727908629 0.728314597,0 1.62394471,0 L1.62394471,0 Z M10.8341717,1.20498388 C11.0989239,1.17744139 13.5269673,1.26990546 15.1902804,2.91753648 C16.2453523,3.96021641 16.7798042,5.37767091 16.7798042,7.13153724 C16.7817462,7.39712553 16.6734832,7.65681185 16.4845151,7.84469097 C16.2945628,8.03453741 16.0426053,8.13978906 15.7709636,8.13978906 C15.2158698,8.13978906 14.7641179,7.68730532 14.7641179,7.13153724 C14.7641179,5.94032461 14.4314553,5.00683097 13.7779406,4.35663151 C12.6549582,3.24116072 10.8735401,3.22148751 10.8558243,3.22148751 C10.4946197,3.22148751 10.1580202,3.03065741 9.97889419,2.7237554 C9.79681554,2.4139024 9.79386291,2.0273239 9.97102052,1.7174709 C10.1471939,1.40565058 10.4769039,1.20990218 10.8341717,1.20498388 Z M7.16011985,2.01650363 L2.23907528,2.01650363 C2.11703338,2.01650363 2.01762828,2.11585333 2.01762828,2.2378272 L2.01762828,2.2378272 L2.01762828,4.69697798 C2.01762828,4.81895186 2.11703338,4.91830155 2.23907528,4.91830155 L2.23907528,4.91830155 L7.16011985,4.91830155 C7.28216176,4.91830155 7.38156686,4.81895186 7.38156686,4.69697798 L7.38156686,4.69697798 L7.38156686,2.2378272 C7.38156686,2.11585333 7.28216176,2.01650363 7.16011985,2.01650363 L7.16011985,2.01650363 Z"
              id="形状结合"
            ></path>
          </g>
        </g>
      </g>
    </g>
  </svg>,
);
```

另外还可以编写 test 根据 svg 文件自动生成组件。

```
import { generate, loadFiles } from "@querycap-dev/generate";
import { svg2tsx } from "@querycap-dev/generate-svg2tsx";

describe("generate", () => {
  it("Icon Components", () => {
    const files = loadFiles(["./icons/*.svg"], { cwd: __dirname });

    const codes = svg2tsx(files, {
      iconCreator: "@querycap-ui/icons.createIcon",
    });

    generate("../index.tsx", codes, { cwd: __dirname });
  });
});
```

此方法的使用需要注意 UI 提供的 svg 文件是否支持，需要注意某些高级属性会失效。

### 状态管理

目前的项目除了用户信息、token、权限等全局状态都已有相应的公共实现，部分业务需要使用状态管理的场景需要考虑该场景采用何种状态管理方案。

- 页面不同组件共享状态：context
- 全局状态共享、持久化： @querycap/contexts

```
import { useStoreState$ } from "@querycap/contexts";
```

- 父子组件共享状态：props

### 样式

采用 CSS in JS 方案，使用 emotion 库。

详见文档：https://emotion.sh/docs/introduction

### 主题设置

通过覆盖 ThemeProvider 中的 theme prop 来实现全局主题的更改：

```ts
import { NotificationHub, Confirmations } from '@querycap-canary/dashboard';
import { ThemeProvider } from '@querycap-ui/core/macro';
import { patchBearerToken } from '@querycap/access';
import { createBootstrap } from '@querycap/bootstrap';
import { useConfig } from '@querycap/config';
import { AxiosProvider, baseURLsFromConfig } from '@querycap/request';
import { useStore } from '@reactorx/core';
import { ReactNode } from 'react';
import { conf } from './config';
import { CSSPreset } from './CSSPreset';
import { rootRoutes } from './routes';
import { defaultTheme } from '@querycap-ui/core';

const Setup = ({ children }: { children: ReactNode }) => {
  const store$ = useStore();
  const c = useConfig();

  return (
    <ThemeProvider
      theme={{
        ...defaultTheme,
        state: {
          ...defaultTheme.state,
          color: '#222222',
          borderColor: '#C6CACC',
        },
        colors: {
          ...defaultTheme.colors,
          primary: '#367BF5',
          danger: '#C93124',
          success: '#0A9899',
        },
      }}
    >
      <CSSPreset>
        <Confirmations />
        <NotificationHub />
        <AxiosProvider
          baseURLs={baseURLsFromConfig(c)}
          interceptors={[patchBearerToken(store$)]}
        >
          {children}
        </AxiosProvider>
      </CSSPreset>
    </ThemeProvider>
  );
};

export const bootstrap = createBootstrap(conf())(() => (
  <Setup>{rootRoutes}</Setup>
));
```

部分局部主题修改可以通过 ThemeState 进行设置：

```ts
import { theme } from '@querycap-ui/core/macro';

const Main = () => {
  return (
    <ThemeState
      fontSize={theme.fontSizes.s}
      lineHeight={theme.lineHeights.condensed}
    ></ThemeState>
  );
};
```

主题使用：

```ts
import { theme } from '@querycap-ui/core/macro';

const Link = () => {
  return (
    <a
      css={select()
        .display('inline-block')
        .fontSize(theme.state.fontSize)
        .colorFill(theme.state.color)
        .borderRadius(theme.radii.normal)}
    ></a>
  );
};
```

### 权限管理

需要进行权限控制的组件名称前面加 Ac，如 AcButton，增加 Ac 后，会对组件内使用到 useRequest 和 useTempDataRequest 的 hook 的请求 API 收集，判断是否存在当前用户的权限中，如果没有权限，对应组件不会显示，对应路由的菜单不会显示。
[https://www.npmjs.com/package/@querycap/babel-plugin-access-control-autocomplete](https://www.npmjs.com/package/@querycap/babel-plugin-access-control-autocomplete)
对于组件内部没有使用到 useRequest 等请求 hook 的组件增加权限，需要使用*mustOneOfPermissions 或者 mustAllOfPermissions 进行包裹，并传入对应请求方法。*

```typescript
const ConfigGraphButton = mustOneOfPermissions(saveGraphDevicePosition)(
  ({ graphID }: { graphID: string }) => {
    return (
      <ProjectNavLink to={`/optimize/config/${graphID}`}>
        <Button type="button" small primary css={select().marginY('0.5em')}>
          组态图配置
        </Button>
      </ProjectNavLink>
    );
  },
);
```

### 编码

commit 和 push 阶段会自动运行 eslint 和 tsc 检查代码规范和 TypeScript 类型。

#### 命名

> 简洁并语义明确，禁拼音。

- 文件夹、路由 path：短横线加小写 （如：demos-user）
- 组件、ts 的 type 和 interface：大驼峰 （如：UserList）
- 变量、普通函数：小驼峰 （如：userName，getUserName）
- 常量：下划线加大写 （如：MAX_LENGTH）

#### 代码风格

```ts
  /**
  * 该换行的地方换行，该空格的空格，该缩进的缩进，便于阅读；
  * 编写符合项目下eslint规范的代码；
  * 如下demo
  */
  import { useEffect } from "react";
  import moment from "moment";

  type IUserType = "AA" | "BB";

  const MAX_LENGTH = 10;

  const users = [{
    name: 'AA',
    userID: '12345678'
  }, {
    name: 'BB',
    userID: '23456789'
  }, ];

  const UserList = () => {
    const [visible, setVisible] = useState<boolean>(true);

    const handleClick = () => {
      ...
    }

    return (
      <ul>
        {
          map(users, user => (
            <li
              key={user.userID}
              css={{
                ...
              }}
              onClick={handleClick}
            >
              { user.name }
            </li>
          ))
        }
      </ul>
    );
  }

  const Users = () => (
    <div>
      <UserOverview />
      <UserList />
    </div>
  )

  export default Users;
  // 最后空行
```

TypeScript 必须保证类型正确，禁止随意使用 any。

#### 防御性编程

TypeScript 并不能百分之百保证类型正确，为了减少 JS 报错导致页面挂掉的情况，引入 lodash 来保证代码的健壮性，
举例：使用数组的 map、forEach、reduce 等方法，或者对象的 keys、values 等等情况下，推荐使用 lodash 的方法。

```ts
import { map } from 'lodash';

export const AlertPanel = ({
  data,
  children,
}: {
  data: IAlertInfo[];
  children?: ReactNode;
}): JSX.Element => {
  return (
    <div
      css={select()
        .position('absolute')
        .top(0)
        .right(0)
        .padding('.6em 1em')
        .borderBottomLeftRadius(theme.radii.s)
        .backgroundColor(rgba(colors.black, 0.5))
        .fontSize(theme.fontSizes.xs)
        .zIndex(999)}
    >
      {map(data, item => (
        <div key={item.label}>
          {`${item.label}:`} &nbsp;&nbsp;
          {item.value}
        </div>
      ))}
      {children}
    </div>
  );
};
```

#### 组件化、抽象化

- 根据组件的用途，合理的抽离组件，使组件更容易维护（注意：粒度不是越细越好，过细会使之松散，难以阅读）；
- 抽离重复功能的组件为公共组件，以减少代码冗余，提高代码统一维护性；
- 越是底层的组件越要考虑抽象化，以提高基础组件的通用性，可拓展性（如：UI 组件）；

#### 代码提交

- git commit 符合标准的 commit type 规范 [查看](https://rockontrol.yuque.com/spec/e2e/biodoq)；
- 推荐使用 git rebase 维护 git 提交历史；
