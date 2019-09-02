---
title: 数据库升级 DevOps 落地实践
date: 2019-08-29 11:01:48
categories: DevOps
tags: DevOps
---

在我们做持续集成/交付的过程中，应用的发布已经通过 DevOps 流水线基本能满足快速迭代的需求，但是很多企业在落地实践 DevOps 的过程中很容易忽略的一点是关于应用数据库版本、升级的管理，每次上线发布数据库的更新依然通过运维或者 DBA 手工更新，在微服务、容器盛行的背景下，服务多，服务发布速度快，显然靠人工该 DB 是跟不上迭代速度的，从而导致 DB 的更新成了整个软件交付周期的瓶颈。这一点我是深有体会，尤其是每次上线时，多个微服务同时上线，同时还需要进行 DB 升级，这个时候研发人员会给我们 SQL 执行，当然研发人员的数量是远远多于开发人员，所以每次上线运维人员经常陷入被”围堵“的尴尬境地。当然还存在其他种种痛点，大概总结下有如下痛点：
- DB 的升级人工执行跟不上版本发布速度，成为软件交付的瓶颈；
- 人工执行 DB 升级错误率比自动化执行更容易出错；
- 各个环境 DB 没有统一的版本管理，经常会出现这个 SQL 有没有在某个环境执行的疑问；
- 环境之间数据脚本同步经常出现遗漏的情况，由于开发或测试环境操作 DB 的人多，在应用从一个环境升级到另一个环境中经常忘记执行某条 SQL，然后导致各种问题故障。这种问题甚至在应用上线时也会频频出现，然后通知运维或 DBA 执行遗漏 SQL；

