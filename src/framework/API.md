---
title: API
order: 2
toc: menu
---

## 配置

@querycap/config 用于解析项目下 config.ts 文件，并将配置对象传入全局 ConfigProvider，
使用时可以通过 useConfig 获取。

使用

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

const Setup = ({ children }: { children: ReactNode }) => {
  const store$ = useStore();
  const c = useConfig();

  return (
    <ThemeProvider>
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

## 启动

@querycap/bootstrap 创建一个 React 应用入口，内部会初始化 history、store、持久化数据、redux-logger，创建根组件，最终将
根组件以 ReactDOM.render 挂载到根节点。

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
import {
  AutoRedirectAccessible,
  Route,
  SwitchRoutes,
} from '@querycap-canary/routes';
import { healthRoutes } from './health/routes';
import { platformRoutes } from './platform/routes';
import { optimizeRoutes } from './optimize/routes';
import { dataRoutes } from './data/routes';
import { overviewRoutes } from './overview/routes';
import { supportRoutes } from './support/routes';

export const rootRoutes = (
  <Route>
    {loginRoutes}
    <Route path={'/'} content={<Main />}>
      <Route index content={<AutoRedirectAccessible />} />
      {overviewRoutes}
      {healthRoutes}
      {optimizeRoutes}
      {supportRoutes}
      {dataRoutes}
      {platformRoutes}
    </Route>
  </Route>
);

const Setup = ({ children }: { children: ReactNode }) => {
  const store$ = useStore();
  const c = useConfig();

  return (
    <ThemeProvider>
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

## 路由

@querycap/route-tree 对@reactor/router 进行封装，提供了路由权限以及 index 路由的功能，在实际使用中主要还是使用@querycap-canary/routes。

### 基本使用

```ts
import {
  EachRoutes,
  isVirtualRoute,
  Route,
  RouteProvider,
  SwitchRoutes,
  useNavigate,
  useRouteContext,
  load,
} from '@querycap-canary/routes';
import { IconOverview } from 'src-modules/power-plant/common/Icons';
import { getRouterByStrategy } from 'src-modules/power-plant/common/Access';

export const overviewRoutes = (
  <Route
    path="overview"
    title="总览"
    icon={<IconOverview />}
    render={getRouterByStrategy('bff-smart-power-plant.MOD_OVERVIEW')}
  >
    <Route index content={load(() => import('./Overview'))} />
  </Route>
);

export const rootRoutes = (
  <Route>
    {loginRoutes}
    <Route path={'/'} content={<FocusUserAndProject />}>
      <Route path={':appName/:projectID'} content={<Main />}>
        <Route index content={<AutoRedirectAccessibleByStrategy />} />
        {overviewRoutes}
        {healthRoutes}
        {optimizeRoutes}
        {supportRoutes}
        {dataRoutes}
        {platformRoutes}
      </Route>
    </Route>
  </Route>
);

export const bootstrap = createBootstrap(conf())(() => (
  <Setup>{rootRoutes}</Setup>
));
```

### Route

路由组件，根据 path 匹配对应的路径，渲染 content，render 可实现路由权限，在 render 内部可以决定是否渲染组件。index 标识根路由，title 和 icon 会显示在菜单栏。

### Link

```ts
import { Link } from '@reactorx/router';

const PlatformManage = () => {
  return (
    <div
      css={{
        display: 'flex',
        height: 'calc(100vh - 4em)',
        alignItems: 'center',
        justifyContent: 'center',
      }}
    >
      <Link to={'/optimize/feedback'}>
        <div css={select().marginTop('1em')}>意见反馈</div>
      </Link>
      <Link to={'/platform/user'}>
        <div
          css={select()
            .marginTop('1em')
            .color(colors.gray8)}
        >
          用户管理
        </div>
      </Link>
    </div>
  );
};
```

### NavLink

```ts
import { NavLink } from '@reactorx/router';

const Example = () => {
  return <NavLink to={`groups/${projectID}?name=${name}`}>查看</NavLink>;
};
```

### SwitchRoutes

类似 react-router-dom 的 Switch，如上面的 Main 内部就使用了 SwitchRoutes，当 content 内部使用了 SwitchRoutes 时，使用 Route 的 children 来实现多个子路由的渲染。content 结合 load 可以实现动态加载。

```ts
const Main = () => {
  return (
    <ThemeState fontSize={theme.fontSizes.s}>
      <ProLayout
        logo={logo}
        layout={'top'}
        renderHeader={<CurrentUser />}
        theme={'light'}
      >
        <SwitchRoutes />
      </ProLayout>
    </ThemeState>
  );
};
```

### 嵌套路由

```ts
export const platformRoutes = (
  <Route path="platform" title="平台管理" icon={<IconPlatform />}>
    <Route index content={load(() => import('./PlatformManage'))} />
    <Route path={'user'} content={load(() => import('./UserManage'))}>
      <Route index content={<AutoRedirectAccessible />} />
      {profileRoutes}
      {roleRoutes}
    </Route>
  </Route>
);
```

### 路径参数

```ts
<Route path={'history/:modelID'}>
  <Route index content={load(() => import('./HistoryRecord'))} />
  <Route
    path={':resultID'}
    content={load(() => import('./HistorySuggestDetail'))}
  />
</Route>
```

获取路径参数

```ts
import { useRouter } from '@reactorx/router';

const HistorySuggestDetail = () => {
  const { match, history } = useRouter();
  const { modelID, resultID } = match.params;

  return <>child</>;
};
```

query 参数解析

```ts
import { parseSearchString, toSearchString, useRouter } from '@reactorx/router';

export const UserMenu = ({ children }: { children?: ReactNode }) => {
  const triggerRef = useRef<HTMLDivElement>(null);
  const logout = useLogout();
  const { history } = useRouter();

  const [isOpened, openPopover, closePopover] = useToggle();

  useToggleControlOnClick(triggerRef, () =>
    isOpened ? closePopover() : openPopover(),
  );

  const [{ selectValue$ }, Select] = useNewSelect();

  useObservableEffect(() => {
    if (!triggerRef.current) {
      return;
    }

    return selectValue$.pipe(
      tap(opt => {
        if (opt === 'to-logout') {
          history.replace({
            search: toSearchString({
              ...parseSearchString(location.search),
              _from: 'close',
            }),
          }); // 兼容OAuth退出
          logout();
        }
        closePopover();
      }),
    );
  }, []);

  const user = useLogonUser();

  return (
    <ThemeState fontSize={roundedEm(0.9)}>
      <div
        css={select()
          .fontSize(theme.state.fontSize)
          .paddingTop(roundedEm(1.4))
          .paddingBottom(roundedEm(1))
          .paddingX(roundedEm(1.4))
          .with(
            select('& > a')
              .outline('none')
              .paddingY(roundedEm(0.6))
              .colorFill(pipe(theme.state.color, tint(0.1))),
          )}
      >
        <span ref={triggerRef} onClick={preventDefault}>
          <span css={select().display('block')}>
            <span>
              {user.identities[AccountIdentityType.NICKNAME] || '用户'}
            </span>
            &nbsp;
            <IconChevronDown />
          </span>
          <span
            css={select()
              .display('block')
              .opacity(0.4)}
          >
            <small>{user.accountID}</small>
          </span>
        </span>
        {isOpened && (
          <Select>
            <SelectMenuPopover
              triggerRef={triggerRef}
              onRequestClose={() => closePopover()}
              fullWidth
              placement={'bottom-left'}
            >
              {children}
              <MenuOptGroup>
                <div data-opt={'to-logout'}>退出登录</div>
              </MenuOptGroup>
            </SelectMenuPopover>
          </Select>
        )}
      </div>
    </ThemeState>
  );
};
```

其他

```ts
import {
  Navigate,
  Route,
  useLocation,
  useNavigate,
} from '@querycap-canary/routes';

const { pathname, query, hash, params } = useLocation();
const navigate = useNavigate();
```

## 请求

@querycap/request 提供了请求相关的 react hook、axios 全局 Provider、错误处理等 api。

### useRequest

useRequest 用于手动发起请求的场景，如表单。

```ts
import { useRequest } from '@reactorx/request';

const AcFormApp = ({
  action,
  app,
  onSuccess,
}: {
  action: string;
  app?: IApp;
  onSuccess?: () => void;
}) => {
  const [formCtx, Form] = useNewForm<typeof putApp.arg.body>('AcFormApp', app);
  const notify = useNotify();

  const [put] = useRequest(putApp, {
    onSuccess: () => {
      onSuccess && onSuccess();
      notify('success', `${action}成功`);
    },
    onFail: actor => {
      formCtx.setErrors(pickErrors(actor.arg.data.errorFields));
    },
  });

  return (
    <Form
      css={select().maxWidth(400)}
      onSubmit={values => {
        put({
          body: values,
        });
      }}
    >
      <FormControlWithField
        name="name"
        desc="url safe 字符"
        label={displayApp('name')}
        validate={pipe(required(), validateSafeURL())}
        readOnly={!!app}
      >
        {props => <SimpleInputText {...props} />}
      </FormControlWithField>
      <FormControlWithField name="fullName" label={displayApp('fullName')}>
        {props => <SimpleInputText {...props} />}
      </FormControlWithField>
      <FormControls>
        <Button type={'submit'} primary>
          {app ? '更新' : '创建'}
        </Button>
      </FormControls>
    </Form>
  );
};
```

### useTempDataOfRequest

```ts
declare const useTempDataOfRequest: <TRequestActor extends RequestActor<
  any,
  any
>>(
  requestActor: TRequestActor,
  arg: TRequestActor['arg'],
  deps?: any[],
  cacheKey?: string | undefined,
) => readonly [
  TRequestActor['done']['arg']['data'] | undefined,
  (
    req: TRequestActor['arg'] | undefined,
    opt?:
      | (axios.AxiosRequestConfig & {
          requestOptsFromReq?:
            | ((
                arg: TRequestActor['arg'],
              ) => _reactorx_request.IRequestOpts<any>)
            | undefined;
        } & Pick<
            _reactorx_request.IUseRequestOpts<
              TRequestActor['arg'],
              TRequestActor['opts'],
              IStatusError
            >,
            'onSuccess' | 'onFail'
          >)
      | undefined,
  ) => void,
  rxjs.BehaviorSubject<boolean>,
];
```

用于组件挂载完成时发起请求，内部是对 useRequest 的封装。

请求发出的时机：

- 组件实例第一次挂载后；
- deps 数组中任意值变化（浅比较）；

参数：

- requestActor：必传，src-clients/xxx 下对应自动生成的网关接口函数；
- arg：必传，请求的参数（token 不用传，拦截器中默认写入 header 中了）；
- deps：可选，数组(同 useEffect)，当改数组中任何一个值变化(浅比较)，都会重新发起请求；
- cacheKey：可选，字符串。标定当前请求通道的唯一性。主要用于多个请求(同一接口，不同请求参数)同时发起时, 用于区分。

返回值：

- data: 200 时的响应数据，默认：undefined；
- request：手动再次发起请求的方法，参数为请求参数 或 undefined(表示使用默认的请求参数)；
- requesting\$: 表示是否请求中的 observable，通常使用 useObservable 获取该布尔值；

```ts
import { useInstance } from '@querycap-canary/oauth/Instance';
import { tapWhen, useTempDataOfRequest } from '@querycap/request';
import {
  getGraphDeviceConfig,
  saveGraphDeviceRelation,
} from 'src-clients/bff-smart-power-plant';
import { useObservable } from '@reactorx/core';

const MachineSetMonitor = () => {
  const graphID = useGraphID();
  const { projectID, appName, env } = useInstance();
  const [devices, fetch, requesting$] = useTempDataOfRequest(
    getGraphDeviceConfig,
    {
      projectID,
      appIDOrName: appName,
      env,
      graphID,
    },
    [projectID, graphID],
  );

  const requesting = useObservable(requesting$);

  return (
    <div>
      {epicOn(tapWhen(() => fetch(undefined), saveGraphDeviceRelation))}
      <div
        css={select()
          .display('flex')
          .justifyContent('flex-end')}
      >
        <Stack inline spacing={roundedEm(0.6)}>
          {!requesting && (
            <AcCollectionPointConfig
              graphID={graphID}
              devices={devices || []}
            />
          )}
          <ConfigGraphButton graphID={graphID} />
        </Stack>
      </div>
      <MachineSetPreview graphID={graphID} devices={devices || []} />
    </div>
  );
};
```

### AxiosProvider

Axios 全局 Provider，提供 baseURL 以及拦截器配置。

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

const Setup = ({ children }: { children: ReactNode }) => {
  const store$ = useStore();
  const c = useConfig();

  return (
    <ThemeProvider>
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
```

### useAxiosInstance

获取 Axios 实例

```ts
import { RequestActor, useAxiosInstance } from '@querycap/request';
import { useRef } from 'react';

export const useTilesOfRequest = <
  T extends RequestActor<{ z: number; x: number; y: number }>
>(
  requestActor: T,
  arg: Omit<T['arg'], 'x' | 'y' | 'z'>,
) => {
  const axios = useAxiosInstance();
  const argRef = useRef(arg);

  return [
    axios.getUri(
      // eslint-disable-next-line
      requestActor
        .with({
          ...argRef.current,
          z: `{z}` as any,
          x: `{x}` as any,
          y: `{y}` as any,
        })
        .requestConfig(),
    ),
  ];
};
```

### tapWhen

方法签名

```ts
tapWhen(next: () => void, ...actors: RequestActor[])
```

当请求完成时执行相应方法，通常和 epicOn 搭配使用，使用场景：当创建或者更新请求完成后刷新列表。

```ts
import { useInstance } from '@querycap-canary/oauth/Instance';
import { useRequest } from '@reactorx/request';
import {
  createIssue,
  deleteRepairRecord,
  displayGitQuerycapComCustomProjectsCommonTypesxIssueState as displayIssueState,
  displayPowerPlantUrgencyDegree,
  IObjectMeta,
  listIssue,
  updateExpertComment,
} from 'src-clients/bff-smart-power-plant';
import { useNewSearchOfRequest } from '@querycap/search';
import { epicOn, useObservable } from '@reactorx/core';
import { tapWhen } from '@querycap/request';
import { Table } from '@querycap-ui/blocks';
import { PaginationWithTotal } from '@querycap-ui/search-controls';

const IssueList = () => {
  const { projectID, appName, env } = useInstance();
  const [
    { state$, setPager },
    Search,
    fetch,
    requesting$,
  ] = useNewSearchOfRequest(
    listIssue,
    {},
    {
      size: 10,
      syncURL: true,
      queryToArg: (filters, query) => {
        return {
          appIDOrName: appName,
          projectID,
          env,
          ...filters,
          ...query,
        };
      },
    },
  );

  const state = useObservable(state$);
  const requesting = useObservable(requesting$);

  return (
    <Search>
      {epicOn(tapWhen(() => fetch(), createIssue, deleteRepairRecord))}
      <Table
        align={'center'}
        rowKey={'reporterID'}
        columns={columns}
        loading={requesting}
        dataSource={state.data}
      />
      <div css={select().paddingBottom('1em')}>
        {!!state.pager.total && (
          <PaginationWithTotal pager={state.pager} onPagerChange={setPager} />
        )}
      </div>
    </Search>
  );
};
```

## 权限

@querycap/access 下包含 Token 信息、权限认证相关的模块。

### AccessControlProvider

提供权限

```ts
import { AccessControlProvider } from '@querycap/access';
import { useStoreState$ } from '@querycap/contexts';
import { RequestActor } from '@querycap/request';
import { useObservable } from '@reactorx/core';
import { useRequest } from '@reactorx/request';
import { ReactNode, useEffect } from 'react';
import { IGitQuerycapComIdpSrvIdpPkgAppmgrInstanceUserContext as IInstanceUserContext } from 'src-clients/idp';
import { instanceID, useInstance } from './Instance';
import { useLogonUser } from './LogonUser';

export const MustACL = ({
  children,
  currentUserPermissions,
}: {
  currentUserPermissions: RequestActor<
    | {
        projectID?: string;
        appIDOrName?: string;
        env?: string;
      }
    | undefined,
    IInstanceUserContext
  >;
  children: ReactNode;
}) => {
  const { accountID } = useLogonUser();
  const { projectID, appName, env } = useInstance();

  const [acl$, setAcl] = useStoreState$<IInstanceUserContext>(
    `${accountID}:${instanceID({ projectID, appName, env })}:permissions`,
    undefined,
    {
      crossTabs: true,
      expiresIn: 5 * 60,
    },
  );

  const [fetch] = useRequest(currentUserPermissions, {
    onSuccess: ({ arg }) => {
      setAcl(arg.data);
    },
  });

  const acl = useObservable(acl$) as IInstanceUserContext;

  useEffect(() => {
    fetch({ projectID, appIDOrName: appName, env });
  }, [projectID, appName, env]);

  return acl ? (
    <AccessControlProvider
      value={{
        permissions: acl.permissions,
        attrs: acl.attrs,
        strategies: acl.strategies,
      }}
    >
      {children}
    </AccessControlProvider>
  ) : null;
};
```

### useAccessControl

获取权限

```ts
import { ReactNode } from 'react';
import { useAccessControl } from '@querycap/access';
import { get } from 'lodash';

export const getRouterByStrategy = (strategy: string) => {
  const RenderByStrategy = ({
    children,
  }: {
    children: ReactNode;
  }): JSX.Element | null => {
    const { strategies } = useAccessControl();
    if (!get(strategies, strategy)) return null;

    return <>{children}</>;
  };
  return RenderByStrategy;
};
```

### useAccess

获取登陆 Token 信息

### useAccessMgr

对 Token 信息进行获取、修改、删除

```ts
const [access$, set, del] = useAccessMgr();
```

```ts
import {
  createUnauthoriredHandleEpic,
  fromOAuthToken,
  hasLogon,
  IToken,
  useAccess,
  useAccessMgr,
} from '@querycap/access';

export const MustNotLogon = ({ children }: { children: ReactNode }) => {
  const access = useAccess();
  const { query } = useLocation<
    any,
    { redirect_uri: string; state: string; client_id: string }
  >();
  const [isClear, setClear] = useState(
    query.state && query.state.indexOf('!user-switch') > -1,
  );
  const [, , del] = useAccessMgr();

  return (
    <>
      {!hasLogon(access) ? (
        <>{children} </>
      ) : (
        <SuperRedirectAuthorize
          query={query}
          to={redirectURLFromQueryOfLoginPage(query)}
        />
      )}
    </>
  );
};
```

### mustOneOfPermissions、mustAllOfPermissions

组件级别权限配置，传入一个或者多个接口进行权限控制，内部通过 useAccessControl 获取当前用户的权限列表进行控制。

mustOneOfPermissions:配置组件拥有一个接口权限即可显示
mustAllOfPermissions：配置组件拥有传入的所有接口权限即可显示

```ts
import { mustOneOfPermissions } from '@querycap/access';
import { saveGraphDevicePosition } from 'src-clients/bff-smart-power-plant';

export const ConfigGraphButton = mustOneOfPermissions(saveGraphDevicePosition)(
  ({ graphID }: { graphID: string }) => {
    return (
      <Link to={`/optimize/config/${graphID}`}>
        <Button type="button" small primary css={select().marginY('0.5em')}>
          组态图配置
        </Button>
      </Link>
    );
  },
);
```

如果组件内部使用到 useRequest 或者 useTempDataOfRequest,可直接通过添加组件前缀 Ac 来设置权限，需要配合 babel 插件：@querycap/babel-plugin-access-control-autocomplete 使用

```ts
const AcClientSecretList = ({
  modID,
  clientID,
}: {
  modID: string;
  clientID: string;
}) => {
  const [data, request] = useTempDataOfRequest(
    listModClientSecret,
    {
      modIDOrName: modID,
      clientID: clientID,
      size: -1,
    },
    [modID, clientID],
    `modClientSecret:${modID}:${clientID}`,
  );

  return (
    <>
      {epicOn(tapWhen(() => request(undefined), newModClientSecret))}
      <Stack spacing={roundedEm(2)}>
        {map(data?.data, token => (
          <TokenItem key={token.tokenID} token={token} />
        ))}
      </Stack>
    </>
  );
};
```

babel.config.js

```js
module.exports = {
  plugins: [
    [
      '@querycap/babel-plugin-access-control-autocomplete',
      {
        libAccessControl: '@querycap-canary/routes',
      },
    ],
  ],
};
```

## 通知

@querycap/notify 用于全局的通知消息弹窗功能。

### 初始化

```ts
import { NotificationHub, Confirmations } from '@querycap-canary/dashboard';
import { useConfig } from '@querycap/config';
import { AxiosProvider, baseURLsFromConfig } from '@querycap/request';
import { useStore } from '@reactorx/core';

const Setup = ({ children }: { children: ReactNode }) => {
  const store$ = useStore();
  const c = useConfig();

  return (
    <ThemeProvider>
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
```

### 使用

useNotify 用于弹出一个通知消息，几秒后自动消失。支持 info、success、error、warning 等多种样式。

useConfirm 用于交互式弹窗，常用于确认某种操作的场景。

```ts
export const DeleteEntity = ({ entityID }: { entityID: string }) => {
  const confirm = useConfirm();
  const { projectID, appName, env } = useInstance();
  const notify = useNotify();
  const [del] = useRequest(deleteEntity, {
    onSuccess: () => {
      notify('success', '删除成功！');
    },
  });

  return (
    <a
      href="#"
      onClick={pipe(preventDefault, () => {
        confirm(`是否删除该实体`, confirmed => {
          confirmed &&
            del({
              appIDOrName: appName,
              projectID,
              env,
              entityID,
            });
        });
      })}
    >
      删除
    </a>
  );
};
```

## 表单

@querycap/form 提供表单场景下的状态管理解决方案。

### useNewForm

```ts
const [formCtx, Form] = useNewForm<typeof putApp.arg.body>('AcFormApp', app);
```

useNewForm 创建一个 Form 组件，参数分别为 Form 的 name（唯一）和表单初始值。

在 formCtx 上可以获取到表单的所有状态。

```ts
const state$ = formCtx.state$;

const state = useObservable(state$);
```

例子：

```ts
import { SimpleInputText, useNewForm } from '@querycap/form';

const AcFormApp = ({
  action,
  app,
  onSuccess,
}: {
  action: string;
  app?: IApp;
  onSuccess?: () => void;
}) => {
  const [formCtx, Form] = useNewForm<typeof putApp.arg.body>('AcFormApp', app);
  const notify = useNotify();

  const [put] = useRequest(putApp, {
    onSuccess: () => {
      onSuccess && onSuccess();
      notify('success', `${action}成功`);
    },
    onFail: actor => {
      formCtx.setErrors(pickErrors(actor.arg.data.errorFields));
    },
  });

  return (
    <Form
      css={select().maxWidth(400)}
      onSubmit={values => {
        put({
          body: values,
        });
      }}
    >
      <FormControlWithField
        name="name"
        desc="url safe 字符"
        label={displayApp('name')}
        validate={pipe(required(), validateSafeURL())}
        readOnly={!!app}
      >
        {props => <SimpleInputText {...props} />}
      </FormControlWithField>
      <FormControlWithField name="fullName" label={displayApp('fullName')}>
        {props => <SimpleInputText {...props} />}
      </FormControlWithField>
      <FormControls>
        <Button type={'submit'} primary>
          {app ? '更新' : '创建'}
        </Button>
      </FormControls>
    </Form>
  );
};
```

### 自定义表单组件

自定义表单组件需要提供 value 和 onValueChange 两个 props。

```ts
const ColorsSetting = ({
  value,
  onValueChange,
}: {
  value: IColors;
  onValueChange: (colors: IColors) => void;
}) => {
  const [fontColor, bgColor] = value;

  return (
    <Stack inline align="flex-start" spacing={40}>
      <Stack spacing={10} align="flex-start">
        <div
          css={select()
            .padding('.6em 4em')
            .lineHeight(1)
            .color(fontColor)
            .backgroundColor(bgColor)}
        >
          效果预览
        </div>
        <InlineLabel label="文字颜色">{fontColor}</InlineLabel>
        <InlineLabel label="背景颜色">{bgColor}</InlineLabel>
      </Stack>
      <ColorPicker
        value={fontColor}
        onValueChange={(color: string) => {
          onValueChange([color, bgColor]);
        }}
      >
        <IconFont inline css={select().marginRight('.5em')} />
        文字颜色
      </ColorPicker>
      <ColorPicker
        value={bgColor}
        onValueChange={(color: string) => {
          onValueChange([fontColor, color]);
        }}
      >
        <IconBgColor inline css={select().marginRight('.5em')} />
        背景颜色
      </ColorPicker>
    </Stack>
  );
};
```

自定义表单组件使用

```ts
import { FormControlWithNoInput } from '@querycap-canary/dashboard/FormControlNoInput';

<FormControlWithNoInput
  label={<RequiredLabel>颜色设置</RequiredLabel>}
  name="colors"
  validate={required()}
>
  {props => <ColorsSetting {...props} />}
</FormControlWithNoInput>;
```

### 自定义表单验证

```ts
import { createValidator, required } from '@querycap/validators';

const validateSafeURL = createValidator('必须是 URL 安全字符', v => {
  return /^[A-Za-z0-9-_]+$/.test(v);
});
```

## 搜索

@querycap/search 提供了一套搜索场景下的解决方案。

### useNewSearchOfRequest

useNewSearchOfRequest 通过传入一个请求方法，返回 Search 组件、state、setFilters、setPager，state 为接口返回数据，通过 setFilters 方法来改变筛选项，setPager 改变分页参数来自动请求新的数据。

syncURL 参数为是否同步参数到 url 上，当页面刷新时，根据 url 上的 filter 参数请求数据。

下面是一个简单的案例：

```ts
import { createSearchBox, searchInput } from '@querycap-ui/search-controls';
import {
  SearchInputTimeRange,
  createTimeRangeDisplay,
} from '@querycap-canary/dashboard/SearchInputTimeRange';
import { tapWhen } from '@querycap/request';
import { Pager, useNewSearchOfRequest } from '@querycap/search';

const AppListSearchBox = createSearchBox<Omit<typeof listApp.arg, keyof Pager>>(
  {
    name: searchInput()
      .label('名称')
      .sortable()
      .wildcard(),
    appID: searchInput()
      .multiple()
      .label('应用 ID'),
    createdAt: searchInput(props => (
      <SearchInputTimeRange {...props} maxValue={formatRFC3339(Date.now())} />
    ))
      .display(createTimeRangeDisplay('yyyy-MM-dd'))
      .label('创建时间')
      .sortable(),
    sort: searchInput().defaultValue('createdAt'),
  },
);

const AcAppList = () => {
  const [{ state$, setFilters }, Search, request] = useNewSearchOfRequest(
    listApp,
    AppListSearchBox.defaultFilters,
    {
      name: 'app',
      size: 30,
      syncURL: true,
    },
  );

  const state = useObservable(state$);

  return (
    <Section>
      {epicOn(tapWhen(() => request(undefined), putApp, delApp))}
      <Search>
        <Stack spacing={roundedEm(2)}>
          <AppListSearchBox filters={state.filters} onSubmit={setFilters} />
          {map(state.data, app => (
            <AppCard key={app.appID} app={app} />
          ))}
          <SearchPagination />
        </Stack>
      </Search>
    </Section>
  );
};
```

使用此 hook 的接口也需要一些条件：返回的结构必须为如下形式：

```ts
export interface IListCollectTaskResp {
  data: ICollectTaskDetail[];
  total: number;
}
```

必须包含 data 字段。

定义

```ts
declare const useNewSearchOfRequest: <
  TRequestActor extends RequestActor<any, any>,
  TFilters = TRequestActor['arg']
>(
  requestActor: TRequestActor,
  defaultFilters: TFilters,
  {
    size,
    offset,
    name,
    syncURL,
    queryToArg,
  }?: {
    size?: number | undefined;
    offset?: number | undefined;
    name?: string | undefined;
    syncURL?: boolean | undefined;
    queryToArg?:
      | ((
          filters: TFilters,
          pager: Omit<Pager, 'total'>,
        ) => TRequestActor['arg'])
      | undefined;
  },
) => readonly [
  {
    state$: _______reactorx_core.Volume<
      any,
      SearchState<TFilters, TRequestActor['done']['arg']['data']['data'][0]>
    >;
    initial: () => void;
    destroy: () => void;
    setFilters: (arg: Dictionary<any>, opts?: undefined) => void;
    setPager: (
      arg: {
        offset: number;
        size: number;
      },
      opts?: undefined,
    ) => void;
    setData: (
      data: TRequestActor['done']['arg']['data']['data'][0][],
      total?: number | undefined,
      append?: boolean | undefined,
    ) => void;
  },
  ({ children }: { children: ReactNode }) => JSX.Element,
  (query?: TRequestActor['arg']) => void,
  rxjs.BehaviorSubject<boolean>,
];
```

useNewSearchOfRequest 可以和 createSearchBox 结合，非常方便得创建搜索表单。

createSearchBox 目前有 Input、Select 等组件，也可以自定义搜索表单组件。

下面是一个比较全面的案例：

```ts
import { useNewSearchOfRequest } from '@querycap/search';

import {
  createSearchBox,
  createTimeRangeDisplay,
  PaginationWithTotal,
  searchInput,
  SearchInputProps,
  SearchInputSelect,
} from '@querycap-ui/search-controls';
import {
  listCollectTask,
  updateCollectTaskStatus,
  GitQuerycapComAisysCommonTypesxMaterialType as MaterialType,
  displayGitQuerycapComAisysCommonTypesxMaterialType as displayMaterialType,
  MaterialCollectionTaskStatus as TaskStatus,
  displayMaterialCollectionTaskStatus as displayTaskStatus,
  IGitQuerycapComAisysCommonTypesxMaterialSource as IMaterialSource,
  GitQuerycapComAisysCommonTypesxMaterialSource as MaterialSource,
  ICollectTaskDetail,
  getMaterialCountByCollectTask,
  IMaterialCollectionTaskStatus,
  MaterialCollectionTaskStatus,
  listAccounts,
  uploadMaterial,
  getCollectTasksPerformance,
} from 'src-clients/bff-material-collection';
import { useSimpleStore } from '../common/hooks';

const useAccountOption = () =>
  useSimpleStore<ISelectOption>('accountID', {}, false);

const SearchWithUser = (props: SearchInputProps) => {
  const [, setAccountsOption] = useAccountOption();
  const { projectID, appName, env } = useInstance();
  const [accounts] = useTempDataOfRequest(
    listAccounts,
    { projectID, appIDOrName: appName, env, size: -1 },
    [projectID],
  );

  const accountsMap = useMemo(() => {
    return generateEnumAndDisplay(
      accounts?.data || [],
      'accountID',
      'identities.NICKNAME',
    );
  }, [accounts]);

  useEffect(() => {
    setAccountsOption(accountsMap);
  }, [accountsMap]);

  return (
    <SearchInputSelect
      {...props}
      enum={accountsMap.enum}
      display={accountsMap.display}
    />
  );
};

const UserDisplay = (v: string) => {
  const [accountsOption] = useAccountOption();
  if (!accountsOption.display) {
    return v;
  }

  return accountsOption.display(v);
};

type ISearcher = Pick<
  typeof listCollectTask.arg,
  'taskStatus' | 'materialType' | 'accountID' | 'dataSetIDs' | 'createdAt'
> & {
  collectTaskID: string;
};

const TaskListSearcher = createSearchBox<ISearcher>({
  collectTaskID: searchInput()
    .label('任务ID')
    .wildcard(),
  materialType: searchInput(SearchInputSelect)
    .enum(keys(MaterialType))
    .display(displayMaterialType)
    .label('数据类型'),
  taskStatus: searchInput(SearchInputSelect)
    .enum(keys(TaskStatus))
    .display(displayTaskStatus)
    .label('采集状态'),
  accountID: searchInput(SearchWithUser)
    .label('用户')
    .display(UserDisplay),
  createdAt: searchInput(props => (
    <SearchInputTimeRange {...props} maxValue={formatRFC3339(Date.now())} />
  ))
    .label('创建时间')
    .display(createTimeRangeDisplay('yyyy-MM-dd'))
    .defaultValue(`..${formatRFC3339(endOfDay(Date.now()))}`),
});

export const AcTaskList = ({
  sourceType,
}: {
  sourceType?: IMaterialSource;
}) => {
  const { env, appName, projectID } = useInstance();
  const [tasks, setTasks] = useState<ICollectTaskDetailWithCount[]>([]);
  const { match } = useRouter();
  const { dataSetID } = match.params as { dataSetID: string };
  const [
    { state$, setFilters, setPager },
    Search,
    request,
    requesting$,
  ] = useNewSearchOfRequest(listCollectTask, TaskListSearcher.defaultFilters, {
    name: 'ListCollectTask',
    size: 10,
    syncURL: true,
    queryToArg: (filters, query) => {
      return {
        appIDOrName: appName,
        projectID,
        env,
        sourceType,
        ...filters,
        dataSetIDs: [dataSetID],
        collectTaskIDs: [filters.collectTaskID],
        ...query,
      };
    },
  });

  const state = useObservable(state$);

  const taskList = state?.data || [];

  const [requestCount] = useRequest(getMaterialCountByCollectTask, {
    onSuccess: ({ arg }) => {
      const countMap = reduce(
        arg.data,
        (v, c) => {
          return {
            ...v,
            [c.taskID]: c.count,
          };
        },
        {} as Dictionary<number>,
      );

      const data = map(taskList, item => {
        return {
          ...item,
          count: countMap[item.taskID] || 0,
        };
      });
      setTasks(data);
    },
  });
  const [requestPerformance] = useRequest(getCollectTasksPerformance, {
    onSuccess: ({ arg }) => {
      const data = map(tasks, item => {
        return {
          ...item,
          time: arg.data[item.taskID]?.nanosPerItem || 0,
        };
      });
      setTasks(data);
    },
  });
  useEffect(() => {
    if (!size(taskList)) {
      setTasks([]);
      return;
    }
    requestCount({
      projectID,
      appIDOrName: appName,
      env,
      body: {
        taskAndDataSet: map(state.data, item => ({
          taskID: item.taskID,
          dataSetID: item.dataSetID,
        })),
      },
    });
  }, [state.data]);
  useEffect(() => {
    if (!size(tasks)) {
      return;
    }
    requestPerformance({
      projectID,
      appIDOrName: appName,
      env,
      collectTaskIDs: map(state.data, 'taskID'),
    });
  }, [tasks.length]);
  const requesting = useObservable(requesting$);
  return (
    <>
      {epicOn(
        tapWhen(() => request(), updateCollectTaskStatus, uploadMaterial),
      )}
      <Search>
        <TaskListSearcher filters={state.filters} onSubmit={setFilters} />
        <Gap />
        <LoadingTable
          loading={requesting}
          rowKey="taskID"
          columns={
            sourceType === MaterialSource.MANUAL ? columnsOfManualTask : columns
          }
          dataSource={tasks}
        />
        <Gap />
        <div>
          {!!state.pager.total && (
            <PaginationWithTotal pager={state.pager} onPagerChange={setPager} />
          )}
        </div>
      </Search>
    </>
  );
};
```

下面是一个自定义搜索组件的例子：

```ts
import { preventDefault } from '@querycap-ui/core/macro';
import {
  Button,
  MenuPopover,
  useKeyboardArrowControls,
} from '@querycap-ui/form-controls';
import { parseRange, stringifyRange } from '@querycap/strfmt';
import { useToggle } from '@querycap/uikit';
import { useObservableEffect } from '@reactorx/core';
import { endOfDay, format, formatRFC3339, parseISO } from 'date-fns';
import { noop } from 'lodash';
import { useEffect, useRef, useState } from 'react';
import { fromEvent, merge } from 'rxjs';
import { filter as rxFilter, tap } from 'rxjs/operators';
import { displayValue, SearchInputProps } from '@querycap-ui/search-controls';
import { DateHistoryRangePicker } from '@querycap-ui/date-pickers';

export const createTimeRangeDisplay = (f = 'yyyy-MM-dd HH:mm') => (
  v: string,
) => {
  const r = parseRange(v);

  let d = '';

  if (r.from) {
    d += format(parseISO(r.from), f);
  }

  if (r.to) {
    d += ' 至 ';
    d += format(parseISO(r.to), f);
  } else {
    d += ' 至今';
  }

  return d;
};

export const SearchInputTimeRange = ({
  minValue,
  maxValue,
  display,
  defaultValue,
  onSubmit,
  onCancel,
}: SearchInputProps & {
  minValue?: string;
  maxValue?: string;
}) => {
  const [timeRange, setTimeRange] = useState(() => parseRange(defaultValue));

  const inputElmRef = useRef<HTMLInputElement>(null);

  const [isOpened, openPopover, closePopover] = useToggle();

  useKeyboardArrowControls(inputElmRef, d => {
    switch (d) {
      case 'down':
        return;
      case 'up':
        return;
    }
  });

  useObservableEffect(() => {
    if (!inputElmRef.current) {
      return;
    }

    const inputClick$ = fromEvent<MouseEvent>(inputElmRef.current, 'click');
    const inputFocus$ = fromEvent<FocusEvent>(inputElmRef.current, 'focus');
    const inputKeydown$ = fromEvent<KeyboardEvent>(
      inputElmRef.current,
      'keydown',
    );

    const onKey = (k: string) => rxFilter((e: KeyboardEvent) => e.key === k);

    const inputKeydownEnter$ = inputKeydown$.pipe(onKey('Enter'));

    return [
      inputKeydownEnter$.pipe(tap(preventDefault)),

      merge(inputFocus$, inputClick$).pipe(tap(() => openPopover())),
    ];
  }, []);

  useEffect(() => {
    inputElmRef.current?.focus();
  }, []);

  return (
    <>
      <input
        ref={inputElmRef}
        type="text"
        value={String(displayValue(stringifyRange(timeRange), display))}
        onChange={noop}
        readOnly
      />
      {isOpened && (
        <MenuPopover
          triggerRef={inputElmRef}
          onRequestClose={() => closePopover()}
          placement={'bottom-left'}
        >
          <div>
            <DateHistoryRangePicker
              value={[timeRange.from || '', timeRange.to || '']}
              min={minValue}
              max={maxValue}
              onValueChange={([from, to]) => {
                setTimeRange(r => ({
                  ...r,
                  from,
                  to: formatRFC3339(endOfDay(new Date(to))),
                }));
              }}
            />
          </div>
          <div
            css={{
              padding: '0.3em 1em 1',
              display: 'flex',
              justifyContent: 'flex-end',
              '& > * + *': { marginLeft: '1em' },
            }}
          >
            <Button invisible small onClick={onCancel}>
              取消
            </Button>
            <Button
              primary
              small
              onClick={() => {
                console.log(timeRange, stringifyRange(timeRange));
                onSubmit(stringifyRange(timeRange));
              }}
            >
              确定
            </Button>
          </div>
        </MenuPopover>
      )}
    </>
  );
};
```

## 状态管理

@querycap/contexts 提供可持久化的状态管理。

### useStoreState\$

签名

```ts
declare function useStoreState$<T>(
  topic: string,
  initialState: undefined | T | (() => T),
  persistOpts?: {
    crossTabs?: boolean;
    expiresIn?: number;
  },
): readonly [rxjs.Observable<any>, (stateOrUpdater: T | TUpdater<T>) => void];
```

topic 为 state 的 key，initialState 为初始状态，persistOpts 为持久化选项。

使用

```ts
export const MustLogonUser = ({
  currentUser,
  children,
}: {
  currentUser: RequestActor<{ env?: string }>;
  children: ReactNode;
}) => {
  const c = useConfig();
  const access: IToken = useAccess();

  const [logonUser$, setLogonUser] = useStoreState$<LogonUser>(
    `${access?.uid || ''}:logonUser`,
    undefined,
    {
      crossTabs: true,
      expiresIn: 5 * 60,
    },
  );

  const [getCurrentUser] = useRequest(currentUser, {
    onSuccess: ({ arg }) => {
      setLogonUser(
        user =>
          ({
            ...user,
            ...arg.data,
          } as LogonUser),
      );
    },
  });

  useEffect(() => {
    getCurrentUser({
      env: (c as BaseConfig & { APP_ENV: string })?.APP_ENV || c.env,
    });
  }, []);

  const logonUser = useObservable(logonUser$) as LogonUser;

  return logonUser ? (
    <LogonUserProvider value={{ logonUser, setLogonUser }}>
      {children}
    </LogonUserProvider>
  ) : null;
};
```

### createUseState

useStoreState$的封装，简化了useStoreState$的使用。

签名

```ts
declare function createUseState<T>(
  topic: string,
  initialState?: T,
  persistOpts?: {
    crossTabs?: boolean;
    expiresIn?: number;
  },
): () => readonly [any, (stateOrUpdater: T | TUpdater<T>) => void];
```

### useQueryState

```ts
declare const useQueryState: <T extends string | string[]>(
  queryKey: string,
  defaultValue: T,
) => readonly [T, react.Dispatch<react.SetStateAction<T>>];
```

用于将数据同步到 url 上的 query 参数上

使用：

```ts
const [search, setSearch] = useQueryState('key', 'value');
```

当每次调用 setSearch 的时候，url 上的参数也会同步改变。
