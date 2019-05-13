###列出镜像
- 列出已经下载的镜像，使用`docker image ls`进行查看 如下图
```ssh
[root@host ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              18.04               d131e0fa2585        2 weeks ago         102MB
hello-world         latest              fce289e99eb9        4 months ago        1.84kB
```

列表包含了 `仓库名`、`标签`、`镜像 ID`、`创建时间` 和 所`占用的空间`。

###镜像体积
- `docker image ls` 列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于 Docker 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。
```linux
[root@host ~]#  docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              2                   2                   101.8MB             0B (0%)
Containers          4                   1                   0B                  0B
Local Volumes       0                   0                   0B                  0B
Build Cache         0                   0                   0B                  0B
```
###虚悬镜像
随着官方镜像维护，发布了新版本后，重新 `docker pull xxx` 时，xxx 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了 <none>。除了 docker pull 可能导致这种情况，docker build 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 <none> 的镜像。这类无标签镜像也被称为 虚悬镜像(dangling image) ，可以用下面的命令专门显示这类镜像：
- `docker image ls -f dangling=true` 显示虚悬镜像
- `docker image prune` 删除虚悬镜像
###中间层镜像
- `docker image ls -a` `docker image ls`只能显示顶层的镜像 显示所有的是后边加参数-a

###罗列部分镜像
- `docker image ls ubuntu` => 根据仓库名**ubuntu**列出镜像
- `docker image ls ubuntu:18.04` 指定仓库名和标签
- `docker image ls -f since=mongo:3.2` -f filter 过滤器来筛选查找

###特定格式来显示
`docker image ls`会输出一个完成的表格，但是有时候没必要显示这么多，可能只需要镜像的ID就够用了，这时候使用 `-p` 参数
```
[root@host ~]# docker image ls -q
d131e0fa2585
fce289e99eb9
```
`--filter`  配合 `-q` 产生出指定范围的 ID 列表，然后送给另一个 docker 命令作为参数，从而针对这组实体成批的进行某种操作的做法在 Docker 命令行使用过程中非常常见，不仅仅是镜像，将来我们会在各个命令中看到这类搭配以完成很强大的功能。因此每次在文档看到过滤器后，可以多注意一下它们的用法。

另外一些时候，我们可能只是对表格的结构不满意，希望自己组织列；或者不希望有标题，这样方便其它程序解析结果等，这时候就需要[GO模板语法](https://gohugo.io/templates/go-templates/)
```
[root@host ~]# docker image ls --format "{{.ID}}: {{.Repository}}"
d131e0fa2585: ubuntu
fce289e99eb9: hello-world
```
表格 展示都可以自定义的通过模板`{{}}`来处理 类似vue中的模板语法
```
[root@host ~]# docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
IMAGE ID            REPOSITORY          TAG
d131e0fa2585        ubuntu              18.04
fce289e99eb9        hello-world         latest
```

###删除镜像
如果要删除本地镜像可以使用`docker image rm`命令：
```
docker image rm [选项] <镜像> [<镜像2> ...]
```
###用镜像名、ID、摘要来删除镜像
`<镜像>`可以是 镜像的短 `ID`、镜像长`ID`、`镜像摘要`或者镜像名字
```
[root@host ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              18.04               d131e0fa2585        2 weeks ago         102MB


docker image rm d13 
```
可以用镜像的完整 `ID`，来删除镜像。更多的时候是用 短 `ID` 来删除镜像。docker image ls 默认列出的就已经是短 `ID` 了，一般取前3个字符以上，只要可以区分于别的镜像就可。

当然，更精确的是使用 `镜像摘要` 删除镜像。
`docker image ls --digests`查看摘要
`docker image rm [摘要的值]`

###使用 docker image ls 命令来配合
可以使用 `docker image ls -q` 来配合使用 `docker image rm`，这样可以成批的删除希望删除的镜像。我们在“镜像列表”章节介绍过很多过滤镜像列表的方式都可以拿过来使用。

删除所有在 `mongo:3.4` 之前的镜像：
```
docker image rm $(docker image ls -q -f before=mongo:3.4)
```
删除所有仓库名为 `redis` 的镜像
```
docker image rm $(docker image ls -q redis)
```