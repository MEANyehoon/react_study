```React.StatelessComponent<P>```or ```React.SFC<P>```

+ 无状态组件／函数式组件

```
const MyComp: React.SFC<IProps> = ...
```
---

```React.Component<IProps, IState>```

+ 有状态组件

```
class MyComp extends React.Component<Iptops, Istate> {...}
```

---

```React.ComponentType<P>```

+ 组件的联合类型(SFC | Component)

```
const withState = <P extends WrappedComponentProps>(
    WrappedComponent: React.ComponentType<P>,
) => { ...
```

---

```React.ReactElement<P> or JSX.Element```

+ React组件元素, 包括原生Dom标签

```
const element: React.ReactElement = <div /> || <MyComp />;
```

---

```React.ReactNode```

+ 包含React.ReactElement，还有任何原生的js类型

```
const elementOrPrimitive: React.ReactNode = 'string' || 0 || false || null || undefined || <div /> || <MyComp />
cosnt Component = ({children: React.ReactNode}) => ...
```

---

```React.CSSProperties```

+ JSX中使用的样式对象

```
const styles: React.CssProperties = { backgroundColor: 'red', ...};
const element = <div style={styles} />
```

---

```React.ReactEventHandler<E>```

+ 普通事件处理函数的类型

```
const handleChange: React.ReactEventHandler<HTMLInputElement> = (ev) => {...}
<input onChange={handleChange} ... />
```

---

```React.MouseEvent<E> | React.KeyboardEvent<E> | React.TouchEvent<E> | ...```

+ 特殊事件类型

```
const handleChange = (ev: React.MouseEvent<HTMLDivElement>) => (...)
<div onMouseMove={handleChange} ... />
```

---
