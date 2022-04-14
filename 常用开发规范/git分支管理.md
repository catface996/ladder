# Git分支管理

* 每个maven工程都有 master; dev; daily; pre; gray;分支,分别对应 prod,dev,daily,pre和gray环境.
* dev,daily,pre,gray分支可以被删除,从master拉取最新的分支.
* 特性分支(feature/xxxx)需从master拉取.
* 缺陷修复分支(bugfix/xxxx)需要从master拉取.
* dev; daily; pre; gray 只允merge,不允许push,  特性分支(feature/xxx)或者缺陷修复分支(bugfix/xxxx) 作何合并的源分支,dev; daily等作为合并的目标分支;
* 多个feature或者bugfix分支合并到dev; daily等分支产生冲突时,  在dev,daily 等分支上解决冲突,不在特性分支或者缺陷分支上解决,避免合并dev滞后的代码提前发布到master.
* feature或者bugfix合并到master产生冲突时,需要将master合并到feature或者bufix分支解决冲突,冲突解决后,回归验证特性分支或者缺陷分支,再合并到master做线上发布.
* dev; daily; pre; gray 分支不能作为源分支创建新分支;不能作为源分支merge到其他分支.

分支push,merge流线图如下:

![20220414104643](https://picgo.catface996.com/picgo20220414104643.png)

