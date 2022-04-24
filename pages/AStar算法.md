# 曼哈顿距离

欧式几何距离的计算公式:

![img](https://tva1.sinaimg.cn/large/008i3skNly1gvb7v7hh6wj607j014t8i02.jpg)



曼哈顿距离计算公式:

![img](https://tva1.sinaimg.cn/large/008i3skNly1gvb7vvkhdpj606d00qq2p02.jpg)



欧式距离和曼哈顿距离举例:



![图1 曼哈顿距离](https://tva1.sinaimg.cn/large/008i3skNly1gvb7wvkrkcj607v07v74e02.jpg)
# 图
## 邻接表

![在这里插入图片描述](https://tva1.sinaimg.cn/large/008i3skNly1gvb84sbu6wj61c20lewhp02.jpg)



![img](https://tva1.sinaimg.cn/large/008i3skNly1gvb8875xwbj61bc0nc42w02.jpg)

![img](https://tva1.sinaimg.cn/large/008i3skNly1gvb88ofu4vj61aq0nk0ww02.jpg)

![img](https://tva1.sinaimg.cn/large/008i3skNly1gvb897obc2j61b00pigq002.jpg)
## 邻接矩阵

无向图:

![在这里插入图片描述](https://tva1.sinaimg.cn/large/008i3skNly1gvb8dx0x0tj60q10cqaao02.jpg)

有向图:

![在这里插入图片描述](https://tva1.sinaimg.cn/large/008i3skNly1gvb8e5byrhj60rg0egdgn02.jpg)
# 广度优先
# 深度优先
# 最小生成树
## K算法
## P算法
# 最佳优先搜索
# Dijkstra 迪杰斯特拉
# A * 算法

```text
* 初始化open_set和close_set；
* 将起点加入open_set中，并设置优先级为0（优先级最高）；
* 如果open_set不为空，则从open_set中选取优先级最高的节点n：
  * 如果节点n为终点，则：
      * 从终点开始逐步追踪parent节点，一直达到起点；
      * 返回找到的结果路径，算法结束；
  * 如果节点n不是终点，则：
      * 将节点n从open_set中删除，并加入close_set中；
      * 遍历节点n所有的邻近节点：
          * 如果邻近节点m在close_set中，则：
              * 计算节点m的优先级,如果新得到的优先级比原有的优先级低,更新
          * 如果邻近节点m也不在open_set中，则：
              * 设置节点m的parent为节点n
              * 计算节点m的优先级
              * 将节点m加入open_set中
```
# Dijkstra 与 A *
# 参考

https://zhuanlan.zhihu.com/p/54510444

https://zhuanlan.zhihu.com/p/104051892?utm_medium=social