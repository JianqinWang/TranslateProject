在 Xfce 会话中保存窗口的位置
======
摘要：如果你发现 Xfce session 不能保存窗口的位置，那么启用 `save on logout` 然后登出再进来一次，可能就能修复这个问题了 (permanently， if you like keeping the same session and turn saving back off again)。 下面是详细内容。

我用 Xfce 作桌面有些年头了，但是每次重启后进入之前保存的 session 时总会有问题出现。 登陆后， 之前 session 中保存的应用都会启动， 但是所有的工作区和窗口位置数据会丢失， 导致所有应用都堆在默认工作区中，乱糟糟的。

多年来，很多人都报道过这个问题( Ubuntu， Xfce， 以及 Red Hat 的 bug 追踪系统中都有登记这个 bug)。 虽然 Xfce4.10 中已经修复过了一个相关 bug， 但是我用的 Xfce4.12 依然有这个问题。 如果不是我的其中一个系统能够正常的回复各个窗口的位置，我几乎都要放弃找出问题的原因了(事实上我之前已经放弃过很多次了)。 

今天，我深入对比了两个系统的不同点，最终解决了这个问题。 我现在就把结果写出来， 以防有人也遇到相同的问题。

提前的一些说明：

  1。由于这个笔记本只有我在用，因此我几乎不登出我的 Xfce session。 我一般只是休眠然后唤醒，除非由于要对内核打补丁才进行重启， 或者由于某些改动损毁了休眠镜像导致系统从休眠中唤醒时卡住了而不得不重启。 另外，我也很少使用 Xfce 工具栏上的重启按钮重启; 一般我只是运行一下 `reboot`。

  2。我会使用 xterm 和 Emacs， 这些 X 应用写的不是很好，无法记住他们自己的窗口位置。

Xfce 将 sessions 信息保存到主用户目录中的 `.cache/sessions` 目录中。在经过仔细检查后发现，在正常的系统中有两类文件存储在该目录中，而在非正常的系统中，只有一类文件存在该目录下。

其中一类文件的名字类似 `xfce4-session-hostname:0` 这样的，其中包含的内容类似下面这样的：
```
Client9_ClientId=2a654109b-e4d0-40e4-a910-e58717faa80b
Client9_Hostname=local/hostname
Client9_CloneCommand=xterm
Client9_RestartCommand=xterm，-xtsessionID，2a654109b-e4d0-40e4-a910-e58717faa80b
Client9_Program=xterm
Client9_UserId=user

```

这个文件记录了所有正在运行的程序。如果你进入 Settings -> Session and Startup 并清除 session 缓存， 就会删掉这种文件。 当你保存当前 session 时， 又会创建这种文件。 这就是 Xfce 知道要启动哪些应用的原因。 但是请注意，上面并没有包含任何窗口位置的信息。 (我还曾经以为可以根据 session ID 来找到其他地方的一些相关信息，但是失败了)。

正常工作的系统在目录中还有另一类文件，名字类似 `xfwm4-2d4c9d4cb-5f6b-41b4-b9d7-5cf7ac3d7e49.state` 这样的。 其中文件内容类似下面这样：
```
[CLIENT] 0x200000f
 [CLIENT_ID] 2a9e5b8ed-1851-4c11-82cf-e51710dcf733
 [CLIENT_LEADER] 0x200000f
 [RES_NAME] xterm
 [RES_CLASS] XTerm
 [WM_NAME] xterm
 [WM_COMMAND] (1) "xterm"
 [GEOMETRY] (860，35，817，1042)
 [GEOMETRY-MAXIMIZED] (860，35，817，1042)
 [SCREEN] 0
 [DESK] 2
 [FLAGS] 0x0

```

注意这里的 geometry 和 desk 记录的正是我们想要的窗口位置以及工作区号。因此不能保存窗口位置的原因就是因为缺少这个文件。

继续深入下去，我发现当你明确地手工保存 sessino 时，智慧保存第一个文件而不会保存第二个文件。 但是当登出保存 session 时则会保存第二个文件。 因此， 我进入 Settings -> Session and Startup 中，在 Genral 标签页中启用登出时自动保存 session， 然后登出后再进来， 然后 tada， 第二个文件出现了。 再然后我又关闭了登出时自动保存 session。( 因为我一般在排好屏幕后就保存一个 session， 但是我不希望做出的改变也会影响到这个保存的 session， 如有必要我会明确地手工进行保存)， 现在 我的窗口位置能够正常的回复了。

这也解释了为什么有的人会有问题而有的人没有问题： 有的人可能一直都是用登出按钮重启，而有些人则是手工重启(或者仅仅是由于系统漰溃了才重启)。

顺带一提，这类问题， 以及为解决问题而付出的努力正是我赞同为软件存储的状态文件编写 man 页或其他类似文档的原因。 为用户编写文档，不仅能帮助别人深入挖掘产生奇怪问题的原因， 也能让软件作者注意到软件中那些奇怪的东西， 比如将 session 状态存储到两个独立的文件中去。


--------------------------------------------------------------------------------

via: https://www.eyrie.org/~eagle/journal/2017-12/001.html

作者：[J. R. R. Tolkien][a]
译者：[lujun9972](https://github.com/lujun9972)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]:https://www.eyrie.org
