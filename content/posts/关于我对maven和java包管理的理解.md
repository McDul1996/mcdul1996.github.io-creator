---
title: "关于我对maven和java包管理的理解"
date: 2020-05-30T11:45:07+08:00
draft: true
---

# java的包管理

# 1. 什么是包

* 就是一个包含许多类的一个压缩包
* 这个包的作用是： 
当JVM要找一个类的时候，它就会从classpath(类路径)里挨个一个个包里寻找这个类

# 2. 什么是包管理
* 包管理的本质就是告诉JVM如何找到所需要的第三方类库
* 并且可以解决其中的冲突问题(classpath里的某些包都含有同名不同版本的类，由于基本规则选择的那个优先的类里面的代码并不是你想要的，可能还存在BUG)

# 3. Maven是什么
* Maven是划时代的包管理工具(必须强调Maven远远不止是包管理工具)
* Convention over configuration(约定优于配置)
* Maven有两个仓库
* Maven的中央仓库：在远端的服务器上
* 包含了世界上所有的包(按照一定的约定存储包)
* Maven的本地仓库：默认位于~/.m2
* 你不可能每次编译的时候都把世界上所有的包下载下来，你可能想要在断网的时候进行编译
* 当你需要用包的时候，maven会帮你自动的从中央仓库下载，下载的第三方包放在这里进行缓存

# 4. Maven的包
* 按照约定为所有的包编号，⽅便检索
* groupId/artifactId/version
* 扩展：语义化版本
* SNAPSHOT快照版本：
* 因为maven有一个约定：当你这个包发布的时候，不允许你再去修改它，但是你在开发过程中需要频繁的修改它
* 传递性依赖：虽然你只下载了一个包，但maven会根据pom将这个包的完整的依赖树都下载下来
* 传递性依赖的⾃动管理
* 原则：绝对不允许最终的classpath出现同名不同版本的jar包 
* 依赖冲突的解决原则：最近的胜出(但是很可能这个胜出的包里面的类的代码并不是你想要的，这个时候我们就需要根据自己的需要利用Maven使自己想要的包胜出)
* 方法1：既然是谁离得近谁赢，那就直接在我的项目里直接依赖我想要的版本的jar包
* 方法2：直接使用  <exclusions>排除那个胜出的版本的包
