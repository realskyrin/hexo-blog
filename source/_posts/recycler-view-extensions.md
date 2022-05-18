---
title: RecyclerView 各种常用方法拓展
date: 2022-04-24 21:18
tags:  
    - Android
    - View
---
一些常用的 RecyclerView 拓展方法整理
<!--more-->

```kotlin
/**
 * 跳转到顶部，无动画
 */
fun RecyclerView.jump2Top() {
    if (this.adapter?.itemCount ?: 0 > 0) {
        jump2Position(position = 0)
    }
}

/**
 * 滚动到顶部
 */
fun RecyclerView.smoothScroll2Top() {
    if (this.adapter?.itemCount ?: 0 > 0) {
        this.smoothScrollToPosition(0)
    }
}

/**
 * 跳转到 [position]，无动画
 * [onScrolled] 滚动过程回调 dx dy 分别为水平和垂直方向的滚动距离
 * [onScrollDone] 滚动结束回调
 */
fun RecyclerView.jump2Position(
    position: Int,
    onScrolled: ((dx: Int, dy: Int) -> Unit)? = null,
    onScrollDone: ((listener: OnScrollListener) -> Unit)? = null
) {
    layoutManager?.scrollToPosition(position)
    val listener = object : RecyclerView.OnScrollListener() {
        override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
            onScrolled?.invoke(dx, dy)
        }

        override fun onScrollStateChanged(recyclerView: RecyclerView, newState: Int) {
            super.onScrollStateChanged(recyclerView, newState)
            if (newState == SCROLL_STATE_IDLE) {
                onScrollDone?.invoke(this)
            }
        }
    }
    addOnScrollListener(listener)
}


/**
 * 跳转到顶部并以滚动方式结束
 *
 * 用于数据量较多的情况下，在短时间内滚动到指定位置，原理是先跳转再滚动
 */
fun RecyclerView.smoothJump2Top() {
    if (this.adapter?.itemCount ?: 0 > 0) {
        smoothJump2Position(0)
    }
}

/**
 * 跳转到 [position] 并以滚动方式结束
 *
 * 用于数据量较多的情况下，在短时间内滚动到指定位置，原理是先跳转再滚动
 */
fun RecyclerView.smoothJump2Position(position: Int) {
    val itemCount = this.adapter?.itemCount ?: 0

    if (position > itemCount) {
        return
    }

    if (currentPosition() == position) {
        return
    }

    if (itemCount < 20 || (currentPosition() - position).absoluteValue < 20) {
        smoothScrollToPosition(position)
        return
    }

    if (position > currentPosition()) {
        scrollToPosition(position - 19)
    } else {
        scrollToPosition(position + 19)
    }

    smoothScrollToPosition(position)
}

/**
 * 跳转到 [position] 并以滚动方式结束
 *
 * 用于数据量较多的情况下，在短时间内滚动到指定位置，原理是先跳转再滚动
 * [onScrolled] 滚动过程回调 dx dy 分别为水平和垂直方向的滚动距离
 * [onScrollDone] 滚动结束回调
 */
fun RecyclerView.smoothJump2PositionWithListener(
    position: Int,
    onScrolled: ((dx: Int, dy: Int) -> Unit)? = null,
    onScrollDone: ((listener: OnScrollListener) -> Unit)? = null
) {
    val itemCount = this.adapter?.itemCount ?: 0

    if (position > itemCount) {
        return
    }

    if (currentPosition() == position) {
        return
    }

    if (itemCount < 20 || (currentPosition() - position).absoluteValue < 20) {
        smoothScroll2Position(position, onScrolled, onScrollDone)
        return
    }

    if (position > currentPosition()) {
        scrollToPosition(position - 19)
    } else {
        scrollToPosition(position + 19)
    }

    smoothScroll2Position(position, onScrolled, onScrollDone)
}

/**
 * 滚动到 [position]
 * [onScrolled] 滚动过程回调 dx dy 分别为水平和垂直方向的滚动距离
 * [onScrollDone] 滚动结束回调
 */
fun RecyclerView.smoothScroll2Position(
    position: Int,
    onScrolled: ((dx: Int, dy: Int) -> Unit)? = null,
    onScrollDone: ((listener: OnScrollListener) -> Unit)? = null
) {
    layoutManager?.smoothScrollToPosition(this, null, position)
    val listener = object : RecyclerView.OnScrollListener() {
        override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
            onScrolled?.invoke(dx, dy)
        }

        override fun onScrollStateChanged(recyclerView: RecyclerView, newState: Int) {
            super.onScrollStateChanged(recyclerView, newState)
            if (newState == SCROLL_STATE_IDLE) {
                onScrollDone?.invoke(this)
            }
        }
    }
    addOnScrollListener(listener)
}

/**
 * 当前第一个可见 item 的位置
 */
fun RecyclerView.currentPosition(): Int {
    val layoutManager = this.layoutManager
    if (layoutManager is LinearLayoutManager) {
        return layoutManager.findFirstVisibleItemPosition()
    }
    return 0
}

/**
 * RecyclerView 常用的滚动监听
 * [onTop] 到顶部时触发
 * [onBottom] 到底部时触发
 * [onIdle] 滚动暂停时触发
 * [onDragging] 拖拽时触发
 * [onSettling] 惯性滚动时触发
 * [scroll] 滚动过的距离并回传
 */
fun RecyclerView.onScroll(
    onTop: (() -> Unit)? = null,
    onBottom: (() -> Unit)? = null,
    onIdle: (() -> Unit)? = null,
    onDragging: (() -> Unit)? = null,
    onSettling: (() -> Unit)? = null,
    scroll: (Int) -> Unit
) {
    addOnScrollListener(object : RecyclerView.OnScrollListener() {
        override fun onScrollStateChanged(recyclerView: RecyclerView, newState: Int) {
            super.onScrollStateChanged(recyclerView, newState)
            when (newState) {
                // 暂停
                SCROLL_STATE_IDLE -> onIdle?.invoke()
                // 拖动
                SCROLL_STATE_DRAGGING -> onDragging?.invoke()
                // 惯性滑动
                SCROLL_STATE_SETTLING -> onSettling?.invoke()
            }
        }

        override fun onScrolled(
            recyclerView: RecyclerView,
            dx: Int,
            dy: Int,
        ) {
            super.onScrolled(recyclerView, dx, dy)
            scroll(computeVerticalScrollOffset())

            if (recyclerView.canScrollVertically(-1).not()) {
                // 滑动到顶部
                onTop?.invoke()
            }
            if (recyclerView.canScrollVertically(1).not()) {
                // 滑动到底部
                onBottom?.invoke()
            }
        }
    })
}
```