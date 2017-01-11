---
layout: post
title: "iOS多模块管理"
date: 2017-01-05T13:10:32+08:00
categories: iOS
---

# iOS 多模块管理

为了增强代码复用性和屏蔽其他模块的影响等因素，往往会采取将一个iOS项目拆解成多个子工程的方式，每个模块一个子工程。然后用一个大的工程胶合起来。这样子模块在其他的项目中还可以复用。有些团队为了团队之间代码维护方便，也会采取这样的策略。甚至，把每个模块做为一个单独的repo来管理。这样单个项目中就会有多个工程。就拿最近业务做得项目来说已经拆成了60多个工程。而这60多个冲程都是不同的git repo。再加上朱工程，大概有几十个git repo了。管理起来最开始也很是痛苦。于是就有了下面所诉述的探索管理多模块的路程。

最开始的时候是单一工程，所有的代码都堆积在一个工程里面。模块的共享基本上靠手动拖拽。因为我打算把其中一些比较好的工具开源出去。所以很多功能都是单独拆解的。每次同步在这些工具中的修改就非常痛苦，每次都是beyondcompare。文件比对，git的功能完全用不上啊。

为了能够用上git，所以我进行了拆目录的处理。虽然xcode的工程还是只有一个，但是打算开源出去的工具所在的目录都是一个git repo。xcodeproject只是对对应目录下的文件进行了相对引用。这样就组成了：
> 单xcode工程，相对引用源文件，多git repo的结构

可是，每次手工的去同步不同的git repo真心是个繁重的体力活。

