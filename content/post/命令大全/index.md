---
title: 常用命令小全
date: 2025-03-04
slug: common-commands
categories:
  - Tool
description: Linux、Git、Docker等常用的工具的常用命令
draft: false
lastmod: 2025-03-14
---
## Linux 常用命令

##### 文件与目录操作

- **`pwd`**：显示当前工作目录
```bash
pwd  # 输出：/home/user
```

- **`ls`**：列出目录内容
```bash
ls -l  # 详细列表（权限、大小等）[1,2](@ref)
ls -a  # 显示隐藏文件
```

- **`cd`**：切换目录
```bash
cd /var/log  # 绝对路径切换
cd ..        # 返回上级目录
```

- **`mkdir`/`touch`**：创建目录/文件
```bash
mkdir -p a/b/c  # 递归创建多层目录
touch test.txt  # 创建空文件
```

- **`cp`/`mv`/`rm`**：复制/移动/删除
```bash
cp -r dir1 dir2  # 递归复制目录
mv old.txt new.txt  # 重命名
rm -rf dir  # 强制递归删除（慎用）
```

##### 系统监控与管理

- **`top`/`htop`**：实时资源监控
```bash
top     # 查看CPU/内存占用
htop    # 增强版（需安装）
```

- **`ps`/`kill`**：进程管理
```bash
ps aux | grep nginx  # 过滤进程
kill -9 1234        # 强制终止进程
```

- **`df`/`free`**：磁盘与内存
```bash
df -h    # 查看磁盘空间（易读格式）
free -m  # 以MB为单位显示内存
```

##### 文本处理与网络工具

- **`grep`/`awk`/`sed`**：文本三剑客
```bash
grep "error" log.txt  # 搜索关键字
awk '{print $1}' data.txt  # 提取第一列
sed 's/old/new/g' file.txt  # 全局替换
```

- **`curl`/`wget`**：网络请求
```bash
curl -O https://example.com/file.zip  # 下载文件
wget https://example.com/image.jpg
```

- **`ping`/`netstat`**：网络诊断
```bash
ping google.com        # 测试连通性
netstat -tulnp        # 查看监听端口
```

#### Git 常用命令

##### 仓库与提交

- **`init`/`clone`**：初始化/克隆仓库
```bash
git init              # 初始化本地仓库
git clone https://github.com/user/repo.git
```

- **`add`/`commit`**：提交变更
```bash
git add .            # 添加所有修改
git commit -m "fix: update config"
```

- **`status`/`diff`**：查看状态与差异
```bash
git status           # 显示变更文件
git diff HEAD~1      # 对比前一次提交
```
- **`tag`**：打标签
```shell
git tag - a 1.0.0 -m "1.0.0"
git push origin 1.0.0
```

##### 分支与合并

- **`branch`/`checkout`**：分支管理
```bash
git branch feature   # 创建新分支
git checkout main    # 切换分支
```

- **`merge`/`rebase`**：合并代码
```bash
git merge feature    # 合并分支到当前分支
git pull --rebase    # 变基式拉取更新
```

##### 远程与推送

- **`push`/`pull`**：同步远程仓库
```bash
git push origin main  # 推送本地提交
git pull              # 拉取远程更新
```

- **`remote`/`fetch`**：管理远程连接
```bash
git remote -v         # 查看远程仓库
git fetch upstream   # 获取上游仓库更新
```

##### 撤销与修复

- **`reset`/`revert`**：版本回退
```bash
git reset --hard HEAD~1  # 回退到上一版本
git revert abc123       # 撤销指定提交
```

- **`stash`**：暂存修改
```bash
git stash          # 临时保存工作区修改
git stash pop      # 恢复暂存内容
```

#### Docker 常用命令

##### 镜像管理

- **`pull`/`images`**：拉取与查看镜像
```bash
docker pull mysql:8.0  # 拉取镜像
docker images            # 列出本地镜像
```

- **`rmi`/`prune`**：清理镜像
```bash
docker rmi nginx        # 删除镜像
docker image prune      # 清理无用镜像
```

- **`run`/`start`/`stop`**：容器生命周期
```shell
docker run -d \
  --name mysql \
  -p 3306:3306 \                # 映射端口到主机
  -v D:/mysql:/var/lib/mysql \  # 挂载宿主机D:/mysql到容器数据目录
  -e MYSQL_ROOT_PASSWORD=your_password \  # 设置root密码
  mysql:8.0

docker start mysql
docker stop mysql
```

- **`exec`/`logs`**：进入容器与查看日志

```bash
docker exec -it mynginx /bin/     # 进入容器终端
docker logs -f mynginx            # 实时查看日志
```

##### 网络与存储

- **`network`**：网络配置
```bash
docker network create mynet  # 创建自定义网络
docker network ls           # 列出所有网络
```

- **`volume`**：数据卷管理
```bash
docker volume create myvol  # 创建数据卷
docker run -v myvol:/data   # 挂载数据卷到容器
```


##### 构建与发布

- **`build`/`push`**：镜像构建与推送
```bash
docker build -t myapp:v1 .  # 构建镜像
docker push myapp:v1        # 推送至仓库
```