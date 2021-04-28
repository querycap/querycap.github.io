---
title: 开发
order: 2
toc: menu
---

## @querycap-dev

### 介绍

@querycap-dev 下是开发环境下一系列包的集合

## @querycap-dev/devkit

@querycap-dev/devkit 用于启动项目、发布项目到 Gitlab CI 的 npm 包。

配置

```
// package.json
{
  "name": "webapp",
  "workspaces": [
    "@querycap-canary/*"
  ],
  "scripts": {
    "start": "devkit"
  },
  "devkit": {
    "dev": "webpack-browser-sync --config webpack.config.ts --historyApiFallback --index=../index.html",
    "build": "webpack --config webpack.config.ts"
  },
}
```

初始化(以 pnpm 为例)

```bash
pnpx devkit init
```

启动项目

```bash
pnpx devkit dev projectName
```

发布项目

```bash
pnpx devkit release projectName
```

## @querycap-dev/generate-client

@querycap-dev/generate-client 用于根据每个项目下的 config.ts 中的服务配置自动生成请求方法以及 TS 数据类型声明文件。

### 配置

```ts
// src-app/smart-power-plant
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

## @querycap-dev/generate-css2tsx

@querycap-dev/generate-css2tsx 用于将三方库的 css 文件转成 emotion 的 Global 组件。

### 使用

```ts
// styles/__test__/generator.spec.ts
import { generate } from '@querycap-dev/generate';
import { css2tsx } from '@querycap-dev/generate-css2tsx';
import path from 'path';

test('generate mapbox-gl css', () => {
  const codes = css2tsx(
    'MapboxGLCSS',
    path.join(
      __dirname,
      '../../../../node_modules/mapbox-gl/dist/mapbox-gl.css',
    ),
    {
      exclude: ['.mapboxgl-map'],
    },
  );

  generate(path.join(__dirname, '../index.tsx'), codes);
});
```

## @querycap-dev/generate-svg2tsx

@querycap-dev/generate-svg2tsx 用于将 svg 文件转换为 React 的 Icon 组件。

### 使用

```ts
import { generate, loadFiles } from '@querycap-dev/generate';
import { svg2tsx } from '@querycap-dev/generate-svg2tsx';

describe('generate', () => {
  it('Icon Components', () => {
    const files = loadFiles(['./icons/*.svg'], { cwd: __dirname });

    const codes = svg2tsx(files, {
      iconCreator: '@querycap-ui/icons.createIcon',
    });

    generate('../index.tsx', codes, { cwd: __dirname });
  });
});
```

### webpack 配置

```ts
import { generateClientFromConfig } from '@querycap-dev/generate-client';
import { withPresets } from '@querycap-dev/webpack-preset';
import { withAssetsPreset } from '@querycap-dev/webpack-preset-assets';
import { withHTMLPreset } from '@querycap-dev/webpack-preset-html';
import { withTsPreset } from '@querycap-dev/webpack-preset-ts';
import { set } from 'lodash';
import { DefinePlugin } from 'webpack';
import path from 'path';

export = withPresets(
  (c, state) => {
    console.log(state);

    if (!state.flags.production && !process.env.SKIP_CLIENT) {
      await generateClientFromConfig(state.meta.config, {
        cwd: __dirname,
        clientCreator: '@querycap/request.createRequestActor',
      });
    }

    if (process.env.HTTPS) {
      set(c, 'devServer', {
        browserSync: { https: !!process.env.HTTPS },
      });
    }
  },
  withTsPreset({
    polyfill: /babel|core-js/,
    charts: /d3-shape|d3-path|chart|antv/,
    styling: /polished|emotion|react-spring/,
    core: /react|reactorx|scheduler|history|axios/,
    utils: /buffer|date-fns|lodash|rxjs/,
    markdown: /unified|remark|remark-react/,
  }),
  (c, state) => {
    c.plugins?.push(
      new DefinePlugin({
        'process.env.APP': JSON.stringify(state.name),
      }),
    );

    c.resolve!.alias = {
      lodash$: 'lodash-es',
      path: 'path-browserify',
      stream: 'stream-browserify',
      querystring: 'querystring-es3',
    };
    c.resolve!.modules = [
      process.cwd(),
      // root node_modules first
      path.join(process.cwd(), 'node_modules'),
      // then related node_modules
      'node_modules',
    ];
    c.ignoreWarnings = [/Failed to parse source map/];
  },
  withAssetsPreset(),
  withHTMLPreset(),
);
```

当启动项目的时候，会自动根据该项目下的 config.ts 文件中的 APP_CONFIG 中的服务配置，生成 ts 类型文件，
生成的文件位于 src-clients 下，如上面 SRV_IDP 的服务，生成的文件位于 src-clients/idp/index.ts。

## webpack

与 webpack 相关的包有@querycap-dev/webpack-preset、@querycap-dev/webpack-browser-sync、@querycap-dev/webpack-preset-assets
、@querycap-dev/webpack-preset-html、@querycap-dev/webpack-preset-ts。

### @querycap-dev/webpack-preset

基础 webpack 预设

### @querycap-dev/webpack-browser-sync

webpack 本地开发服务器

### @querycap-dev/webpack-preset-assets

webpack 处理静态资源相关预设，如图片、css、json、html、md 文件处理相关 loader

### @querycap-dev/webpack-preset-html

webpack 处理 HTML 相关预设

### @querycap-dev/webpack-preset-ts

webpack 处理 ts 相关预设

### 使用

```ts
import { withPresets } from '@querycap-dev/webpack-preset';
import { withAssetsPreset } from '@querycap-dev/webpack-preset-assets';
import { withHTMLPreset } from '@querycap-dev/webpack-preset-html';
import { withTsPreset } from '@querycap-dev/webpack-preset-ts';
import { set } from 'lodash';
import { DefinePlugin } from 'webpack';
import path from 'path';

export = withPresets(
  (c, state) => {
    console.log(state);

    if (process.env.HTTPS) {
      set(c, 'devServer', {
        browserSync: { https: !!process.env.HTTPS },
      });
    }
  },
  withTsPreset({
    polyfill: /babel|core-js/,
    charts: /d3-shape|d3-path|chart|antv/,
    styling: /polished|emotion|react-spring/,
    core: /react|reactorx|scheduler|history|axios/,
    utils: /buffer|date-fns|lodash|rxjs/,
    markdown: /unified|remark|remark-react/,
  }),
  (c, state) => {
    c.plugins?.push(
      new DefinePlugin({
        'process.env.APP': JSON.stringify(state.name),
      }),
    );

    c.resolve!.alias = {
      lodash$: 'lodash-es',
      path: 'path-browserify',
      stream: 'stream-browserify',
      querystring: 'querystring-es3',
    };
    c.resolve!.modules = [
      process.cwd(),
      // root node_modules first
      path.join(process.cwd(), 'node_modules'),
      // then related node_modules
      'node_modules',
    ];
    c.ignoreWarnings = [/Failed to parse source map/];
  },
  withAssetsPreset(),
  withHTMLPreset(),
);
```

withPresets 可接收多个预设，也可以自定义预设覆盖默认配置。
