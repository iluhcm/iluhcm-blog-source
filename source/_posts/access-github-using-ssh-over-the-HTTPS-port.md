title: 通过SSH-over-HTTPS访问Github
date: 2015-12-29 20:11:01
tags: [Github,SSH,Https]
categories: Course

---

最近博客一直没有更新，原因之一就是去实习了，然后在公司内网推送代码到远程Github分支的时候总是报如下错误：

```
ssh: connect to host github.com port 22: Operation timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: ssh: connect to host github.com port 22: Operation timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

    at ChildProcess.<anonymous> (/Users/xingli/Workspace/Hexo/node_modules/hexo-deployer-git/node_modules/hexo-util/lib/spawn.js:42:17)
    at emitTwo (events.js:87:13)
    at ChildProcess.emit (events.js:172:7)
    at maybeClose (internal/child_process.js:818:16)
    at Socket.<anonymous> (internal/child_process.js:319:11)
    at emitOne (events.js:77:13)
    at Socket.emit (events.js:169:7)
    at Pipe._onclose (net.js:469:12)
```

尝试把Github上的SSH keys全部删除并重新添加，结果并没有什么*用，还是报同样的错误，但奇怪的是同样的`SSH key`推送到公司的Gitlab上完全没有问题，并且通过`git`下载Github上的源码并没有任何问题。我猜测是公司为了避免把内部代码开源把`git`协议的上传端口给封了，结果一测果然如此。执行

	ssh -T git@github.com
报超时错误。

	ssh: connect to host github.com port 22: Operation timed out
使用代理上传也遇到同样的问题。作为懒人，`git`协议不能正常在Github使用简直让人抓狂，又不想使用http协议来推送（每次都要输入密码，且不安全）。网上搜索了一番，在[Github官网](https://help.github.com/articles/using-ssh-over-the-https-port/)找到了一种办法，通过https协议建立SSH连接，即让SSH走443端口。经过测试，公司的防火墙并没有封443端口（基本不可能封）。测试代码如下：

	ssh -T -p 443 git@ssh.github.com
如果执行命令后不报错并且显示

	Hi userName! You've successfully authenticated, but GitHub does not provide shell access.
那接下来就好办了。编辑`~/.ssh/config`文件（没有则创建一个），然后补充下面的代码：

```
Host github.com
Hostname ssh.github.com
Port 443
```
代码较简单就不解释了，保存退出后再使用上面的命令来测试：

	ssh -T git@github.com

由于我的博客是同时托管在Github和Gitcafe的，更新的时候发现公司只封了Github的端口，而Gitcafe可以正常使用^_^`