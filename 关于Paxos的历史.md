### Paxos历史回顾
来自：https://blog.csdn.net/voidccc/article/details/39647787

作者：phylips@bmy 2012.12.21

出处：http://duanple.blog.163.com/blog/static/709717672012112203543166/

```
自Paxos提出，迄今已有20多年了，围绕着该算法曾经发生过一些非常有趣的事情，这些也已成为人们津津乐道的一段轶事，故事的主角自然是Paxos的提出者Lamport，当然Lamport的特立独行也是很早就出了名的。首先来讲述下这些有趣的八卦，之后会再理一下Paxos的整个发展过程，以及在这个过程中产生的一系列比较重要的论文，总共会涉及到十几篇论文，如果有时间还是最好都研读一下。

关于这段历史， Lamport本人在他的“My Writings”里对于论文<<The Part-Time Parliament>>的整个从创作到发表的过程，也做过非常详细的描述。我们还是先从这里开始吧。

（转述开始）“上世纪80年代末，SRC(Systems Research Center，DEC于1984年创立，Lamport也曾在此工作过)构建了一个被称为ECHO的容错文件系统。该系统的构建者们声称，系统可以保证在发生任意数目的非拜占庭式错误情况下的一致性，并且只要有半数以上的进程可以正常工作，它就可以保证系统向前运转。对于大多数像这样的系统来说，在没有错误发生的时候都是很简单的，但是要处理好实现者所有能够想到的各种错误的话，是需要一个非常复杂的算法的。我感觉他们在做的事情基本上是不可能的，并打算去证明它。结果事与愿违，在这个过程中我发现了Paxos算法，也就是本文描述的那个。该算法的核心实际上是一个三阶段的一致性协议。Dale Skeen应该是第一个意识到为避免任意可能的单点失败引起的阻塞需要引入三阶段协议的人。但是，据我所知，Paxos是第一个具有清晰正确性条件说明及完整正确性证明的真正意义上的三阶段提交算法。

无论那时，还是现在，我一直认为Paxos是一个非常重要的算法。在写此文之前，我通过采用拜占庭将军的描述方法使得一致性问题成功地被人们所接受(即论文The Byzantine General Problem)，受此启发，我决定通过一个古希腊岛屿上的议会来描述该算法。Leo Guibas提议将这个岛命名为Paxos。我又用工作在该领域的计算机科学家的名字对里面的希腊议员进行命名，并在 Guibas的帮助下翻译成了伪造的希腊方言。(论文题目是由Peter Ladkin提议的)。通过描述一个失落的文明，既增加趣味性又可以让我免于陈述其中乏味的细节，只要直接声明一下关于该议会协议的某些细节已经遗失就可以了。为了让这种形象化的描述更逼真，我还戴上了Stetson毡帽屁股系上了小酒瓶装扮成印第安纳琼斯style的考古学家，进行了几次演讲。

只是我这种尝试不幸地失败了。那些参加我演讲的人们只记住了印第安纳琼斯style，而不是该算法。阅读该论文的人们看起来也饱受里面的希腊故事的干扰以至于根本没有理解该算法。在我发论文给他们阅读的那些人中，其中 Nancy Lynch(分布式理论研究大牛，<<Distributed Algorithms>> 一书作者), Vassos Hadzilacos, 和Phil Bernstein声称已经读完。大概两个月后，我给他们发了封邮件，邮件内容为“是否能够实现一个可以容忍任意数目(可能是所有)的进程出错且能保持一致性的分布式数据库，同时当半数以上进程恢复正常时该系统也能够恢复正常”。但是他们并没有人意识到该问题与Paxos算法间的关联。

我在1990将该论文提交给了TOCS。三个审稿者都认为该论文尽管并不重要但还有些意思，只是应该把其中所有Paxos相关的内容删掉。在该领域工作的人们如此缺乏幽默感，实在令人生气，此后我也没有对该论文做任何修改。很多年后，在SRC工作的两个人需要为他们正在构建的分布式系统寻找一些合适算法，而Paxos恰恰提供了他们想要的。我就将论文发给他们，他们也没觉得该论文有什么问题。下面是Chandu Thekkath所讲述的Paxos在SRC的历史：“在Ed Lee和我从事Petal相关的工作时，我们需要一种提交协议来确保分布式系统中的全局操作即使是在发生故障的情况下也能保证正确性。我们了解到一些3PC的概念，并阅读了Bernstein, Hadzilacos和 Goodman的经典书籍<<Concurrency Control and Recovery in Database Systems>>，对其进行了研究。我们发现这个协议有些难以理解，于是放弃了对它进行实现的尝试。大概也是在这个时间，Mike Schroeder与我们谈起Leslie Lamport 曾经发明了一个一致性方面的协议并建议我们直接问下Leslie本人。后来Leslie给了Ed一份<<The Part-Time Parliament>>文章的拷贝，我们两都读地很happy。我尤其喜欢其中的幽默，直到今天也无法理解为什么人们都不太喜欢这篇文章。Paxos具备我们的系统所需要的所有属性，同时我们也觉得能够将它实现出来。此外Leslie还为我们提供了很多咨询帮助，这样就产生了我所知的Paxos算法的第一个实现(包含了动态重配置)。一年后，我们又用Paxos实现了Frangipani file system所需的分布式锁服务器”

因此，我觉得重新发表该论文的时机到了。

与此同时，在整个悲催的经历中(指论文一开始被拒，没有人重视)，Butler W.Lampson(1992年图灵奖得主)是一个例外，他立刻意识到这个算法的重要性，并在他的演讲和一篇论文(即<<How to Build a Highly Availability System using Consensus>>)中对该算法进行了描述，这引起了Nancy Lynch的关注。此后De Prisco, Lynch和Lampson发表了他们那个版本的描述和证明(即<<Revisiting the PAXOS algorithm>>)。他们那篇论文的发表更使我确信是时候发表我的这篇论文了。于是我提议当时TOCS的编辑Ken Birman发表该论文。他建议我再修改下，比如添加一个关于该算法的TLA描述。但是重读该论文后，我更确信其中的描述和证明已经足够清晰，根本不需要再做改动。诚然，该论文可能需要参考下最近这些年发表的研究成果进行修订。但是一方面作为一种joke式的延续，另一方面为保存原有工作，我建议不是再写一个修订版本，而是以一个被最近发现的手稿的形式公布，再由Keith Marzullozz做注。Keith Marzullo很乐意这样干，Birman也同意了，最终该论文得以重见天日。

这样该论文就有了一些有趣的排版注脚{!即论文<<The Part-Time Parliament>>里Keith Marzullo的那些注，具体内容见原文}。为了让Marzullo的注解更显眼，我认为它们应该印在一个灰色背景上。那时ACM刚获取了一些非常棒的排版软件，同时TOCS也准备不再接收camera-ready copy。不幸的是，他们的新软件做不出阴影效果。于是，我不得不为那段文字提供一个camera-ready copy。但是，他们的那个牛逼软件只能以漂浮图的形式接收我提供的这个拷贝，因此Marzullo的注释的画面效果看起来可能并不理想。还有更不幸的是，他们那个超级昂贵的软件竟然不支持数学公式。(好吧，毕竟他们是一家计算机期刊，so可能不需要对公式排版吧…)。因此我又不得不为A2节里的不变性定义提供一个camera-ready copy，在发表的版本里他们将它作为了图3。这就是那张图里的字体看起来与其他部分的字体并不匹配的原因。

该论文获得了2012年的ACM SIGOPS Hall of Fame Award{!是不是很熟悉，之前介绍过的Lamport的另一篇文章<<Time Clocks and the Ordering of Events in a Distributed System>>就获得了2007年的该奖项。该奖项始创于2005年，主要颁发给那些在操作系统领域产生了深远影响的论文，这些论文都至少经过了十年的检验。获奖结果将会在每年的OSDI或SOSP(操作系统领域顶级会议，通常是偶数年开OSDI，奇数年开SOSP)上宣布。今年的OSDI上除Lamport的这篇获奖外，Liskov(2008年图灵奖得主，她导师是1971年图灵奖得主John McCarthy)1988年写的<<Viewstamped Replication: A New Primary Copy Method to Support Highly-Available Distributed Systems>>也是获奖论文，这两篇都是Paxos相关的。}”（转述结束）

从上面可以看到，在Lamport的各种挖坑和吐槽下，已有无数人中枪，连ACM的排版软件也不能幸免(好吧，其实此处Lamport有为他的LaTeX打广告之嫌)。关于该论文的整个曲折的发表过程就是这样的。另外值得关注的是SRC，它的Petal和Frangipani是不是非常类似于Google的某些东西啊，而且在<<The Part-Time Parliament>>发表前，这些系统就已经完成了Paxos的实现。

现在我们重新回到Paxos。它的整个发展过程实际上大概可以分为三个阶段。

第一个阶段，为算法创立阶段，基本就是88年和89年，代表性事件是88年Liskov发表 <<Viewstamped Replication: A New Primary Copy Method to Support Highly-Available Distributed Systems>>，89年Lamport写出了<<The Part-Time Parliament>>，并在90年将它提交给TOCS。这两篇都是独立完成的，而且Liskov那篇还要早些，但是因为上述种种轶事，大家最终还是称之为Paxos，而且在Paxos的盛名下，Liskov那篇估计更少有人知道了。最早指出二者本质上是一样的人是Butler Lampson。在这个阶段，这两篇论文基本上都还没有人关注。当然这两篇论文的殊途同归，也从一个侧面说明解决异步一致性问题基本上也没有其他的路子可走。

第二个阶段，可以从开始引起理论研究界的重视算起。代表性事件是96年Butler Lampson发表<<How to Build a Highly Availability System using Consensus>>。正是经过Lampson的大力宣传，该算法才逐渐为理论研究界的人们所重视，也直接导致了The Part-Time Parliament的重新发表。为啥当年Lamport一方面费尽心机地扮考古学家，另一方面千辛万苦把论文写的那么幽默，都没能让理论界的人们看上眼，而Lampson振臂一挥，就有无数响应呢？这可能是因为天时、地利、人和吧，从时间上来看，此时人们对于一个分布式一致性算法的需求日益紧迫，而且与Lamport相比，Lampson明显说话还是要更有分量些，Lampson 92年就拿到了图灵奖，江湖地位自然高些，而且他更善于把复杂的问题讲清楚，另外还有就是功力的确深厚，就算Lamport这样的人物对其也是佩服得不行。这篇论文发表之后，后续便引出了一系列关于Paxos的研究。1999年，De Prisco, Lynch和Lampson联合在TCS(Theoretical Computer Science)发表了<<Revisiting the PAXOS algorithm>>对Paxos算法进行了全新的描述及证明，分析了其时间开销及容错性。2001年，Lampson又写了一篇<<The ABCD’s of Paxos>>对Paxos的各种不同形式进行了描述，包括AP：Abstract Paxos；BP：Byzantine Paxos；CP：Classic Paxos；DP：Disk Paxos，发表在PODC’01上。也是在这一年，在参加PODC会议时，Lamport发现人们还是觉得Paxos很难理解，于是回家后对Paxos进行了重新描述和表达后，形成了<<Paxos Made Simple>>这篇文章，而这篇文章基本上就是The Part-Time Parliament的一个简化版本。可以看出，到了2001年，Paxos已经成为理论界关注的热点。此后的2004年，Lamport写了篇<<Cheap Paxos>>，2005年又写了篇<<Fast Paxos>>，对经典Paxos算法进行了改进。

第三个阶段，由Google发表的两篇论文开启，也是从那时起Paxos开始从理论界进入工业实践，并被越来越多的工程技术人员所熟知。2006年，Google有两篇论文发表在OSDI上，一篇是<<Bigtable:A Distributed Storage System for Structured Data>>，另一篇就是<<The Chubby lock service for loosely-coupled distributed systems>>，而在Chubby这篇中有这样一段话“Indeed, all working protocols for asynchronous consensus we have so far encountered have Paxos at their core”。另外还有一句流传甚广的话，“世上只有一种一致性算法，那就是Paxos—Mike Burrows”，这句话可能是出于这篇文章<<Consensus Protocols: Two-Phase Commit>>，是否为Mike Burrows的原话就不得而知了，但是Chubby中的那句的确与这话十分接近。Chubby发表之后，开源社区也推出了与之对应的Zookeeper。此后，Paxos逐步为大众所了解，至少听说过的人越来越多了。Google除了Chubby那篇，当然实际上Chubby并未讲述Paxos相关的内容，还有另一篇非常好的文章<<Paxos Made Live>>，这篇文章详细解释了Google实现Paxos中越到的各种问题及解决方案，讲述了如何将理论应用到实践，而二者之间通常都是具有很大的鸿沟的,尤其是在分布式系统领域，往往都是理想丰满现实悲惨。其实这篇已经完全超越了Paxos算法本身，重要的不是如何实现Paxos(实际上大多数人都不需要实现Paxos)，而是应理解如何将理论应用到实践，如何弥补理论与实践的差异，如何进行分布式系统的工程实现，如何进行分布式系统的测试，而实践中的很多问题理论界并没有给出答案。

另外，如果要自己实现Paxos，还有如下几篇文章可供参考：<<Paxos Made Code>>作者Macro Primi，他实现了一个Paxos开源库libpaxos，更多信息可以参考此处。<<Paxos for System Builders>>以一个系统实现者的角度讨论了实现Paxos的诸多具体问题，比如leader选举，数据及消息类型，流控等。<<Paxos Made Moderately Complex>>，这篇比较新，2011年才发表的，该文章介绍了很多实现细节，并提供了很多伪代码，一方面可以帮助理解Paxos，另一方面也可以据此实现一个Paxos。<<Paxos Made Practical>>是少有的一篇对Liskov的那篇论文进行重新解读的文章，主要介绍如何采用Paxos实现replication，如果要读Liskov的那篇，也可以同时参考这篇。除了Macro Primi的那个开源实现外，目前还可以找到如下一些Paxos实现的代码：Java版，Python版。

关于Paxos的介绍，这两篇文章也非常不错：<<Consensus Protocols: Paxos>>，<<Consensus>>。

```
