   (1923560) : develop_6.4.0.103247_无网状态下，本地视频无法播放m3u8格式视频 线上问题
   此bug包含的几个问题
1. m3u8文件下载完成,切片文件为.image后缀的文件,在描述文件中地址依然为网络地址 https://p1.pstatp.com/img/tos-cn-i-0000/ca03815f720940a792869c6b9ba30870~tplv-obj.image
并非切片文件的相对地址
参考竞品，结合浏览器的m3u8描述文件内容，发现，在视频下载过程中，m3u8描述文件的切片地址，为网络地址(切片文件的下载地址)
当视频下载完成时，m3u8描述文件中切片地址更新为下载目录的相对地址了。
排查m3u8下载完成的代码
客户端修复: 描述文件中地址改为,为本地相对地址(参考ts文件的方式)

修复完成后,使用本地播放器却无法播放
 排查描述文件与切片文件，正常，怀疑是我们播放前的问题。
使用QQ浏览器可正常播放,(QQ浏览器支持.ts与.image后缀的播放) 可见下载描述文件的修改正确。

判断或为本地播放器播放.image文件的问题
做以下尝试: 客户端将.image后缀改为.ts
本地播放器,在线离线都可以正常播放

结论:
由于在线场景下，image能够播放，所以
客户端先不做后缀名修改,若播放器无法支持,客户端可以修改

2. m3u8文件下载完成
查看已下载完成的m3u8描述文件
为正常的相对地址,无网络地址,但离线场景下,本地播放器无法播放. @内核排查
此网站里的目前都是
http://www.179u.com/mps/39545-1-7.html
m3u8下载地址
https://vod6.wenshibaowenbei.com/20210425/9riVxkg7/1000kb/hls/index.m3u8

在加密的m3u8文件中,包含密钥url地址
#EXT-X-KEY:METHOD=AES-128,URI="https://ts4.chinalincoln.com:9999/20210409/d1ewdQMX/1000kb/hls/key.key"
若支持离线播放,需将密钥下载到本地,并在m3u8文件中更新密钥地址
更新后如下
#EXT-X-KEY:METHOD=AES-128,URI="key.key"
需内核支持.key离线播放

在验证问题2时,花费大量时间,原因如下:
① 对m3u8视频格式的了解不够深入
    完整格式的 以#EXT-X-ENDLIST 结尾的, 只要列表与ts切片正确对应,则可以正确播放  
    直播格式的无此结尾
    
### ②下载m3u8时间较长,未找到最小验证case  这是首要解决的问题
    不确定是否可以直接修改已下载的m3u8文件,不了解是否有文件校验的步骤,也未曾尝试 经过测试后,发现可以
    测试方法,改错再改对,依然可以正常播放

③通过代码实现了key的下载与m3u8文件更新,播放依然失败,一度以为是路径的问题,却未曾拿竞品测试 思路不够开阔
    也未曾想过是否是播放器的支持度问题 是否支持.key

④竞品无法播放我们的m3u8. 未曾思考过是什么问题
    结论.QQ浏览器在检测到m3u8里的切片文件已下载到本地,再去检测key文件, 发现key文件不存在,也未去检测URI网络地址,再去下载,算是个bug

⑤URI在注释里,简单认为无用, 曾手动删除URI后,无法再次播放,未想过原因,简单认为是不能够随意修改文件,未曾通过改错再改对的方式验证是否可以修改.

