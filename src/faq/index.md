---
title: FAQ
order: 1
toc: menu
nav:
  title: FAQ
  path: /faq
  order: 4
---

## 概览

用于记录一些项目中不常见的坑和解决办法。

## rxjs 和 hooks 使用陷阱

部分 UI 组件内部使用 rxjs 导致父组件回调函数中获取不到最新的状态
如 InputSelect 组件内部在 useEffect 内部使用 rxjs 绑定事件，当选择某项的时候，调用 onValueChange 传递选项给父组件，如果父组件在 onValueChange 回调中拷贝之前的 state 进行修改，然后更新 state，可能在回调中获取不到最新的 state。

```ts
import { useState } from 'react';
import { times, cloneDeep, map } from 'lodash';
import { headings } from '../../texts/utils';
import { Stack } from '../../layouts/Stack';
import { roundedEm, theme } from '../../core/theme';
import { select } from '../../core/select';
import { Input } from '../Input';
import { InputSelect } from '../InputSelect';

export const InputSelects = () => {
  const [list, setList] = useState<Array<{ name: string }>>([]);

  const enums = ['Apple', 'Orange', 'Banana'];

  const handleChange = (val: string, index: number) => {
    const newVal = cloneDeep(list);
    newVal[index].name = val;
    setList(newVal);
  };

  return (
    <>
      {map(list, (val, index) => {
        return (
          <Input>
            <InputSelect
              allowClear
              enum={enums}
              value={val}
              onValueChange={val => handleChange(val, index)}
            />
          </Input>
        );
      })}
    </>
  );
};
```

原因：由于 InputSelect 组件内部的 rxjs 事件监听函数只在组件挂载后执行一次，而 onValueChange 回调依赖了外部状态，外部状态的更新导致回调函数更新，而 InputSelect 内部的 onValueChange 不是最新的函数，故出现了 state 渲染出错的问题。

解决：
setState 采用回调函数来获取最新的 state。

```ts
const handleChange = (val: string, index: number) => {
  setList(prev => {
    const newVal = cloneDeep(prev);
    newVal[index].name = val;
    return newVal;
  });
};
```

## 自定义 hooks 的使用陷阱

部分自定义 hooks 使用不当导致页面崩溃
之前遇到过一次，开发环境页面正常，生产环境页面崩溃出现白屏，
控制台报错指向如下链接：https://reactjs.org/docs/error-decoder.html/?invariant=300
大概意思是 hooks 执行顺序不对，
最终定位到出错的组件大概逻辑如下：

```ts
import { useObservable } from '@reactorx/core';

const Example = () => {
  const { projectID, appName, env } = useInstance();
  const [data, , requesting$] = useTempDataOfRequest(
    listEconomyIndex,
    {
      projectID,
      appIDOrName: appName,
      env,
      dataType: time?.timeType as IDataReportTimeTypeTimeType,
      dataTime: time ? from : undefined,
    },
    [projectID, time],
  );

  const requesting1 = useObservable(requesting$);

  const [data2, , requesting$2] = useTempDataOfRequest(
    listEconomyIndex,
    {
      projectID,
      appIDOrName: appName,
      env,
      dataType: time?.timeType as IDataReportTimeTypeTimeType,
      dataTime: time ? from : undefined,
    },
    [projectID, time],
  );

  const requesting2 = useObservable(requesting$2);

  if (requesting1 || requesting2) {
    return <div>Loading</div>;
  }
};
```

以上组件出问题的 hook 是 useObservable，可以看到上面组件实现的功能是对两个请求的 loading 状态做了一个或运算，而 Webapck 在 prod 模式下启用编译压缩会将该或运算优化为如下代码：

```ts
if (useObservable(requesting$) || useObservable(requesting$2)) {
  return <div>Loading</div>;
}
```

由于或运算的短路特点，就会出现两个 hook 可能只执行一个的情况，生产环境下该页面就出现崩溃的现象。

解决：

```ts
const loading1 = useObservable(requesting$);

const loading2 = useObservable(requesting$2);

if (loading1) {
  return <div>Loading</div>;
}
if (loading2) {
  return <div>Loading</div>;
}
```