为了能够对拆解出来的git repo进行有效的管理，避免每次手工维护。于是开始使用git submodule功能。这个真是从入门到放弃，大概只是用了一个月的时间。这段时间虽然git submodule比手工管理git repo方便多了，但是还是有很多问题。可以参考这片文章。参考[Git submodule的坑](http://blog.devtang.com/2013/05/08/git-submodule-issues/)。

这个管理的工具链还得继续演进。正好这个时候我打算把其中的一些工具和项目进行开源，要进行cocoapod发布。就顺便对这些工程进行了cocoapod的兼容。这个过程中，意外的发现原来cocoapod还有个开发模式！！！可以相对路径引用所有的源文件。这个简直就是梦幻般的功能啊。于是我毫不犹豫的进行了大刀阔斧的改造。 把主工程拆成了一个空壳。其中的业务逻辑代码按照功能进行业务模块进行拆解。并且将业务无关的工具，进行了抽离组成单独的模块。如下图所示。


~~~
├── Bizs
│   ├── NetContants
│   ├── YHActionGroup
│   ├── YHAppearance
│   ├── .......
├── Core
│   ├── Podfile
│   ├── Podfile.lock
│   ├── Pods
│   ├── Rakefile
│   ├── YaoHe
│   ├── YaoHe.app.dSYM.zip
│   ├── YaoHe.ipa
│   ├── YaoHe.itmsp
│   ├── YaoHe.xcodeproj
│   ├── YaoHe.xcworkspace
│   ├── YaoHeTests
│   ├── YaoHeUITests
│   ├── dsymtool
│   ├── fastlane
│   └── hive_pod.rb
├── Libs
│   ├── CTFeedback
│   ├── DZAccountFileCache
│   ├── DZAdjustFrame
│   ├── ......
├── Project
│   ├── Readme.md
│   └── default.xml
└── tools
    └── cassowary
~~~

1. Core是主工程，现在里面基本上就是个空壳。没有任何业务相关的代码，只有一些全局共享的资源还有工程配置文件。
2. Bizs是业务逻辑部分，每个子目录都是一个单独的业务模块。这些业务模块都是一个个的Podspec。
3. Libs是打算开源出去的代码。其中也包含一些第三方开源的Github项目，因为可能会涉及到修改，也放在里面了。之所以将Bizs和Libs进行隔离，也是按照机制与策略分离的思想，把业务逻辑放一堆，把机制部分放一堆。
4. Project是为了适配Google Repo工具的配置文件。
5. tools下面放一些脚手架。


将主工程拆成空壳之后，势必有一个问题是：如何再将所有的源码胶合进来。这里就用到刚才提到的Cocoapods的开发模式。因为将Bizs和Libs下面的子模块都做成了单独的podspec。所以我在Core下面的Podfile对其进行相对路径引用：

~~~
pod "xxx", :path=> "../Bizs/xxx"
~~~

这样就完成了模块的拆解与胶合。而在这里每一个模块都是一个git repo，累积下来也得有60多个了。其实原先的git submodule的方式也是用的我心累。于是就在网上搜这方面的工具，尝试了git slave之类的东西，都不是很顺手。最后意外的发现了Android源码管理工具**repo**这个神奇的工具。于是进行了向repo迁移的工作。结果你还别说，repo进行多个git库管理的时候还真是方便多了。于是现在的多模块管理变成了现在的样子：

> 以Cocoapods为核心工具，将每个子模块拆解成podspec。主工程中只有配置文件，没有代码，代码通过cocoapods的开发模式相对引入到的主工程中。每个子模块同时也都是一个git repo。通过Google Repo来管理多个git repo。

使用google repo不太推荐直接用google的原版，经验证[https://github.com/esrlabs/git-repo](https://github.com/esrlabs/git-repo)这个版本的repo进行了多项改进，更适合在没有girrit的情况下使用。因为我后台的源码管理用的是phabricator。所以需要一个直接push的repo工具。

到这个时候基本的多模块管理的框架就算搭起来了。但是吧还是有一些小的问题需要演进一下。比如每次新建一个子模块都得手工的去创建git repo得各种配置。于是就写了一个简单的创建podspec的脚手架[Hive](https://github.com/yishuiliunian/.hive)。意思吗就是蚁巢。希望所有的模块就像一个蚁穴一样错落有致。

还有个问题就是每次新建了项目会后都得手工的去修改Core下面的Podfile文件。需要保持Podfile文件中的podspec与Libs和Bizs下面的子工程保持一直，这也是个啰嗦的事情。于是对Podfile进行了改进。其实Podfile就是一个Ruby文件，在里面可以直接写Ruby的代码：

~~~
require 'pathname'
REPO_ROOT_NAME = '.repo'

def FindRepoRoot(path)
  pwd = Pathname(path)
  if pwd.to_path == '.repo'
    return pwd
  end
  aimPath = nil
  pwd.entries.each { |x|
    if x.to_path == REPO_ROOT_NAME
      aimPath = pwd.join(x)
      break
    end
  }
  if aimPath != nil
    return aimPath
  else
    parentDir = pwd.dirname
    if parentDir.to_path == REPO_ROOT_NAME
      return nil
    end
    return FindRepoRoot(parentDir)
  end
end


REPO_ROOT = FindRepoRoot(Pathname.getwd).realpath


def FindAllSubPodspec(path)
  podspecs = []
  path.entries.each { |entry|
    if (entry.extname == '.podspec' || entry.extname=='.podspec.json')
        podspecs.push(path.join(entry))
    end
  }
  if podspecs.count != 0
    return podspecs
  else
    path.entries.each {  |entry|
      next if entry.to_path == "."
      next if entry.to_path == ".."
      if File.directory?(path.join(entry))
        subpath = path.join(entry)
        ps = FindAllSubPodspec(subpath)
        podspecs = podspecs + ps
      else
      end
    }
  end
  return podspecs
end

def LibPod(name)
  pod name , :path=>"../Libs/"+name
end

def DebugLibPod(name)
  pod name , :path=>"../Bizs/"+name, :configurations => ['Debug']
end

def BizPod(name)
  pod name , :path=>"../Bizs/"+name
end
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENABLE_BITCODE'] = 'NO'
    end
  end
  require "fileutils"
  require "rake"
  Rake.sh ''' markdown2 Pods/Target\ Support\ Files/Pods-YaoHe/Pods-YaoHe-acknowledgements.markdown > Pods/Target\ Support\ Files/Pods-YaoHe/Pods-YaoHe-acknowledgements.html
 echo "<title>法律声明</title> <meta charset="utf-8" /> " | cat -  Pods/Target\ Support\ Files/Pods-YaoHe/Pods-YaoHe-acknowledgements.html  > temp.html; mv temp.html Pods/Target\ Support\ Files/Pods-YaoHe/Pods-YaoHe-acknowledgements.html
              cp Pods/Target\ Support\ Files/Pods-YaoHe/Pods-YaoHe-acknowledgements.html YaoHe/Resources/OpenSource.html

           '''
end
DEBUG_PODS = [
  "YHMessageTest"
]

EXCLUDE_PODS = [
  'Mars'
]

target 'YaoHe' do
  podspecs = FindAllSubPodspec(REPO_ROOT.dirname)

  puts("全部本地模块 #{podspecs}")
  podspecs.each { |itor|
    name = File.basename(itor,".*")
    path = itor.relative_path_from(Pathname.pwd).dirname
    puts("pod name #{name} path #{path}")
    if not  name.include? "TestCase"
      if DEBUG_PODS.include? name
        pod(name, :path=>path, :configurations => ['Debug'])
      elsif EXCLUDE_PODS.include? name
        puts "Ignore Pod #{name}"
      else
        pod(name, :path=>path)
      end
    end

  }

  ## 引用的第三方库
  pod 'ChameleonFramework'
  .....
end

target 'YaoHeTests' do
  podspecs = FindAllSubPodspec(REPO_ROOT.dirname)

  puts("对测试模块进行 pod 注入....")
  podspecs.each { |itor|
    name = File.basename(itor,".*")
    path = itor.relative_path_from(Pathname.pwd).dirname
    puts("pod name #{name} path #{path}")
    if name.include? "TestCase"
      if DEBUG_PODS.include? name
        pod(name, :path=>path, :configurations => ['Debug'])
      else
        pod(name, :path=>path)
      end
    end
  }

  pod 'JXAsyncTest'
end
~~~

通过查找google repo的根目录，确定项目根目录，然后遍历每个目录找到所有的子模块。这样就可以一个函数将podspecs加进Podfile里面。经过这样改造之后，增加子模块时就不用关心Podfile的问题了。

这样一个基本顺手的模块管理方式就算基本上完工。这套东西其实也适合大团队使用，但是我一个人用起来也是蛮爽的。后面Libs下面的东西都在陆陆续续的开源出来。虽然不是大的作品，也算是自己这一年来的折腾的收获。

