IDEA和java都是内存大户，不仅ide自身吃内存厉害，调试时运行的java  application也特别能吃。

如果项目基于spring boot，通常有较多的模块，每个模块基本都是一个独立的服务。那么每次开启一个分支都要重新去Run/Debug Configurations里去一个个配置JVM参数，特别麻烦。

上官网搜了下，2006年就有人提过[这个问题](https://intellij-support.jetbrains.com/hc/en-us/community/posts/206330349-Default-VM-Parameters-For-All-Run-Configurations)，但是至今都没有一个完美的解决方案。一种可用的做法是，把Run/Debug Configurations里当前所有Spring Boot删掉，再从Template-Spring Boot里配置一下JVM（以及其他通用的配置），需要运行时找到对应的application.java执行一下即可。

关于Template，[官网](https://www.jetbrains.com/help/idea/2018.2/run-debug-configurations-dialog.html?utm_content=2018.2&utm_medium=link&utm_source=product&utm_campaign=IU)是这样解释的：
> Under the Templates node in the tree view of run configurations, you can select a run configuration template and edit its default settings. This will not affect the configurations that are already created, but will be used as defaults when creating new configurations of the corresponding type.

就是说Template里的修改对已存在的不会生效，但当有新的此类模板建立时会沿用，给人一种踢一脚动一下的感觉。