---
title: Android Compose 中的状态
date: 2022-07-24 11:10
tags:  
    - Android
    - Jetpack
    - Compose
---

## 引言
在声明式 UI 框架中，Widget 或 View 这些描述界面的基本组件有两个定义: Stateful (有状态)、Stateless (无状态)。
接下来我们结合实际项目中的一个小示例聊一聊 Stateful 和 Stateless 是什么，以及为何需要这样设计。
<!--more-->
## Stateful & Stateless 组件
- **State**  任何在 App 运行时存在于内存中的东西都可以简单粗暴的理解为状态，因为我们本次研究的主要对象是 Compose UI 框架，所以在这里我们暂时只考虑那些会影响 UI 变化的值，例如 Text 的 **text**、Image 的 **drawble**、Checkbox 的 **checked**。
> 总结: **状态是小组件创建时可以同步读取的信息，在小组件的生命周期内可能会发生变化。**
> 
> 换句话说，小组件的状态是它的属性（参数）在创建时（小组件被画在屏幕上时）所维持的对象的数据。当它被使用时，状态也可以改变，例如，当一个 CheckBox 小组件被点击时，CheckBox 会根据其 **checked** 属性变更复选框的状态。
- **Stateless 组件**  一旦创建，其状态就无法更改的组件，因为它依赖于外界传入的参数来绘制界面，并且**无法在其内部自行变更状态**，最常见的如 Text、Image 等组件。

不包含任何逻辑计算的 Stateless 组件示例:
```kotlin
@Composable
fun StatelessBottomTextIconButton(
    modifier: Modifier = Modifier,
    icon: ImageVector,
    text: String,
    onClick: () -> Unit,
) {
    IconButton(
        modifier = modifier,
        onClick = onClick
    ) {
        Column(
            horizontalAlignment = Alignment.CenterHorizontally,
        ) {
            Icon(
                imageVector = imageVector,
                modifier = Modifier.size(20.dp),
                contentDescription = "Button"
            )
            Text(
                text = text,
                style = MaterialTheme.typography.caption.copy(fontSize = 10.sp)
            )
        }
    }
}
```
**StatelessBottomTextIconButton** 的 Icon 和 text 在创建时被传入后直接赋予组件且不可变更。

- **Statefu 组件l** 有状态的组件，即创建后无需外界再次传入状态(参数)也**能在内部自行修改状态的组件**，例如 Checkbox、Radio。
接下来，我们尝试将 Stateless 示例中的 **StatelessBottomTextIconButton** 包装成 **Stateful** 组件来供不同的 UI 业务场景使用。

## 组件的状态提取
我们希望每次点击下载按钮，Icon 下方的数字都能 +1 以统计下载次数。点击赞按钮时，如果是点赞则 +1 否则 -1，点赞数是 0 时显示 "赞"。

![stateless item.mov.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e80d71d3940e4905a90185a3e479953b~tplv-k3u1fbpfcp-watermark.image?)

这个状态显然不符合需求，我们需要把 **StatelessBottomTextIconButton** 包装成 Stateful 组件。

有状态的 **RuleDownloadButton** :
```kotlin
@Composable
fun RuleDownloadButton(
    modifier: Modifier = Modifier,
    rule: Rule,
    onClick: () -> Unit,
) {
    // 从 rule 中获取初始下载数量并通过 rememberSaveable 包装为 Compose 可记住的状态
    var download by rememberSaveable { mutableStateOf(rule.download) }
    // 默认值 "下载"
    val defaultText = stringResource(R.string.download)
    // 通过 download 计算出 text 的值
    val text by derivedStateOf {
        if (download == 0) defaultText else download.toString()
    }

    StatelessBottomTextIconButton(
        modifier = modifier,
        icon = Icons.Outlined.Download,
        text = text,
    ) {
        download++
        onClick()
    }
}
```
可以看到，我们新建了一个 RuleDownloadButton 并在其内部新增了 download、text 属性，并通过 rememberSaveable 让 Compose 可以在重组过程中记住此值，然后将这些属性传递给 StatelessBottomTextIconButton 做为初始值，在 StatelessBottomTextIconButton 的点击事件下调用 **download++**  来修改下载数，同时 **text** 也会随着 download 被修改重新计算出新的值，这样就达到了**在组件内部自行修改状态**的目的。

我们继续封装一个 **RuleLikeButton**:
```kotlin
@Composable
fun RuleLikeButton(
    modifier: Modifier = Modifier,
    rule: Rule,
    onClick: () -> Unit,
) {
    // 点赞状态 Boolean
    var likes by rememberSaveable { mutableStateOf(rule.likes) }
    // 点赞数量 Int
    var star by rememberSaveable { mutableStateOf(rule.star) }
    val defaultText = stringResource(R.string.thumbup)
    val text by derivedStateOf {
        if (star == 0) defaultText else star.toString()
    }
    val icon by derivedStateOf {
        if (likes) Icons.Filled.ThumbUpAlt else Icons.Filled.ThumbUpOffAlt
    }

    StatelessBottomTextIconButton(
        modifier = modifier,
        icon = icon,
        text = text,
    ) {
        likes = likes.not()
        if (likes) star++ else star--
        onClick()
    }
}
```
点赞按钮也很简单，只需要让 Compose 记住点赞状态（true/false）和点赞数量即可，text 可以通过点赞数量计算出来，icon 也可以通过点赞状态获取。然后我们只需在 StatelessBottomTextIconButton 的点击事件中更新点赞状态和点赞数量即可。

状态的修改会触发 Compose 树进行重组，接着 StatelessBottomTextIconButton 就会被重新 new 出来并根据新的状态进行界面渲染。
通过这样的包装，我们就做到了 Stateless 组件到 Stateful 组件的转变:

![stateful item.mov.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/411c8db97838418c9c34a594ae00e914~tplv-k3u1fbpfcp-watermark.image?)

## 总结
不难看出，在实际应用中我们用到的绝大部分组件都是 Stateful 的，需要实时接收用户的事件输入并更新组件状态。

那么为什么要有 Stateful 和 Stateless 这两种设计呢? 

正如上面的示例中的应用场景，我们用到的很多组件虽然都是 Stateful 的，但是基本形状和操作逻辑都有很多重复的部分，比如都是 Icon + Text、都可以响应点击事件等等。
这时，提取出一个 Stateless 组件就非常必要了，之后我们就可以根据不同的业务逻辑去自定义各种 Stateful 组件，比如通过之前的例子，你很容易就能封装出分享、收藏按钮。