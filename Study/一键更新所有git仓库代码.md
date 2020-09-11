# 一键更新所有git仓库代码

```shell
#! /usr/bin/env bash
find . -maxdepth 3 -name .git -type d | rev | cut -c 6- | rev | xargs -I {} git -C {} pull
```

解释: 该代码能从当前目录往子目录递归查找, 最多三层, 发现所有.git 目录也就是git仓库的根路径. 然后通过裁剪, 得到所有git仓库的路径. 逐个进行`git pull`. 适合所有git收藏控~