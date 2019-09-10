---
layout: post
title:  "iOS 构建 - jenkins 基础"
# date:   2018-06-23 10:05:05 +0800
categories: blog
---

* 目录
{:toc}

## jenkin 相关设置

### 在 git 拉去代码前修改参数

设置 Prepare an environment for the run ，可以在 git 拉代码之前修改参数

![](/assets/img/jenkins1.png)

groovy 脚本
``` groovy
if ("".equals(giturl)) {
  giturl= "git@172.20.10.102:mobile/client.git"
}

// 会覆盖全局参数
return ["giturl": giturl]
```

### 自定义工作空间

可以把指定的目录作为 WORKSPACE 来构建项目，在 “genral -> 高级” 中设置

![](/assets/img/jenkins2.png)

pipeline 中通过 customWorkspace 来设置工作空间，注意里面不可以使用变量。

```
agent {
    node {
        label "$label"
        // 不能使用 WORKSPACE 或 node 节点预定义的变量，只限使用 jenkins 中的参数
        customWorkspace "../iOS_yzj_build_mixed/projects/${projectcode}/workspace"
    }
}
```

### 修改显示名称

修改 job 显示的名称，算是 job 的别名 alias

![](/assets/img/jenkins2.png)

### 修改每一个构建任务的名称

每次的构建都可以重新命名，在 “构建环境” 中设置

![](/assets/img/jenkins3.png)

在 pipeline 中可以通过命令设置 

``` groovy
pipeline {
    stages {
        stage('name') {
            steps {
                script {
                    currentBuild.displayName = "#${BUILD_NUMBER}_${projectcode}"
                    currentBuild.description = "描述"
                }    
            }
        }
    }
}
```

### node 节点管理

- 可以通过“环境变量” 设置不同节点的变量。例如用户名、密码、PATH 等
- 启动方式 通过 ssh 连接，在 Credential 中设置 node 节点的用户名、密码
- 要设置为 “Non verfying verification strategy”
- 工作目录是默认的工作目录，可以在不同的 job 中设置
- slave 的机器需要打开 "共享 -> 远程登录" ，支持远程访问


注意点：

1. Host 设置为 Mac 的 ip 地址
2. Credentials 设置为电脑的用户名和密码
3. Host Key Verification Strategy 设置为 Non veryfing Verification Strategry
4. Remote root directry 设置为 /Users/kingdee/jenkins_workspace   (kingdee为用户名)
5. Labels 用于给 node 分组，在脚本中通过指定 Label 来实现同一个 job 自动负载均衡到多机器构建

![](/assets/img/jenkins4.png)

## pipeline

### 相关概念

Pipelines 由多个 step 组成，包括构建、测试、部署等流程。

**节点 node**

节点是一个机器

**阶段 stage**

stage 块定义了在整个流水线的执行任务的概念性地不同的的子集(比如 "Build", "Test" 和 "Deploy" 阶段)

**步骤 step**

一个单一的任务, a step 告诉Jenkins 在特定的时间点要做什么。


### shell 执行结果检测

exit(0) 会认为 shell 正常执行完成。如果想 shell 执行某个命令失败后，导致 jenkins 任务失败，需要使用 exit(100) (不是0，就认为失败)

例如，检测 pod install 是否成功
``` shell
if [[ -f "/kdweibo.xcworkspace" ]]; then
    exit(0)
else
    echo "文件不存在"
	eixt(100)
fi
```

pod 执行有概率会失败，可以做几次重试操作

``` groovy
stage('pod_install') {
    steps {
        retry(3) {
            sh './jenkins/script/pod_install.sh'
        }
    }
}
```

### demo

在 git 项目中有 jenkins 目录，在 script 目录中放脚本。

``` groovy
pipeline {
    agent {
        node {
            label "$label"
            // 不能使用 WORKSPACE 或 node 节点预定义的变量，只限使用 jenkins 中的参数
            customWorkspace "../iOS_yzj_build_mixed/projects/${projectcode}/workspace"
        }
    }
    stages {
        stage('git_checkout') {
            steps {
                script {
                    currentBuild.displayName = "#${BUILD_NUMBER}_${projectcode}"
                    currentBuild.description = ""
                }
                
                echo "WORKSPACE: $WORKSPACE"
                checkout([$class: 'GitSCM', branches: [[name: '$definedBranch']], browser: [$class: 'AssemblaWeb', repoUrl: ''], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: '$git_url']]])            
            }
        }
        stage('config') {
            steps {
                echo ""
                sh './jenkins/script/config.sh'
            }
        }
        stage('pod_install') {
            steps {
                echo ""
                retry(3) {
                    sh './jenkins/script/pod_install.sh'
                }
            }
        }
        stage('build') {
            steps {
                echo ""
                sh './jenkins/script/build.sh'
            }
        }
        stage('archive') {
            steps {
                echo ""
                sh './jenkins/script/archive.sh'
            }
        }
        stage('upload') {
            steps {
                echo ""
                sh './jenkins/script/upload.sh'
            }
        }
    }
    post { 
        always { 
            sh './jenkins/script/post_always.sh'
        }
        success {
            sh './jenkins/script/post_success.sh'
        }
        failure {
            sh './jenkins/script/post_failure.sh'
        }
        aborted {
            sh './jenkins/script/post_aborted.sh'
        }
    }
}
```

文档：

- 语法规则：https://jenkins.io/zh/doc/book/pipeline/syntax/