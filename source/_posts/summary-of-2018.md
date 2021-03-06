---
title: 2018年终总结
categories: Life
date: 2018-12-30 11:06:14
tags: [Summary, Transfer, TODO]
---

从来没有写过真正意义上的年终总结，一方面是懒，另一方面是文笔不行，但年纪越大越发觉得有些事情需要用“笔”记录下来，否则一转眼就忘了。毕业差不多3年了，通过校招入职网易至今，随着公司人数和队伍的壮大，已经感觉是半个“老人”了。入职进来的时候从事客户端研发，负责单独的几个业务开发，随着业务发展及人员扩充，后面负责一条单独的业务线；今年7月中旬内部转岗到了后台开发，负责某基础服务后台系统，并提供前台服务。

# 工作

今年最大的变化，就是换岗位了。从自己最熟悉的客户端领域，转到了从零开始的后端。这一转变来得非常突然，在没有想好应对之策时，我一般的选择是顺其自然。人总要对新鲜的事物保持足够的好奇心，何况是我心心念的后台开发，以致于在我还没能够深入了解后端需要的知识体系与技术架构时，我毅然决然地离开了喂我温饱的土壤，踏上披荆斩棘之路。

说来也巧，在客户端的最后一个版本，尝试了三种新的东西——kotlin、RxJava和构建组件化。有人能够推动项目使用新技术，是一件非常不容易的事情，有几位新入职的同事对kotlin和RxJava比较熟悉，也得益于主管愿意尝试，索性也就跟着用起来。整体感受是踏出了原有的编程思维定势，编写流式函数更有利于构建复杂的模块，通过不断地将大功能拆分成一个个小功能，再通过流式将这些小功能串行或者并行地跑起来，最终可以构造一个复杂的系统。遗憾的是，我用的时间很短，上手较慢，很难从思维定势中走出来，需要不断地练习。组件化也是今年的重点，在各个大公司都差不多推出各种成熟的组件化方案时，反观考拉的工程，也需要思考以一种怎样的方式来使工程解耦，提升人效。年中的时候组内来了一位对组件化有着丰富经验的大神，在他的推动下，结合考拉现有工程的基础架构，组件化的基础构建也就开始了。上线的第一个版本，实现了一套编译期自动收集服务注册的组件化架构，为组件的隔离提供了架构基础。

体验了不到两个月的kotlin，在转到后端后，就结束了，回到了Java时代，而且是JDK7。当然，这不是主要的。对于后端知识的匮乏才是我转岗以后的首要问题。刚转的前两周，除了将工程跑起来，在测试环境部署，剩下的事情就是东看看，西看看。applicationContext.xml是什么，有什么用？谷歌一下。@Component、@Service注解怎么用，有什么区别？继续谷歌。Spring和SpringMVC什么关系？谷歌。RabbitMQ，kafka？谷歌谷歌谷歌。谷人希（原意谷歌是人类的希望，解读为谷歌给人带来了希望啊）。

Spring是整个后端架构基础中的基础，开始啃官网的文档，啃了一段时间发现太慢了，消化也不好（后来才知道由Spring孵化出来的项目有几十个，基础的Spring代码也有几百兆），于是又回到了项目中的代码，看见一处不明白的就谷歌相关的内容，极度焦虑地在啃着。

业务上，跟之前在移动端负责的交易链路完全不一样，这里属于基础服务提供方。一开始在大佬的指导下，通过写单测（写单测和想象中完全不一样……）来熟悉相关的业务，熟悉的过程也不是一蹴而就，对于不好难理解的概念，需要多在测试环境体验系统的各个功能，甚至通过断点来走每一个功能，过程颇为艰辛。

经过了几个月的摸索与迭代，基本能够摸清后台的技术框架路线，业务上能够帮助别人排查问题，并开发新功能了。

# 生活

3月份的时候和阿咪小朋友跟着公司的年度旅游去了一趟苏梅岛，来回花了差不多2天，剩下的4天时间在岛上自由行。与其说是去旅游，不如说是去散散心。最近的几次旅游，观念发生了变化，再也不是想去哪个网红的景点打卡，而是去体验一下当地纯正的美食。真正的当地美食一般都是在不起眼的小店里，价格也不贵，吃起来颇为顺心。最记忆犹新的是在一家店了前后点了3盘空心菜。

从7月转岗到后端以后，又回到了本部，加上胃一直不怎么好，胃动力不足，于是恢复了健身。前期的时候基本坚持一周四天，每天半小时以上，主要是撸铁，练胸肌、背、肱二肱三以及大腿。后面天气逐渐转冷了，业务也开始了新项目，每周变成不定期的去健身房了。希望能够坚持下来吧。

18年最重大的一件事，就是摇到了号，买个了房子。回头来看，第一感觉是买在了高点，行情已经回落了，特别是二手房，降价特别厉害；第二是掏空了“六个钱包”，还债路漫漫；第三是最近几个月买新房都非常闹心，几乎所有的楼盘都在打精装修的主意，减配、“惊”装、质量差、维权成了大部分楼盘的标签。但内心还是喜悦的，再过一年，终于在杭州有个家了。

12月底，回了一趟家，参加了两场非常要好的朋友的婚礼。一场是林丫丫的，和她一起度过了人生中最难熬的高考时光，那时候的相互鼓励至今令我难忘，一段美好的时光就此定格，祝福她！另一场是阿咪小朋友的好朋友，也是陪伴她度过大学时光的闺蜜，感谢她对阿咪的照顾，祝福！