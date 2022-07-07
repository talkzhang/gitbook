## github error: fatal: Authentication failed for 'https://github.com/xxx/xxx.git/'

这个错搜出来的结果，一大堆千篇一律的，让你设置什么user.name和user.email，也不知道他们解决没有，反正基本都是这个答案，反正我是没解决，最后还是通过该[答案](https://ginnyfahs.medium.com/github-error-authentication-failed-from-command-line-3a545bfd0ca8)解决的该问题，分享如下。

大致流程就是生成一个personal token，在git push时password将该token填写即可。

Settings:

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/20210907181432.png)

Developer Settings:
![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/20210907181632.png)

Personal access tokens:
![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/20210907184518.png)

生成一个token，并确保复制即可。

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/20210907184613.png)

将生成的token填写至该位置：

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/20210907184925.png)

原文链接：[https://ginnyfahs.medium.com/github-error-authentication-failed-from-command-line-3a545bfd0ca8](https://ginnyfahs.medium.com/github-error-authentication-failed-from-command-line-3a545bfd0ca8)