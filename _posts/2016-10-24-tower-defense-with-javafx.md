---
layout: post
title: 用JavaFX Canvas实现了个Tower Defense
date: 2016-10-24 15:03:16
updated: 2016-10-24 23:03:16
tags: ["Java", "JavaFX"]
---

　　学校的数据结构小组作业...唔，这里啥代码都没有，图也没有，就扯下感受 o .o
 <!--more-->
 
　　本来在暑假用C++照着论文和一篇文章实现了个Double Array Trie，Qt折腾了个界面，找了下MDX的解析代码，造了个词典拿来当作业的，不过后来写着写着烂尾了，没啥动力写下去，看队友也没啥兴趣也就直接弃坑了。这学期OOP是拿Java来当教学语言的（然而讲了快一半的课时才渐渐涉及点OOP的东西），自己之前除了看《Algorithm 4<sup>th</sup>》外，一直没咋正式接触过Java。后来无意中看到czh聚聚博客里曾经写了个C#版的Tower Defense 当作业，又想到几年前某日下午中过一个 HTML5 塔防 游戏的毒，想着要不随便实现下呗。于是上周匆匆忙忙翻了下教材《Introduction to Java 10<sup>th</sup>》上半部分的后半部分(lll￢ω￢)，瞄了下 <a href="https://gamedevelopment.tutsplus.com/tutorials/introduction-to-javafx-for-game-development--cms-23835" target="_blank">Introduction to JavaFX for Game Development</a>，又瞄了<a href="http://www.redblobgames.com/pathfinding/tower-defense/" target="_blank">Pathfinding for Tower Defense</a>，实现能力还是略捉急的，差不多三个上午才实现了个Demo，这周末修修补补了下界面总算搞定了。
 
　　略感奇怪的是，自己写的BFS出来的路径居然不是看到的文章里走Z字形，而是L或者7...后来用Dijkstra才走Z字形，想到不能傻傻的总贴炮塔走，最后还是选了带权值+Dijkstra实现，也就这样了。不过还有好多没处理的，想着A*还没用，还有平滑下Z字形的问题，还有四叉树折腾下碰撞检测等等。至于游戏数值问题，呃...调了好久还是没找到合适的，最后随机过掉了。
 
　　效率上JavaFX Canvas还是可以的，没卡顿的感觉，瞄了下Windows平台上后端用的是Direct3D，就是内存不知为啥比C#版的大了10倍多，可能跟没掌握各种Best practice也有关吧，有空是得去翻下《Think in Java》《Effective Java》什么的了。总之，编码效率上觉着Java还是比C++好多的样子，比用C++ Direct2D写山寨2048那会好多，那会C++一不小心陷入语言坑，或者自己还是太弱了啊哈哈哈 (*/ω＼*) 。大概学一门语言总要能找个练手，又容易有成就感的东西吧。然而成就感并没有，不甘自己太晚才接触这些了，而且自己大多代码还是靠蛮力堆起来的，技能树上还是得多点算法才行哇 QvQ。（可笑的是，每每搜索时，总会不经意间搜到五六年前就“雪藏”在浏览器书签夹的文章。