记忆方式

时序图，侧重方向，线只使用 - 。箭头得使用double

流程图，侧重连接，线使用- . 。箭头有无即可

类图，固定方式表达。线与箭头5+3表示

# 时序

```mermaid
sequenceDiagram 
participant l as lushixiang
participant j as jiangfengdie 
l -->+ j: hello
j -->- l: hello too

l ->+ j: hello
j ->- l: hello too

l -->>+ j: hello
note right of l: love jiangfengdie
note over l : wait for response
j -->>- l: hello too

note over l,j : conversation is over

loop talk test
	l -x j: hello
	j --x l: hello too
end 

alt 1
	l -> j: hello 
else 2
	j ->> l: hello too
else 3 
	l --> j: hello 
end



par 摸鱼
	l ->> j: hello 
and 
	l ->> l: do not give up
end 
```

```mermaid
graph LR
	kaishi  --> jieshu
	start[start node]
	start_01(start node)
	start_02{start node}
	start_03{{start node}}
	start_04((start node))
	start_05[/start node/]
	start_06[\start node/]
	
	start_01 --> start_02
	start_02 ==> start_03
	start_03 -.-> start_04
	start_05 --> start_01
	start_05 --Y--> start_06
```

page content example content[^exampleContent]

[^exampleContent]:example content 是为了演示脚注功能的实现

$x+y=z$

$ x \geq y = z$



$ x \div y =z $





# 类图



> 描述类与类与类之间的关系
>
> 类：所属包，类型，属性，方法
>
> 关系：数量关系，逻辑关系
>
> [参考地址：30分钟学会UML类图](https://zhuanlan.zhihu.com/p/109655171)

**属性**

public +

protect #

private -

default default

**方法**

+/-/# MethodName(XX) ReturnType

抽象类使用斜体，类使用常规体，接口标明\<interface>

![image-20221212171920326](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20221212171920326.png)

包的表示

关系的表示

![image-20221212172116728](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20221212172116728.png)

```mermaid
classDiagram
	鸟 --|> 动物 : 继承
  翅膀 "2" --> "1" 鸟 : 组合
  动物 ..> 氧气 : 依赖
  动物 ..> 水 : 依赖
  
	class 动物 {
    <<interface>>
    +有生命
    +新陈代谢(氧气, 水)
    +繁殖()
	}
	
	class 鸟 {
		+羽毛
		+有角质喙没有牙齿
		+下蛋()
	}
	class 鸟 {
		+羽毛
		+有角质喙没有牙齿
		+下蛋()
	}
```





```mermaid
stateDiagram
        [*] --> 激活状态

        state 激活状态 {
            [*] --> NumLock关
            NumLock关 --> NumLock开 : 按下 NumLock 键
            NumLock开 --> NumLock关 : 按下 NumLock 键
            --
            [*] --> CapsLock关
            CapsLock关 --> CapsLock开 : 按下 CapsLock 键
            CapsLock开 --> CapsLock关 : 按下 CapsLock 键
            --
            [*] --> ScrollLock关
            ScrollLock关 --> ScrollLock开 : 按下 ScrollLock 键
            ScrollLock开 --> ScrollLock关 : 按下 ScrollLock 键
        }			
```



# Git

> 不错的文章[实用技巧和原理解读](https://colstuwjx.github.io/2020/11/git实用技巧和原理解读/)
>
> [彻底搞懂 Git-Rebase](http://jartto.wang/2018/12/11/git-rebase/)
>
> 千万不要用增量的方式理解。git一直是全量快照。开发使用intellij 一般都是自动add

Git区域概念：工作区，暂存区，本地仓库。对于所有分支来说，工作区和暂存区是公共的

本地仓库层级模型：git会有当前指针，head，head指向不同breach（方便切换breach）。breach指向version（提交对象）（breach切换到不同version），version指向Object（树对象记录着目录结构和blob对象索引），Object指向不同文件（含hash值）

工作区产生作业时：被改动文件会被另存。索引index指向最新

Git文件记录方式：Git记录各历史版本的全量快照，但是为了节约空间，对于没有更改的文件只是通过链接指向历史版本中的最新实体文件。SVN等是各个对于不同version保存增量diff。俩者带来的差别就是version切换后，Git可以直接切换到具体文件，而SVN需要逐个作用diff，生成该version的文件。

常见使用场景

1. 文件回退：git checkout fileName，git reset fileName，git checkout fileName
2. 文件暂存&分支切换：git stash/git stash pop 
3. commit合并：git rebase -i HEAD~4
4. branch合并：git rebase master/git merge master
5. commit转接：cherry-pick
<<<<<<< HEAD
=======

LATEX
>>>>>>> 2d3d04346993a0bd92580c05f2e9a99ab73b2a68
