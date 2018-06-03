# 优秀的React/Redux应用目录结构 #

译自:[A Better File Structure for React/Redux Applications](https://marmelab.com/blog/2015/12/17/react-directory-structure.html)

我能找到的大多数React/Redux应用(包括客户端和服务器端)例子都是非常简单的。他们按照分类去安排文件夹(actions, component, container, reducer)。文件的组织方式看起来就像下面这样
```
actions/
    CommandActions.js
    UserActions.js
components/
    Header.js
    Sidebar.js
    Command.js
    CommandList.js
    CommandItem.js
    CommandHelper.js
    User.js
    UserProfile.js
    UserAvatar.js
containers/
    App.js
    Command.js
    User.js
reducers/
    index.js
    command.js
    user.js
routes.js
```

[Redux book](https://redux.js.org/advanced/example-reddit-api)就是遵循的这种组织方式，并且据我所知，至少有两个Redux样板仓库也是如此。[3tree](https://github.com/GordyD/3ree), [react-redux-universal-hot-example](https://github.com/erikras/react-redux-universal-hot-example)

这种方式虽然很好，但是在我想增加新的领域需求(包括actions, components, reducer)的时候，会发生什么呢？比如说，如果我想增加products相关的目录，我需要在每一个文件夹中增加文件,像下边这样:
```
actions/
    CommandActions.js
    ProductActions.js   <= Here
    UserActions.js
components/
    Header.js
    Sidebar.js
    Command.js
    CommandList.js
    CommandItem.js
    CommandHelper.js
    Product.js          <= Here
    ProductList.js      <= Here
    ProductItem.js      <= Here
    ProductImage.js     <= Here
    User.js
    UserProfile.js
    UserAvatar.js
containers/
    App.js
    Command.js
    Product.js          <= Here
    User.js
reducers/
    index.js
    foo.js
    bar.js
    product.js          <= Here
routes.js
```
你可以看出发生了什么。从两个月前到现在，components文件夹已经包含了一大堆的文件，并且当我需要处理一个特定的功能时，我需要在4个不同的文件夹中打开4个不同的文件.

所以，我们为什么不按照领域去给文件分组呢？为了区分actions, components, reducers我可以使用不同的名字后缀:
```
app/
    Header.js
    Sidebar.js
    App.js
    reducers.js
    routes.js
command/
    Command.js
    CommandContainer.js
    CommandActions.js
    CommandList.js
    CommandItem.js
    CommandHelper.js
    commandReducer.js
product/
    Product.js
    ProductContainer.js
    ProductActions.js
    ProductList.js
    ProductItem.js
    ProductImage.js
    productReducer.js
user/
    User.js
    UserContainer.js
    UserActions.js
    UserProfile.js
    UserAvatar.js
    userReducer.js
```
通过合并container和其相关组件,可以提高可读性.
Redux使得containers(连接指state的容器)和component(无状态组件)有所区分.大多数教程都是通过以下方式来表现的:
```
// in Product.js
export default function Product({ name, description }) {
    return <div>
        <h1>{ name }</h1>
        <div className="description">
            {description}
        </div>
    </div>
}

// in ProductContainer.js
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import * as ProductActions from './ProductActions';
import Product from './Product';

function mapStateToProps(state) {
    return {...state};
}

function mapDispatchToProps(dispatch) {
    return bindActionCreators({
        ...ProductActions,
    }, dispatch);
}

export default connect(mapStateToProps, mapDispatchToProps)(Product);
```

驱使我们将容器组件和无状态组件分开的原因是，我们可以很方便的对无状态组件进行单元测试.在99%的情况中，component是不会在container外部使用的.而且es6允许我们在一个文件中暴露多个元素.所以我们可以将上述的两个文件合并成一个，通过default的形式暴露container,通过属性的形式暴露component。就像下面这样:
```
// in Product.js
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import * as ProductActions from './ProductActions';

// component part
export function Product({ name, description }) {
    return <div>
        <h1>{ name }</h1>
        <div className="description">
            {description}
        </div>
    </div>
}

// container part
function mapStateToProps(state) {
    return {...state};
}

function mapDispatchToProps(dispatch) {
    return bindActionCreators({
        ...ProductActions,
    }, dispatch);
}

export default connect(mapStateToProps, mapDispatchToProps)(Product);
```

通过这种方式，单元测试的时候，只需要import { Product } from './Product.js'. 现在每个文件夹就可以少放置一个文件了:
```
app/
    Header.js
    Sidebar.js
    App.js
    reducers.js
    routes.js
command/
    Command.js         // component & container
    CommandActions.js
    CommandList.js
    CommandItem.js
    CommandHelper.js
    commandReducer.js
product/
    Product.js         // component & container
    ProductActions.js
    ProductList.js
    ProductItem.js
    ProductImage.js
    productReducer.js
user/
    User.js            // component & container
    UserActions.js
    UserProfile.js
    UserAvatar.js
    userReducer.js
```
之前提到了测试，他们通常有自己的目录，和运行时的代码分离开:
```
src/
    app/
        Header.js
        Sidebar.js
        App.js
        reducers.js
        routes.js
    command/
        Command.js
        CommandActions.js
        CommandList.js
        CommandItem.js
        CommandHelper.js
        commandReducer.js
    product/
        Product.js
        ProductActions.js
        ProductList.js
        ProductItem.js
        ProductImage.js
        productReducer.js
    user/
        User.js
        UserActions.js
        UserProfile.js
        UserAvatar.js
        userReducer.js
test/
    app/
        Header.js
        Sidebar.js
        App.js
        reducers.js
        routes.js
    command/
        Command.js
        CommandActions.js
        CommandList.js
        CommandItem.js
        CommandHelper.js
        commandReducer.js
    product/
        Product.js
        ProductActions.js
        ProductList.js
        ProductItem.js
        ProductImage.js
        productReducer.js
    user/
        User.js
        UserActions.js
        UserProfile.js
        UserAvatar.js
        userReducer.js
```
我发现如果这样的话，将很难发现缺少测试的组件，或者在领域扩展的时候浏览文件结构。
所以我尝试将测试文件和要测试的文件放在同一个目录中，并且加上-spec.js后缀。如果在Python中，测试文件都会被要求和要测试的文件拥有一样的名字。
现在所有同一上下文的文件都被放在了同一个目录里，这样有助于阅读和推理。
```
src/
    app/
        Header.js
        Header-spec.js
        Sidebar.js
        Sidebar-spec.js
        App.js
        App-spec.js
        reducers.js
        reducers-spec.js
        routes.js
        routes-spec.js
    command/
        Command.js
        Commands-spec.js
        CommandActions.js
        CommandActions-spec.js
        CommandList.js
        CommandList-spec.js
        CommandItem.js
        CommandItem-spec.js
        CommandHelper.js
        CommandHelper-spec.js
        commandReducer.js
        commandReducer-spec.js
    product/
        Product.js
        Product-spec.js
        ProductActions.js
        ProductActions-spec.js
        ProductList.js
        ProductList-spec.js
        ProductItem.js
        ProductItem-spec.js
        ProductImage.js
        ProductImage-spec.js
        productReducer.js
        productReducer-spec.js
    user/
        User.js
        User-spec.js
        UserActions.js
        UserActions-spec.js
        UserProfile.js
        UserProfile-spec.js
        UserAvatar.js
        UserAvatar-spec.js
        userReducer.js
        userReducer-spec.js
```
配置测试工具(Jest或者Mocha)也很简单：更改一下配置参数 ./src/**/*-spec.js

这种文件结构在项目逐渐扩大的情况下，有很好的可扩展性。而且当 需要讲部分代码拆分为独立的项目的时候，重构会是很轻量级的。我强烈推荐这种组织方式!