## 排查m3u8下载问题,构建最小测试场景方法
根据切片时间长短,考虑下载几个切片文件(.ts),达到能够播放的测试效果即可
如果想要离线播放,需要手动更新m3u8列表文件为相对路径@示例
如果是加密的视频(包含这句#EXT-X-KEY),需下载key密钥文件,并将下载的key目录更新到m3u8列表文件
eg: 更新此句
#EXT-X-KEY:METHOD=AES-128,URI="https://ts4.chinalincoln.com:9999/20210409/d1ewdQMX/1000kb/hls/key.key"
#EXT-X-KEY:METHOD=AES-128,URI="key.key"
用播放器/竞品播放器来验证上述文件的生效性与正确性


### 关于下载m3u8完成后,下载管理目录的视频扫描
浏览器的文件扫描中,并未包含已下载的m3u8文件
原因: 视频扫描是查询系统数据库方式,系统指定的uri为 MediaStore.Video.Media.EXTERNAL_CONTENT_URI `content://media/external/video/media`
FileOperationThread#findVideoList
而系统本身不是不m3u8文件,这一点可在手机文件管理器下确认,打开不以视频的方式

QQ浏览器之所以能够扫描出m3u8文件一定是做了特别处理

基于浏览器的实现方案, 需要以文件的方式来查询
FileOperationThread#findOtherFileList 指定uri为 `content://media/external/file`
通过后缀名的方式筛选 在MimeTypes.OTHER_SELECTION_ARGS中添加 `%.m3u8` MediaStore.Files.FileColumns.DATA 指的是文件路径 用模糊查询like即可 通过此方式,可以筛选出任何想要筛选的文件.



### 关于m3u8下载的问题
- 07-12
fixbug 1936166
m3u8 取tab title

- 07-13
1936166 的修改通过tab title来判断，但这样影响范围太大，有许多不必要的重命名。
问题背景:
m3u8的下载url不是一成不变的，坑！所以不能通过查找插入的url来确定m3u8任务下载的唯一性，所以`使用m3u8文件下载路径来判断`
但对于bug描述中的网址http://www.g5ds.com/goulb/rihan/  
文件名取的是tab 的title 所以同一系列eg"日韩电影" 获得的名称相同，（但实际却是不同的下载资源）。

由此可见，原有的`使用m3u8文件下载路径来判断`是不可靠的
于是只能另辟蹊径，则衍生出新的方案
`通过extra_text来存储原始url` 的方案来确定唯一性，具体修改如下：

- 插入一条数据时 
```
    // m3u8的下载url会变更,无法通过url来确认唯一性,所以添加extra_text4存储原始url
    if (PieceVideoParser.isPlayListFile(filename)) {
        LogUtil.d("DownloadM3u8", url);
        LogUtil.d("DownloadM3u8", filename);
        values.put(Downloads.COLUMN_EXTRA_TEXT4,url);
    }
```

- 查询数据
```
    Cursor cursor = null;
    boolean isM3u8 = PieceVideoParser.isPlayListFile(name);
    try {
        String where = "";
        if (!isM3u8) {
            where = Downloads.COLUMN_URI + " = ? " + " and ("
                    + Downloads.COLUMN_TITLE + " = ? or "
                    + Downloads.COLUMN_FILE_NAME_HINT + " = ? )";
            cursor = context.getContentResolver().query(
                Downloads.CONTENT_URI, null, where, new String[] { url, name, name }, null);
        }
        else {
            where = Downloads.COLUMN_EXTRA_TEXT4 + " = ? "; //COLUMN_EXTRA_TEXT4 此字段浏览器未使用
            cursor = context.getContentResolver().query(
                    Downloads.CONTENT_URI, null, where, new String[]{url}, null);
        }
    } catch (Exception ignored) {}

    boolean urlInDB = (cursor != null && cursor.moveToFirst() && cursor.getCount() > 0);
    if(urlInDB) {
        //can't download and show reDownload hint
    }
    else {
        if (isNeedUpdateM3u8Name(context,isM3u8,name)) {
            name = updateM3u8FileDirName(url,name);
        }
        ...
    }

    private static boolean isNeedUpdateM3u8Name(Context context, boolean isM3u8, String fileName){
        // url不存在，但m3u8切片的存在(某些m3u8名称相同,但url不同)需对此m3u8重命名
        // 示例网址http://www.g5ds.com/goulb/rihan/  对应bug编号:1936166
        return isM3u8 && !TextUtils.isEmpty(readPieceVideoPath(context, fileName));
    }
    // 根据特定规则对m3u8更名，竞品QQ浏览器也是如此处理的

```
- 重新下载
```
values.put(Downloads.COLUMN_EXTRA_TEXT4, c.getString(c.getColumnIndex(Downloads.COLUMN_EXTRA_TEXT4)));

```
- 07-14

跟进bug 1936166 基于上述的修改，存在一个问题。
url不存在，但m3u8切片的存在时，才重命名，存在一个问题:
下载任务已开启，但m3u8下载路径或未完成创建，导致`重命名的前提条件`（m3u8文件目录存在时）失效
技术方案：可通过已入库的名称来判断
产品认为‘下载名称用户不敏感’
经三方沟通，逻辑简化处理为：
只要是m3u8文件的下载，`isM3u8`统一在名称后拼接原始url的haseCode
影响范围：
m3u8文件下载名称

