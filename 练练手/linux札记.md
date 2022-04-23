# 常用技巧



## 手动释放缓存

~~~shell
To free pagecache, use
echo 1 > /proc/sys/vm/drop_caches

to free dentries and inodes, use
echo 2 > /proc/sys/vm/drop_caches

to free pagecache, dentries and inodes, use
echo 3 >/proc/sys/vm/drop_caches
~~~

~~~shell
[root@iZbp11tgamktkihhgge5mmZ vm]# free -m
              total        used        free      shared  buff/cache   available
Mem:           7551        6597         167           0         786         677
Swap:             0           0           0
## used = free + buffer + cached
~~~







# 参考

[手动释放linux缓存](https://blog.csdn.net/u013673976/article/details/53378592)

