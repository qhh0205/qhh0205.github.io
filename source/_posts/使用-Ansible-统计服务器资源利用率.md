---
title: 使用 Ansible 统计服务器资源利用率
date: 2019-01-10 23:30:52
categories: Ansible
tags: Ansible
---

分享一个 ansible playbook，统计服务器 CPU、内存、磁盘利用率，3 条 shell 脚本实现统计:

CPU 利用率统计：
```bash
top -bn1 | grep load | awk '{printf "CPU Load: %.2f\n", $(NF-2)}'
```
内存利用率统计：
```bash
free -m | awk 'NR==2{printf "Memory Usage: %s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }'
```
磁盘利用率统计（列出每块磁盘利用率）：
```bash
df -h -t ext2 -t ext4 | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print "Disk Usage:"" " $1 " " $3"/"$2" ""("$5")"}'
```

Ansible playbook: `server-cpu-mem-disk-usage.yml`
```yaml
---
- name: Statistics CPU Memory Disk Utilization
  hosts: "{{ hosts }}"
  become: no
  remote_user: "{{ user }}"
  gather_facts: no
  tasks:
    - name: "Statistics CPU Memory Disk Utilization..."
      shell: |
        free -m | awk 'NR==2{printf "Memory Usage: %s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }'
        df -h -t ext2 -t ext4 | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print "Disk Usage:"" " $1 " " $3"/"$2" ""("$5")"}'
        top -bn1 | grep load | awk '{printf "CPU Load: %.2f\n", $(NF-2)}'
      register: out
    - debug: var=out.stdout_lines
```
输出结果样例：
```json
ok: [gke-test-standard-pool] => {
    "out.stdout_lines": [
        "Memory Usage: 8766/16052MB (54.61%)",
        "Disk Usage: /dev/root 449M/1.2G (37%)",
        "Disk Usage: /dev/sda8 28K/12M (1%)",
        "Disk Usage: /dev/sda1 61G/95G (64%)",
        "CPU Load: 0.92"
    ]
}
```

