关于如何配置charles，如何安装手机端与mac端证书的方法，网上有一大堆，不再赘述，这里有个系统的教程，可自行了解。
`https://www.axihe.com/tools/charles/charles/tutorial.html` 

我遇到的问题是：已按照规范配置，且安装了证书，但是host与ip的映射依然失败。
host与ip的映射，我是这样配置的，很熟悉，对不对？

于是我得到了这样的错误



我仔细观察了charles的配置项，在Tools下方有DNS Setting / DNS Spoofing
与Map Remote 二者的描述看起来差不多，又有些差距。二者的具体差别请查看这里
`https://www.charlesproxy.com/documentation/tools/map-remote/`
`https://www.charlesproxy.com/documentation/tools/dns-spoofing/`


通过对DNS协议的了解，加上DNS的描述，它是能够解决host到ip映射的问题的，于是我果断取消了Map Remote的配置，将映射关系添加到了DNS Spoofing中，
果然有效果，且同事的charles如法炮制配置后，也ok了。
后来查看了上述文章，发现自己真的是只是运气好而已。

- 总结
综上文章的描述，ip与host的映射，就交由DNS Setting / DNS Spoofing
Map Remote 可以用来做http到https的映射，或者实现本地重定向。
