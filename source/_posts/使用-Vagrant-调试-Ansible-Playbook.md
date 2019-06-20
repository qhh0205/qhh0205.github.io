---
title: 使用 Vagrant 调试 Ansible Playbook
date: 2019-06-07 15:09:09
categories: Ansible
tags: Ansible
---

 ![Alt text](/images/vagrant-vbox-ansible.png)

#### 简介
本文主要介绍使用 Vagrant 本地调试 Ansible Playbook 的最佳实践。

我平时用 ansible 做一些自动化任务，难免要写很多 playbook，如果直接将写的 playbook 在线上或者真实的服务器运行难免会担心出错，而且很可能会导致严重的错误。最好的方法就是先在本地虚拟机测试好，然后跑到真实的环境。我们可以将 Vagrant 和 ansible 结合使用来轻松地在本地调试 playbook。为什么使用这种方式呢？我觉得有如下好处（当然用了之后就知道有多爽了）：
- 虚拟机用 Vagrant 管理，随时可以方便地删除、重建，这些操作都是简单的命令行；
- ansible 脚本在本地虚拟机可以随便折腾，哪怕 VM 折腾坏了，可以马上重建 VM；
- 所有操作都是基于配置文件，没有界面点触式操作，可以很好地将其工程化，放到 git 仓库统一管理；

### Vagrant 结合 Ansible 的 workflow
Vagrant 结合 Ansible 的主要工作原理是使用 Vagrant 的 [ansible_local](https://www.vagrantup.com/docs/provisioning/ansible_local.html) 或者 [ansible](https://www.vagrantup.com/docs/provisioning/ansible.html) 配置器(Provisioner)，这两个的唯一区别是前者会在 provision 时自动在 VM 安装 ansible，后者不会自动安装，需要自行安装。我选择用 ansible_local 配置器，懒得装一遍 ansible... 而且在 `vagrant destroy` 销毁虚拟机后重建时还能得到与之前一致的配置。

下面介绍下我本地调试 ansible playbook 脚本的 workflow。
1. 使用 Vagrantfile 定义虚拟机
```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "Centos7"
  config.vm.hostname= "ansible"
  config.vm.network "private_network", ip: "192.168.10.100"
  config.vm.provider :virtualbox do |vbox|
      vbox.name = "ansible_vagrant"
      vbox.memory = "512"
      vbox.cpus = 1
  end
  # ansible 相关配置
  config.vm.provision "ansible_local" do |ansible|
    # ansible 运行时输出详细信息，作用同 ansible-playbook 的 -v 参数
    # ansible.verbose = "v"
    
    # 指定运行哪个 playbook
    ansible.playbook = "playbook.yml"
  end
end
```
2. 在 Vagrant 工程目录下编写 playbook，示例：
```yaml
---
- name: Test Playbook...
  hosts: all
  become: yes
  gather_facts: no
  vars:
    - ip_list:
      - 127.0.0.1
      - 127.0.0.1
      - 127.0.0.1
      - 127.0.0.2
  tasks:
    - name: remove ip list file
      file:
        state: absent
        path: /tmp/ip.txt
    - name: test gen file
      shell: echo "{{ item }}" >> /tmp/ip.txt
      with_items: "{{ ip_list }}"
```
3. playbook 写完后 执行 `vagrant up` 启动虚拟机，启动过程中会自动执行 Vagrantfile 中配置的 playbook 文件（在 Vagrant 工程目录下）；
4. 如果 playbook 运行有问题，则继续修改；
5. 执行 `vagrant rsync && vagrant provision` 重新运行 playbook；
   >注意：在执行 vagrant provision 之前，一定要先 vagrant rsync 同步下本地主机和 VM 的共享目录，否则本地修改后不会生效。ansible 脚本的运行最终是在 VM 上面，读取的文件都是在 VM 的 /vagrant 目录。
6. 如果测试 playbook 还是有问题则返回到第 4 步继续修改并测试；
7. 如果一切都测试顺利的话，为了保险，最后模拟一下真实的环境：`vagrant destroy` 销毁 VM，然后 `vagrant up` 重新创建 VM 并自动运行 ansible playbook。

### 相关资料
https://linux.cn/article-9502-1.html | 使用 Vagrant 测试 Ansible 剧本
https://www.vagrantup.com/docs/provisioning/ansible_intro.html | Vagrant 和 Ansible
https://www.vagrantup.com/docs/provisioning/ansible_local.html | Vagrant ansible_local 配置器
https://www.vagrantup.com/docs/provisioning/ansible_common.html | Vagrant ansible 和 ansible_local 配置器通用配置

