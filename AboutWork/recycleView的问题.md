#### Kotlin写法导致的问题

kotLin代码写法一定要注意，由于kotLin强大的判空方式
data?.let{
    ...
}
导致忘记写else 如果是ListView或RecycleView中刷新数据会出现问题。

#### RecycleView#notifyItemChanged
调用RecycleView#notifyItemChanged方法，刷新未生效，不知什么问题，简单使用 adapter.notifyDataSetChanged()解决了，又一次容忍了模糊 ┭┮﹏┭┮ 

问题又一次出现了，似乎adapter.notifyDataSetChanged()也不一定能解决问题（测试复现而开发未复现）

### 背景
首页是一个大的RecycleView  里面有许多不同的type 
```
        addItemProvider(
            HomeFragmentBannerProvider(
                Banner.type,
                R.layout.provider_home_banner
            )
        )

        addItemProvider(
            HomeFragmentSecKillProvider(
                SecKill.type,
                R.layout.layout_seckill_mall
            )
        )

```
不同于单层的recycleView，这里的item是以类别划分的。
在某些类型内部还存在recycleView，当内部recycleView的item刷新数据时，使用了notifyItemChanged(index)但是并未生效  
于是排查了下代码，得出如下结论

notifyItemChanged(index)会重新创建一个viewHolder，并同时使用新旧viewHolder，原因如下：
RecycleView使用这两个viewHolder来实现平滑的新旧切换状态，这是RecycleView.ItemAnimator的默认行为。当然你可以禁用动画
```
listView.setItemAnimator(null);
```

为了避免ViewHolder在调用recyclerView.notifyItemChanged()后重新创建并且不禁用平滑动画，您可以设置DefaultItemAnimator并覆盖canReuseUpdatedViewHolder()方法。
```
val itemAnimator: DefaultItemAnimator = object : DefaultItemAnimator() {
      override fun canReuseUpdatedViewHolder(viewHolder: RecyclerView.ViewHolder): Boolean {
            return true
      }
}
recyclerView.itemAnimator = itemAnimator
```
未亲测验证过哈

我遇到的问题是
使用notifyItemChanged(index) 也不能说完全未生效，产生了奇怪的效果，item延迟展示了刷新
review了代码  发现在RecycleView嵌套RecycleView中，使用了判空
```
if(recyclerView == null) {
   recyclerView = itemView.findViewById(R.id.rl_discount)
}

```
且view的初始化在onCreateViewHolder中
会导致的问题是：如只是复用的viewHolder 那当然没问题，只有一个recycleView
当出现复用场景时，这里就有问题，导致视图不可见。
这里大可不必，recycleView自身的复用机制保证了不会重复创建view。
- ！！！ 所以将view的初始化放在convert中 亲测与 onViewAttachedToWindow方法一致
- ！！！ 在convert中可以通过数据的更新来判断是否正常刷新数据 
具体示例如下

```
	override fun convert(holder: BaseViewHolder, item: HomeFragmentBean) {
		if (item.obj == null || item.obj !is HomeSecKill) {
			return
		}
		if (item.obj === secKillBean) { //判断是否需要更新data
			return
		}
		initView(holder.itemView) 
		secKillBean = item.obj as HomeSecKill
		adapter?.setList(secKillBean?.goodsList)
		updateTitleView(secKillBean)
		initCountTime(secKillBean)
	}

```


### 边距的问题 
Recycle通过addItemDecoration的方式可以设置上下左右的边距
当有更负责的场景时，可重写getItemOffsets方法，来自定义边距，瀑布流的效果，通过spanIndex来判断瀑布流的左右列  具体代码如下
```
        recycleView.addItemDecoration(object : RecyclerView.ItemDecoration() {
            override fun getItemOffsets(
                outRect: Rect,
                view: View,
                parent: RecyclerView,
                state: RecyclerView.State
            ) {
                val spanIndex =
                    (view.layoutParams as StaggeredGridLayoutManager.LayoutParams).spanIndex
                if (spanIndex == 1) { // 瀑布流右边的列
                    outRect.right = ConvertUtils.dp2px(12f)
                }
            }
        })

```
可以参考的文章 https://cloud.tencent.com/developer/ask/sof/38077