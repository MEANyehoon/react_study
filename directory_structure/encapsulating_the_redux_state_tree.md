# Redux状态树封装 #

在一个Redux应用中，应用的大部分数据存储在一个称之为“状态树”的中心位置。状态树的形状和结构对应用程序的开发和性能有很大的影响。随着时间的推移，通过重构状态树来解决问题，常常是很有价值的。我们如何的安全的进行重构呢？

我之前写过很多[关于Redux的文章](http://randycoulman.com/blog/categories/redux/).如果需要相关的介绍或者复习其中的内容，可以进行查阅。

当我们需要安全地改变数据结构时，最常用的工具是封装，也称为数据隐藏或信息隐藏。封装基本理论就是，外部不允许直接访问数据结构，而是通过数据结构提供的统一接口来获取数据。

这就是本文要介绍的内容。

## 我们是如何使用状态树的? ##
为了对状态树进行封装，我们需要先看看应用是如何使用状态树的。对于大多数数据结构来讲，有两种使用方式：读和写。

对于Redux应用也是如此。因为Redux是[Flux架构](https://facebook.github.io/flux/docs/overview.html)的一种实现，他遵循[单向数据流](https://redux.js.org/basics/data-flow)。这样更容易进行推导。

## 写 ##
我们从写这个角度入手，因为它是Redxu中最显式的一部分内容。

状态树是由处理各种actions的reducer来修改的。数据不是直接被更改，而是reducer返回一个被修改过后的新的state树。Redux Store会记住这个新的状态树。

## 读 ##
在大多数的Redux应用中，读取状态树的地方往往很分散。每一个实现了```mapStateToProps```方法的的```container component```(容器组件)，它从state tree中读取数据，然后通过props注入到它包裹的组件中。

一些不太明显的用法是，在要被thunk中间件处理的action creators中使用。Thunk actions中的一个参数是store的```getState```，通过它，可以在创建action的时候从state树中获取数据

第三个地方是在reducer的测试中。它们通常需要知道reducer正确的处理了一个action.

## 现在怎么办？ ##
现在我们已经知道了，状态树是如何被使用的。那么我们可以对它做什么呢？

首先，我们可以发现，在写的方面，reducer和action creators已经为我们做了一层封装。应用不允许直接的修改状态树，取而代之的是应用发出action，reducer接受后去更改状态树。在这一方面，我们没有什么可以作为的。

其次,读的方面并没有被很好的定义。读的逻辑被分散在了容器组件中，thunk actions中，还有测试中。当状态的形态发生更改时，我们很难去更新他们的结构。

一个新兴的方案就是引入一个抽象层，称之为**selector**(选择器).

## 选择器 ##
选择器是一个接受state和其他可选参数，返回一些数据的函数.

```selector:: (state, ...args) => data```

作为在程序的各个地方使用```state.todos```的替代，我们定义一个返回所有todos的选择器```allTodos(state)```。这看起来像是一个无意义的行为，但是这对封装状态树至关重要。

通过使用选择器，程序的其他部分不需要知道，也不需要关心todos在状态树中是如何存储的，这就使我们重构状态树变的相对容易。

选择器可以直接返回状态树中的数据，也可以在返回之前进行一些计算。

Redux官方文档在核心概念介绍中并没有过多的介绍选择器，仅仅在计算派生数据的章节提及了一下，还有一个很好的库**reselect**.

## 我们达成目标了么？ ##
通过actions和reducer在写方面的封装，还有选择器在读方面的封装，我们达成目标了么？现在重构状态树变得安全了么？

大多数情况下，是的。如果我们所有的代码都是用actions和selector，并且从不直接访问状态树，那么当我们需要重构状态树的时候，我们就确切的知道要去修改那些部分。

Dan Abramov甚至建议将selector和reducer放在一个文件中，这样会使封装更清晰。我没有尝试，但我绝对明白它背后的道理.

那么测试呢？测试中要使用选择器么？

## 测试 ##
早些时候我写过一篇[测试Redux应用](http://randycoulman.com/blog/2016/03/15/testing-redux-applications/)。在测试reducer部分中，我展示了下面的测试，它是从Redux官方文档中修改而来的：
```
import { expect } from 'chai'
import reducer from 'reducers/todos'
import ActionTypes from 'constants/ActionTypes'
 
describe('todos reducer', () => {
  it('initializes the state', () => {
    expect(reducer(undefined, {})).toEqual([])
  })
 
  it('adds a todo', () => {
    expect(reducer([], addTodo('Run the tests'))).toEqual([
      {
        text: 'Run the tests',
        completed: false,
        id: 0
      }
    ])
  })
})
```
请注意，在这个测试中有几个地方是知道状态树的形态的。在所有与状态树形态耦合的情况中，这种情况是最容易避免的。但是我们能做得更好吗？

我在某种程度上优化了我的reducer测试策略，在我目前的项目中，我正在尝试测试与状态树形态完全解耦。

我在测试的开始部分通过```const```定义了一个通过reducer生成的初始状态。
```
describe('todos reducer', () => {
  const initialState = reducer(undefined, {})
 
  # ...
})
```

如果有什么重要的初始状态，我会使用reducer写一些测试去测试它。我要确保从使用状态树的角度去测试，而不是从reducer的实现角度去测试。
```
describe('initial state', () => {
    it('starts with no todos', () => {
        expect(allTodos(initialState)).to.be.empty  
    })
})
```

请注意，这个测试并没有说明todos是如何存储的，也没有说明状态树的确切形态。它只是检查了一些重要的可观察属性。

为了测试对于各种actions的处理，我写了像下面一样的测试：
```
describe('adding a todo', () => {
    const state = initialState
    const action = addTodo('TODO')
    const newState = reducer(state, action)
    const addedTodo = allTodos(newState)[0]

    it('includes the todo in the list', () => {
        expect(addedTodo).to.exist
    })

    it('remembers the name', () => {
        expect(addedTodo.name).to.eq('TODO')
    })

    it('assigns the next available id', () => {
        expect(addedTodo.id).to.eq(0)
    })

    it('marks the todo as initially incomplete', () => {
        expect(addedTodo.complete).to.be.false
    })
})
```

这种书写方式是有点冗长的。在一个测试中检查添加一个todo后的所有属性是很好的。我倾向于使用单独的```it```块，因为这些描述很好的表达了我的意图，并明确了reducer的指责。

我倾向于使用```const state = ...; const action = ...; const newState = ...;```这种模式。这种规则的结构可以提高可读性。

如果我需要一个初始状态意外以外的状态来书写测试，我尝试着使用actions来获得其他状态。

```
const state = reducer(initialState, addTodo('Pre-existing'))
```

如果我需要一个以上的action来获得测试起始状态。我会用reduce处理actions的数组。这里我们使用了**Ramda**版本的reduce，当然还有很多其他好的选择。

```
const state = reduce(reducer, initialState, [
    addTodo('Already complete'),
    completeTodo(0),
    addTodo('Still outstanding')  
])
```

未完待续...