那么在 DevOps 落地实践中如何很好地处理好数据库升级这一环呢，从而解决上述存在的种种痛点，不要让数据库升级成为软件交付的瓶颈，使得数据库的升级流程融入自动化流水线。我调研了下业界关于应用 DB 升级的方案，不少文章或者圈内人士推崇专门的数据库管理工具版本化管理，自动化执行，比如 [Flyway](https://flywaydb.org/)，[Liquibase](https://www.liquibase.org/) 等著名的工具，都是专业的数据库版本管理和自动化工具。

在本文中主要介绍如何将 Flyway 和其他 DevOps 工具链整合，实现 DB 升级的自动化和管理的版本化，从而解决之前存在的一系列痛点。本文用到的工具链有：Flyway + Jenkins 2.0（Pipeline 脚本）+ Gitlab + MySQL。需要说明的一点是本文并不是一步步讲解各种工具链如何使用和相关介绍，重点在于工具链的整合实践，以及如何恰当地应用。在文末附有完整的 Pipeline 实现脚本，仅供参考！


### 数据库升级 DevOps 实践带来了什么收益
其实在文章开言已经说清楚了，总结起来就两点：
- 所有环境数据库版本统一管理；
- 数据库升级变更自动化；

### 实践方案概要
数据库升级脚本统一按微服务模块以独立 git 仓库的形式管理起来，每次版本迭代，规划好 SQL 模型定义（DDL），将 db 脚本签入独立的 git 仓库，然后使用专门的数据库版本管理工具自动扫描仓库目录的 db 升级脚本，由于 db 升级脚本文件名称符合一定的命名规范，所以工具可以自动按版本号顺序执行脚本，并且已经执行过的脚步文件再次执行会忽略。关于 DB 升级工具的选择，我们选用 Flyway，功能单一、容易上手，以规约优于配置的思想规范 DB 的版本化管理，我们写的 SQL 脚本文件都必须符合 Flyway 的文件名命名规范，这样才能在升级过程中生效。

### 具体实践
借助的工具链：Flyway + Jenkins 2.0（Pipeline 脚本）+ Gitlab + MySQL(Google Cloud SQL)

![](/images/flyway-devops.png)

1. 以微服务应用 git 工程名称在 gitlab 一个单独的组创建 db 代码工程；
2. 在 db 代码工程中创建以数据库命名的目录，存放对应数据库升级的脚本文件，脚本文件名称需要符合 FlywayDB 的命名规范：
![](/images/flyway-file-version.png)
3. db 代码工程分支管理：dev 环境对应 dev 分支，test 环境对应 test 分支，stage 环境对应 stage 分支，生产环境对应 master 分支；
4. Jenkins 脚本注册相应代码工程名称和对应 db 名称；
5. 点击 Jenkins 执行数据库升级；

### 强制规约
1. gitlab 代码工程名称和 db 工程名称一致，db 工程目录下文件夹以数据库名称命名；
2. db 脚本名称符合 FlywayDB 命名规范；
3. db 脚本文件版本名统一大于 1.0，比如: 可以是 V1.0.1，但不能是 V0.2.3；
4. db 脚本内容为 DDL 语句，不能包含 DML 或者 DCL 语句，这个要严格审核，因为 DML 和 DCL 版本追踪没意义，而且各个环境可能还不兼容，FlywayDB 的本质是数据库 Sechma 版本管理，只关心表结构，表里面的数据不关心。关于数据库 DDL、DML、DCL 相关概念及区别见[这里](https://www.cnblogs.com/zhangmingcheng/p/5295684.html)；
5. 已经执行过的 db 脚本不能修改后重复执行，并且执行过的 db 脚本文件需要原封不动保留，不能丢失和修改，否则升级会失败，这个一定要注意。如果对已经执行的 db 脚本不满意，有改动需要变更，则新加 db 脚本文件，可以小版本号比原先增 1，相当于临时 fix，但是我们尽量减少这种情况的发生；

### 具体 Workflow
#### 开发人员 Workflow
开发、测试、预发布环境开发人员点击 Jenkins job 执行数据库升级：
1. 将 SQL 脚本按照 FlywayDB 规范提交到对应的 db 仓库，提交 MR 到对应分支；
2. 小组 db 脚本审核人审核没问题后合并 db 代码；
3. 小组成员点击 Jenkins Job，执行数据库升级
3.1 选择环境+服务名称+要升级的数据库名称
![](/images/flyway-devops-workflow-dev1.jpeg)
3.2 运行 Job
![](/images/flyway-devops-workflow-dev2.jpeg)

#### 运维人员 Workflow
运维人员只负责线上 SQL 的升级：
1. 开发人员告知运维人员本次上线 db 脚本已提交到代码仓库并 merge 到 master 分支；
2. 运维人员点击 Jenkins Job 执行相应服务的数据库升级：
2.1 选择服务名称+要升级的数据库名称
![](/images/flyway-devops-workflow-ops1.jpeg)
2.2 运行 Job，Pipeline 会阻塞在确认节点，做最后的审查
![](/images/flyway-devops-workflow-ops2.jpeg)
2.3 Job 执行完成
![](/images/flyway-devops-workflow-ops3.jpeg)

#### Workflow 举例
1. 新建一个 gitlab 工程，专门存放 db 脚本：
服务名称假设为 db-migration-demo，db 名称为 demo，仓库里面存放的 SQL 脚本如下：
![](/images/flyway-devops-example1.png)

2. git 提交代码，然后点击 Jenkins Job，执行数据库升级
![](/images/flyway-devops-example2.png)

### 关于 Pipeline 设计的两个功能点
#### 1. 数据库整库备份策略
数据库 DDL 变更前整库备份一下是有必要的，但是每次变更都整库备份也不合理，因为可能某天上线，数据库升级比较集中，一天内会触发很多次备份，造成了资源的浪费。解决方案是给备份一个时间窗口（比如 2 小时），每次执行前判断下最近两小时是否有备份，如果没有则触发整库备份，这样就能避免每次执行 Job 都会触发整库备份。
具体解决方法：
获取当前时间减去两小时的时间，然后和上次整库备份的时间戳比较，如果前者大，说明最近两小时内没备份，然后自动触发整库备份，时间戳比较用 Shell 脚本实现：
```bash
# !/bin/bash

t1=`date -d "$1" +%s`
t2=`date -d "$2" +%s`

if [ $t1 -ge $t2 ]; then
    echo "true"
else
    echo "false"
fi
```

#### 2. 每次变更前备份库下的所有表结构，同时记录下 FlywayDB 更改前后状态
表结构备份和 FlywayDB 更改前后状态信息都以制品的方式归到 Jenkins，这样可以随时在 Jenkins 界面查看相关信息，比如查看 Flyway 前后执行状态如何，点开制品页即可看到：
![](/images/flyway-devops-artifact.jpeg)

![](/images/flyway-devops-flywahystate.png)

### 附：Pipeline 脚本实现
为例减小文章的篇幅，这里只贴下运维人员 Workflow 的 Jenkins pipeline 脚本，研发人员的和这个类似，只是一些小的改动。
```
pipeline {
  parameters {
    //服务名称
    choice(name:'serviceName', choices: [
     'db-migration-demo'
     ]
    , description: '服务名称')
    //数据库
    choice(name:'dbName', choices: [
     'demo'
    ]
    , description: '数据库名称')
  }
  agent {
    kubernetes {
      label "sql-${UUID.randomUUID().toString()}"
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: db-imgration
spec:
  containers:
  - name: flyway
    image: boxfuse/flyway
    command:
    - cat
    tty: true
  - name: mysql-client
    image: arey/mysql-client
    command:
    - cat
    tty: true
  - name: gcloud
    image: google/cloud-sdk:alpine
    command:
    - cat
    tty: true
"""
    }
  }
  post {
      failure {
        echo "Database migration failed!"
      }
      success {
        echo "Database migration success!"
      }
  }

  
  options {
      gitLabConnection('gitlab-connection')
      //保持构建的最大个数
      buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  stages {
    stage('初始化') {
        steps {
          script {
            currentBuild.description = "production环境${params.serviceName}服务${params.dbName}库升级..."
          }

          container('gcloud') {
            withCredentials([file(credentialsId: 'cloudInfrastructureAccess', variable: 'cloudSQLCredentials')]) {
              sh "gcloud auth activate-service-account ${env.cloudInfrastructureAccessSA} --key-file=${cloudSQLCredentials} --project=${env.gcpProject}"
            }
          }
          
          // 判断是否要进行数据库备份，如果两小时内没有备份则自动触发全量备份
          script {
            isBackup = 'false'
            // 默认 jenkins 跑在 busybox 容器，获取时间和普通 Linux 发行版有点区别
            date2HoursAgo = sh(returnStdout: true, script: "date -u +'%Y-%m-%d %H' -d@\"\$((`date +%s`-7200))\"").trim()
            container('gcloud') {
              latestDBBackupTime = sh(returnStdout: true, script: "gcloud sql backups list --instance=${env.prodMySqlInstance} --limit=1 | grep -v 'WINDOW_START_TIME' | awk '{print \$2}' | awk -F ':' '{print \$1}'|sed 's/T/ /g'").trim()
            }
            withCredentials([file(credentialsId: 'time-compare.sh', variable: 'timeCompare')]) {
              isBackup = sh(returnStdout: true, script: "sh ${timeCompare} \'$date2HoursAgo\' \'$latestDBBackupTime\'").trim()
              echo "$date2HoursAgo"
              echo "$latestDBBackupTime"
              echo "$isBackup"
            }
          }
        }
    }

    // 如果两小时内没有备份则自动触发全量备份
    stage('整库智能备份') {
        when {
          expression { isBackup == 'true' }
        }
        steps {
          script {
            container('gcloud') {
              // 列出最近 10 个备份，便于观察
              sh "gcloud sql backups list --instance=${env.prodMysqlInstance} --limit=10"
              backupTimestamp = sh(returnStdout: true, script: "date -u +'%Y-%m-%d %H%M%S'").trim()
              backupDescription="Flyway backuped at $backupTimestamp (UTC)"
              // gcloud 创建 db 备份
              sh "gcloud sql backups create --async --instance=${env.prodMysqlInstance} --description=\'$backupDescription\'"
              sh "gcloud sql backups list --instance=${env.prodMysqlInstance} --limit=10"
            }
          }
        }
    }
    
    stage('表结构备份') {
      steps {
        withCredentials([usernamePassword(credentialsId: "sql-secret-production", passwordVariable: 'sqlPass', usernameVariable: 'sqlUser')]) {
            container('mysql-client') {
              sh "mysqldump -h ${env.prodMySqlHost} -u$sqlUser -p$sqlPass -d ${params.dbName} --single-transaction > ${params.serviceName}-${params.dbName}-`TZ=UTC-8 date +%Y%m%d-%H%M%S`-dump.sql"
            }
        }
        // 表结构备份同步到 gcs 存储桶
        container('gcloud') {
          sh "gsutil cp *-dump.sql ${env.gcsBackupBucket}/db/production/${params.serviceName}/"
        }

        // jenkins 归档数据库备份，可在 BlueOcean 页面制品页查看
        archiveArtifacts "*-dump.sql"
      }
    }

    stage('拉取 db 脚本') {
      steps {
        script {
          checkout([$class: 'GitSCM', branches: [[name: "master"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'jenkins-deploy', url: "${env.dbMigrationGitRepoGroup}/${params.serviceName}"]]])
        }
      }
    }
    
    stage('flyway migrate') {
      steps{
        script {
          host = "${env.prodMySqlHost}"
          timestamp = sh(returnStdout: true, script: "TZ=UTC-8 date +%Y%m%d-%H%M%S").trim()
          flywayStateFile = "flyway-state-production-${params.serviceName}-${params.dbName}_${timestamp}.txt"

          container('flyway') {
            withCredentials([usernamePassword(credentialsId: "sql-secret-production", passwordVariable: 'sqlPass', usernameVariable: 'sqlUser')]){
              sh "flyway -url=jdbc:mysql://${host}/${params.dbName}?useSSL=false -user=${sqlUser} -password=${sqlPass} -locations=filesystem:${params.dbName} -baselineOnMigrate=true repair"
              sh "echo \"[ flyway 升级前 db 状态 ]\" > $flywayStateFile"
              sh "flyway -url=jdbc:mysql://${host}/${params.dbName}?useSSL=false -user=${sqlUser} -password=${sqlPass} -locations=filesystem:${params.dbName} -baselineOnMigrate=true info \
                 | tee -a $flywayStateFile"
              try {
                timeout(time: 8, unit: 'HOURS') {
                  env.isMigrateDB = input message: '确认升级 DB?',
                  parameters: [choice(name: "isMigrateDB", choices: 'Yes\nNo', description: "您当前选择要升级的是${params.serviceName}服务${params.dbName}库，确认升级？")]
                }
              } catch (err) {
                sh "echo 'Exception!' && exit 1"
              }
              if (env.isMigrateDB == 'No') {
                sh "echo '已取消升级!' && exit 1"
              }
              sh "flyway -url=jdbc:mysql://${host}/${params.dbName}?useSSL=false -user=${sqlUser} -password=${sqlPass} -locations=filesystem:${params.dbName} -baselineOnMigrate=true migrate"
              sh "echo \"\n\n[ flyway 升级后 db 状态 ]\" >> $flywayStateFile"
              sh "flyway -url=jdbc:mysql://${host}/${params.dbName}?useSSL=false -user=${sqlUser} -password=${sqlPass} -locations=filesystem:${params.dbName} -baselineOnMigrate=true info \
                 | tee -a $flywayStateFile"
              archiveArtifacts "$flywayStateFile"
            }
          }
        }
      }
    }
  }
}
```
