## 默认参数组件 ##
+ 组件内部类型安全
```
interface IProps {
    name: string
}

interface IDefaultProps {
    age: number
}

const CompWithDefault: React.ComponentClass<IProps & Partial<IDefaultProps>> =
    class extends React.Component<IProps & IDefaultProps, {}> {
        static defaultProps: IDefaultProps = {
            age: 25
        }
        render() {
            const { age, name } = this.props;
            return (
                <>
                    <span>名字：{name}</span>
                    <span>年龄：{age}</span>
                </>
            )
        }
    }

export default CompWithDefault;

// use
import CompWithDefault from 'compWithDefault'
<CompWithDefault name="Yehoon"/>
```