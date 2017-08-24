---
title: Complete and Sound
date: 2017-03-03 00:04:26
tags: 
- 知识点
---

### Complete and Sound

我总觉得自己脑子有问题，有些知识点怎么都记不住。就算是恍然大悟，过些天竟然还能忘掉。比如上面的complete和sound，我就是怎么都记不住，气都气死了。Stackexchange有个问题，问的是算法中的complete和sound有什么区别，连接在[这里](http://softwareengineering.stackexchange.com/questions/140705/what-does-it-mean-to-say-an-algorithm-is-sound-and-complete)。

如果一个算法是sound的，那么这个算法的输出一定是正确的（这里并没有说说算法一定会终止）。比如，一个排序算法的输出总是排好序的数，那么这个算法就是sound。

如果一个算法是complete的，那个对于**任何**一个输入，算法都能给出正确的输出。

---

再看这个[链接](http://stackoverflow.com/questions/21437015/soundness-and-completeness-of-systems)。对于一个系统来讲，也有sound和complete之说。一个系统是sound的，那么它绝对不会接受有问题的程序（有问题的进不来）。如果一个系统是complete，那么它不会拒绝任何一个没有问题的程序（没问题的一定被容纳）。

这里再引入连个概念，false negative和false positive。positive既阳性，一个程序是positive则它是有问题的，反之，negative就是一个良好的程序。false negative的意思是“假阴性”，也就是本来是有问题，系统认为是没问题的，可以理解为“漏报”。false positive既是把没问题的说成是有问题的，即“误报”。一个系统是sound的，即它不会接受有问题的程序，那么它就能够识别出一切有问题的程序，因此，就不会出现漏网之鱼，即“漏报”，就false negative。如果它是complete，没问题的程序它是一定能接收的，这样没有问题的程序，就不可能被识别为是有问题的，即不会出现“误报”的现象。所以它避免了false positive。这就是为什么说*Soundness prevents false negatives and completeness prevents false positive*。

虽然我写下来，貌似好像我懂了，但是概念还是很绕，说不定过段时间又忘了。。。

