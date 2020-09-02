## git remote

添加远程分支：

```
git remote add <name> <url>
```



查看当前远程仓库状态：

```
$ git remote -v
```



当我们fork远程仓库时，默认的远程分支是origin，其指向我们fork之后的仓库中，此时我们push，fetch等操作都是与fork之后的库关联的。

如果我们想直接推送到源库，可先做下关联：

```bash
$ git remote add main <主仓库git地址>
```

这样的话我们就可以对该main仓库进行操作了。

```bash
$ git fetch main #拉取主仓库的代码
$ git rebase main/master #rebase主仓库代码
```

移除远程分支：

```
git remote remove <name>
```

修改远程分支url：

```
git remote set-url <name> <url>
```

## git commit

commit规范：



定义commit规范





## git submodule



## git reset

