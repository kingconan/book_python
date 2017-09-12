公司的代码在github，jenkins在阿里云 打包速度太慢了，有时完全连接不上

在美国机房买vps 搭建shadowsocks

在阿里云机房设置客户端

sslocal -s &lt;美国IP&gt; -p &lt;美国端口&gt; -k &lt;密码&gt; -m &lt;加密方式&gt; -l &lt;本地端口&gt;

在git config全局配置代理端口

git config --global http.proxy socks5://127.0.0.1:1080

git config --global https.proxy socks5://127.0.0.1:1080

取消git config配置：

git config --global http.proxy ""

git config --global https.proxy ""



作者：MichaelD0ng

链接：http://www.jianshu.com/p/b1f6e6944f94

來源：简书

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

