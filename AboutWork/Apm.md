# apm
Apm在查询数据时，对于哪个变量确定用户唯一性非常模糊，但是一直都没关注，每次需要确定用户唯一性的时候都会卡主，值得反思。
1. 觉得自己能搞定，很熟悉，错误:<全面的熟悉，而不是仅是一个查数据的产品或运营>
2. 觉得不重要，不影响问题定位。错误:`80%的人能解决80％的问题，强者解决剩下的20％`。
总而言之: 不是"可以"就可以了，要做到卓越，这不正是你追求的吗？
- 结论: 
u_id为设备id 即浏览器中BrowserUtil.getDeviceId()方法。
uniqueueId 是三方关联的realate_id 