## 通用逻辑组件 ##
+ 创建包含重复逻辑的可复用组件
+ 试用场景： 通用列表

```
export interface IGenericListProps<T> {
    items: T[],
    itemRenderer: (item: T) => JSX.Element
}

export default class GenericList<T> extends React.Component<IGenericListProps<T>, {}> {
    render() {
        const { items, itemRenderer } = this.props;
        return (
            <>
                {items.map(itemRenderer)}
            </>
        )
    }
}
```