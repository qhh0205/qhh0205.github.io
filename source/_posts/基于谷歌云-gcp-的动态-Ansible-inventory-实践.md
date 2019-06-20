---
title: 基于谷歌云 gcp 的动态 Ansible inventory 实践
date: 2019-01-09 22:58:55
categories: Ansible
tags: Ansible
---

### 关于 Ansible inventory 说明
ansible inventory 文件可以分为如下两类：
1. 静态 inventory：主机信息写死到文件，这种情况一般适用于管理少量主机，对于成百上千规模的主机人工管理主机清单文件显然是不合理的；
2. 动态 inventory：ansible 根据脚本动态获取云提供商的主机清单文件，这样可以省去人工维护静态清单文件的繁琐工作，对于大批量主机管理也是非常可靠的；

### Ansible 动态获取云提供商主机 inventory 原理
ansible 通过 `-i` 参数指定动态 inventory 目录，该目录底下放置获取云提供商主机清单的脚本（ansible 社区提供的一般是 Python 脚本），ansible 在执行时该脚本会自动执行并将结果保存到内存中。

那么上面说的获取云提供商主机清单的可执行脚本在哪里获取呢？在 [这里](https://github.com/ansible/ansible/tree/devel/contrib/inventory) （ansible 官方源码仓库：社区提供的脚本）获取，这里有各个云提供商对应的主机清单脚本(\*.py)及配置文件(\*.ini)，比如谷歌的 `gce.py` 和 `gce.ini`，Aws 的 `ec2.py` 和 `ec2.ini` 等等。

### 基于 gcp 的动态 inventory 使用
下面是配置使用谷歌云动态 ansible inventory 的详细步骤

1. 相关软件包安装;
```bash
pip install apache-libcloud pycrypto
```

2. 谷歌云控制台创建一个服务账号（需要有 gce 的访问权限），获取 json 认证文件;
3. 从 [ansible 官方仓库](https://github.com/ansible/ansible/tree/devel/contrib/inventory) 下载 `gce.py` 和 `gce.ini` 文件;
```bash
mkdir -p inventories/gcp-dynamic-inventory
cd inventories/gcp-dynamic-inventory
wget https://github.com/ansible/ansible/blob/devel/contrib/inventory/gce.py
wget https://github.com/ansible/ansible/blob/devel/contrib/inventory/gce.ini
```

4. 编辑 `gce.ini` 配置文件
```bash
[gce]
libcloud_secrets =
gce_service_account_email_address = <服务账号邮箱：在第 2 步的 json 认证文件里面可以找到>
gce_service_account_pem_file_path = <第 2 步中 json 认证文件路径：绝对路径>
gce_project_id = <gcp 项目 id>
gce_zone =
```

5. 测试配置的正确性
```bash
# 如果输出一个很长的 json 串表示没问题
./gce.py --list
```

6. 执行 ansible 任务
```bash
ansible -i inventories/gcp-dynamic-inventory <pattern> -m <module_name> -a 'module_args'

Ex:
  # 查看 asia-east1-a 区域的所有主机时间
  ansible -i inventories/gcp-dynamic-inventory asia-east1-a -m shell -a 'date'
```
	**参数说明：**
	`inventories/gcp-dynamic-inventory`:  gce.py 脚本所在的目录，ansible 运行时会自动在该目录下执行该脚本获取主机清单；

	`pattern`：./gce.py --list 执行结果的 json 顶级节点都可以作为 ansible 的目标主机；

	最佳实践：可以给 gce 主机添加 tag，然后通过 tag 对主机分组；

7. `gce.ini` 文件位置
`gce.ini` 文件没必要必须和 `gce.py` 在一个目录，可以设置环境变量放到系统其他目录，这样就可以将配置和脚本分离，避免敏感配置放到代码仓库。设置方法：`~/.bashrc` 文件添加如下内容:
	```bash
	[[ -s "$HOME/.ansible/gce.ini" ]] && export GCE_INI_PATH="$HOME/.ansible/gce.ini"
	```

### Tips
  ansible 执行时可以通过 `--list-host` 参数先测试下本次操作影响到哪些主机，不会真正执行 task；

### 参考文档
https://temikus.net/ansible-gcp-dynamic-inventory-bootstrap

https://medium.com/vimeo-engineering-blog/orchestrating-gce-instances-with-ansible-d825a33793cd

