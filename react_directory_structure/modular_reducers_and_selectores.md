# Modular Reducers And Selectors #
译自:[Modular Reducers and Selectors](http://randycoulman.com/blog/2016/09/27/modular-reducers-and-selectors/)


>不久前，我们讨论过[Redux状态树](http://randycoulman.com/blog/2016/09/13/encapsulating-the-redux-state-tree/)。在[上一讲](http://randycoulman.com/blog/2016/09/20/redux-reducer-selector-asymmetry/)中,我们研究了在使用combineReducers时，reducers和selectors产生的不对称性。我们当时提出了一种在Rais-style项目中运行良好的解决方案。但是如果我们想要在modular(domain-style)的项目组织中使用，我们就会遇到问题.

## 什么是 Modular(模块化) 项目结构 ##
[Redux FAQ]()中描述了各种目录结构组织方式的区别:

>* Rails-style: “actions”、“constants”、“reducers”、“containers” 以及 “components” 分属不同的文件夹
>* Domain-style:为每个功能或者域创建单独的文件夹，可能会为某些文件类型创建子文件夹
>* “Ducks”：类似于 Domain-style，但是明确地将 action、 reducer 绑定在一起，通常将它们定义在同一文件内。

在这一讲中，我们讲Domain-style和"Ducks"认为是一种风格，因为在reducers和selectors方面，他们没有任何区别.我通常将这种风格称为"modular模块化", 因为他们将应用拆分为单独的模块.

在Redux FAQ中有很多链接，关于模块化组织结构的说明中，我最喜欢的是Jack Hsu的[Rules For Structuring (Redux) Applications](https://jaysoo.ca/2016/02/28/organizing-redux-application/).

在那篇文章中，Jack给出了关于组织项目的三条规则:

1. 根据特性管理：应用中每一个领域或者特性拥有自己的模块或者文件夹。相关的actions, reducer, components, selectors都在这个模块中。
2. 创建严格的模块边界：每个模块定义统一的明确的对外接口。在该模块的顶层目录中的index.js文件将仅暴露出允许其他模块使用的内容。模块绝对不允许使用"深层引用"从其他模块引用内容(比如：```import Component from 'modules/foo/components/Component'```)。如果不能通过```import { Component } from 'modules`/foo'```引用，那么久说明这个component不允许在其他模块使用。
3. 避免循环引用：如果模块A依赖了模块B(A通过B的公共接口引用)，那么模块B就不能引用模块A的任何内容。这就是[之前提到的](http://randycoulman.com/blog/2014/02/04/packaging-principles-part-2/)鲍勃大叔说的[无环依赖原则(ADP)](https://en.wikipedia.org/wiki/Acyclic_dependencies_principle)

## 目前为止我们有什么? ##