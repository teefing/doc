---
category: 日记
tags:
  - git
date: 2020-5-11
title: git revert--一场代码回滚导致的加班
vssue-title: git revert--一场代码回滚导致的加班
---

记录一次代码回滚导致的加班

<!-- more -->

2020年5月9日下午五点。
滕某人本来今天高高兴兴，心想着又是可以下早班的一天，只要把这些小改动发上线就OK了，还不是轻轻松松。可是一顿操作之后把代码提审通过，发到预发环境一看，？？？怎么改的东西有的生效了有的没生效，会不会是代码提得不对，嗯，一定是这样，再提一遍。gitlab上一看，呐尼，no commit，可是本地代码和线上代码的确是不一样呀，怎么会no commit呢？一番冥思苦想后，脑海中浮现了两个小时前的场景。

时间回溯到2020年5月9日下午三点。
打开gitlab，source branch填上我的分支，target branch填上今天的hotfix分支（这里其实没有点上，默认master了），回车，指定代码审核人，OK，走你！！！五分钟后，回gitlab一看，嗯，审核通过了，可是。。。，怎么有种奇怪的赶脚，艹了，我怎么把代码直接提到master上去了，他（指审核人）竟然还给我通过了，这么盲目信任我的吗，可是我好菜的。。。我这次的改动依赖后端，可千万不能发线上，得赶紧把master代码回滚一下，
-那个谁，刚刚提的merge request，提错分支了，你赶紧把master回滚一下。 
-xxxxxx(祖安语),好了，我给你revert一下。
作为一个平时只会push操作的git菜鸟，面对revert这样的陌生词汇，一脸蒙蔽的点了一下头，又忙起了手上的活，之后又提了做了一些改动，提了几个commit。

OK，事情的经过回溯完毕，问过大佬后，原因明了了：master代码回滚是通过revert方式的，从而导致我之后提交的时候显示no commit，为什么呢。

revert操作，中文意思是反转，顾名思义，是将之前有问题的commit操作的逆操作，作为一个新的commit，提交了上去，比如之前我加了一行有问题的代码，通过revert，我相当于又进行了一次提交，提交的内容是删除了那行有问题的代码，从而达到恢复代码的目的。可是，在revert操作下，
有问题的那个commit和revert的commit都是保留在了commit记录中的，git分支都是向前走而非回退，导致了之后我想把代码提上去，是提不上去的，因为那个commit之前已经提过了(在回滚的时候那个commit是有问题的commit，可是之后走正常流程了，该commit就是正常的commit了，可是git无法判断有没有问题)，因此显示的是no commit。

当时的解决方法是对那个revert的commit，我再revert一次，从而抵消了revert带来的影响。

从发现问题，到找原因，到解决问题，再到重新发预发，确认，再发线上，确认，时间也不知不觉地到了晚上八点，哎，都八点了，直接等到九点打车吧（公司九点后打车免费）--又被996了 --其实互联网人的996，就是能力不行导致的瞎折腾，还是得学习进步呀 -- END