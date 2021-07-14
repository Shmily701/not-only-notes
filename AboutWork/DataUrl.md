
`bug编号1928928` 

1.访问网址：https://m.liantu.com/
2.点击保存图片，查看文件名

由于内核无法传递mimeType，且url无法获取类型、名称，导致猜名失败。
通过chrome调试发现,图片资源: href = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAASw..."

`bug背景` 内核传递文件类型是一件非常麻烦的事情,所以我们对此资源类别获取的唯一途径，
就是通过href的内容进行解析来获取类别。
于是根据href前缀`data:image`迅速的解决了此问题  
即将提交代码,对data:image/png;base64.... 进行了判断,但此方式仅仅解决了这个问题。

- Error!!!
如果你不了解它，如何使用它？ 会不会还有其他类型`data：image/jpg` `data：text/...`

思考问题一: 它是谁?

这个类型是否还有其他类型？怎么能做到通用性？
终于了解到Data URLs，庆幸自己手不快，代码还没提，要不又要贻笑大方了。
原则：解决问题的原因，而不是现象。

Data URLs 内容协议[Data URLs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs)

允许内容创建者在文档中嵌入小文件 16M以下

- `Syntax` `data:[<mediatype>][;base64],<data>` 
The mediatype is a MIME type string, such as 'image/jpeg'... , If omitted, defaults to text/plain;charset=US-ASCII

- 长度限制 
建议 浏览器将 URL 限制为 65535 个字符,dataURL限制为 65529 个字符（65529 个字符是编码数据的长度，而不是源，如果您使用普通data:，而不指定 MIME 类型）

- 安全问题
许多安全问题（例如网络钓鱼）与数据 URL 相关，并在浏览器的顶层导航到它们。

根据Data URLs 规则,解析代码如下

```
    /**
     * parse data Urls
     * syntas data:[<mediatype>][;base64],<data>
     * @param url
     * @param mimeType
     * @return
     */

    private static String parseDataUrlMimeType(String url, String mimeType){
        try {
            if (TextUtils.isEmpty(url)) {
                return mimeType;
            }
            if (!URLUtil.isDataUrl(url)) {
                return mimeType;
            }
            // mimeType不为空,则以其为准
            if (!TextUtils.isEmpty(mimeType)) {
                return mimeType;
            }
            int commaIndex = url.indexOf(',');
            if (commaIndex < 0) {
                return mimeType;
            }

            // 默认text/plain
            mimeType = "text/plain";
            int colonIndex = url.indexOf(":");
            int semicolonIndex = url.indexOf(';');
            int subIndex = 0;
            if (semicolonIndex >= 0) {
                subIndex = Math.min(commaIndex,semicolonIndex);
            }
            else {
                subIndex = commaIndex;
            }

            String type = "";
            if (subIndex >= colonIndex+1) {
                type = url.substring(colonIndex+1, subIndex);
                if (!TextUtils.isEmpty(type)) {
                    mimeType = type;
                }
            }
            return mimeType;
        } catch (Throwable e) {
            e.printStackTrace();
            CrashHandler.getInstance().saveExceptionAsCrash(e);
        }
        return mimeType;
    }

    ```
    根据其格式信息，正确解析其资源的类别信息。

    思考问题二
    如何下载这样的资源？
    通过二进制流的方式写文件，并以某种方式打开？TODO 可尝试