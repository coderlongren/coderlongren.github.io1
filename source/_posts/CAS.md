---
title: CAS
date: 2010-03-20 23:18:40
categories:
	- Java
tags:
	- Java
---
CAS的一点探究
<!-- more -->

一、首先是开场白：
各位老师，上午好！我叫……，是……级……班的学生，我的论文题目是……。论文是在……导师的悉心指点下完成的，在这里我向我的导师表示深深的谢意，向各位老师不辞辛苦参加我的论文答辩表示衷心的感谢，并对三年来我有机会聆听教诲的各位老师表示由衷的敬意。下面我将本论文设计的目的和主要内容向各位老师作一汇报，恳请各位老师批评指导。
二、 内容
首先，我想谈谈这个毕业论文设计的目的及意义。……
其次，我想谈谈这篇论文的结构和主要内容。
本文分成……个部分.
第一部分是……。这部分主要论述……
第二部分是……。这部分分析……
第三部分是……
三、 结束语
最后，我想谈谈这篇论文和系统存在的不足。
这篇论文的写作以及修改的过程，也是我越来越认识到自己知识与经验缺乏的过程。虽然，我尽可能地收集材料，竭尽所能运用自己所学的知识进行论文写作，但论文还是存在许多不足之处，有待改进。请各位评委老师多批评指正，让我在今后的学习中学到更多。
谢谢！
四、 老师提问
答辩的准备工作学生可以从下列问题（第4~10题）中，根据自己实际，选取二三个问题，作好汇报准备，（第1~3题必选）。时间一般不超过10分钟。内容最好烂熟于心中，不看稿纸，语言简明流畅。
1．为什么选择这个课题（或题目），研究、写作它有什么学术价值或现实意义。
2．说明这个课题的历史和现状，即前人做过哪些研究，取得哪些成果，有哪些问题没有解决，自己有什么新的看法，提出并解决了哪些问题。
3．文章的基本观点和立论的基本依据。
4．学术界和社会上对某些问题的具体争论，自己的倾向性观点。
5．重要引文的具体出处。
6．本应涉及或解决但因力不从心而未接触的问题；因认为与本文中心关系不大而未写入的新见解。
7．本文提出的见解的可行性。
8．定稿交出后，自己重读审查新发现的缺陷。
9．写作毕业论文（作业）的体会。
10．本文的优缺点。总之，要作好口头表述的准备。不是宣读论文，也不是宣读写作提纲和朗读内容提要。
学生答辩注意事项
1．带上自己的论文、资料和笔记本。
2．注意开场白、结束语的礼仪。
3．坦然镇定，声音要大而准确，使在场的所有人都能听到。
4．听取答辩小组成员的提问，精神要高度集中，同时，将提问的问题——记在本上。
5．对提出的问题，要在短时间内迅速做出反应，以自信而流畅的语言，肯定的语气，不慌不忙地—一回答每个问题。
6．对提出的疑问，要审慎地回答，对有把握的疑问要回答或辩解、申明理由；对拿不准的问题，可不进行辩解，而实事求是地回答，态度要谦虚。
7．回答问题要注意的几点：
（1）正确、准确。正面回答问题，不转换论题，更不要答非所问。
（2）重点突出。抓住主题、要领，抓住关键词语，言简意赅。
（3）清晰明白。开门见山，直接入题，不绕圈子。
（4）有答有辩。有坚持真理、修正错误的勇气。既敢于阐发自己独到的新观点、真知灼见，维护自己正确观点，反驳错误观点，又敢于承认自己的不足，修正失误。
（5）辩才技巧。讲普通话，用词准确，讲究逻辑，吐词清楚，声音洪亮，抑扬顿挫，助以手势说明问题；力求深刻生动；对答如流，说服力、感染力强，给教师和听众留下良好的印象。
祝毕业季的同学们都能顺利通过答辩哦

Abstract

Raft is a consensus algorithm for managing a replicated
log. It produces a result equivalent to (multi-)Paxos, and
it is as efficient as Paxos, but its structure is different
from Paxos; this makes Raft more understandable than
Paxos and also provides a better foundation for building practical systems. In order to enhance understandability, Raft separates the key elements of consensus, such as
leader election, log replication, and safety, and it enforces
a stronger degree of coherency to reduce the number of
states that must be considered. Results from a user study
demonstrate that Raft is easier for students to learn than
Paxos. Raft also includes a new mechanism for changing
the cluster membership, which uses overlapping majorities to guarantee safety

1. Introduction
Consensus algorithms allow a collection of machines
to work as a coherent group that can survive the failures of some of its members. Because of this, they play a
key role in building reliable large-scale software systems.
Paxos [15, 16] has dominated the discussion of consensus algorithms over the last decade: most implementations
of consensus are based on Paxos or influenced by it, and
Paxos has become the primary vehicle used to teach students about consensus.
Unfortunately, Paxos is quite difficult to understand, in
spite of numerous attempts to make it more approachable.
Furthermore, its architecture requires complex changes
to support practical systems. As a result, both system
builders and students struggle with Paxos.
After struggling with Paxos ourselves, we set out to
find a new consensus algorithm that could provide a better foundation for system building and education. Our approach was unusual in that our primary goal was understandability: could we define a consensus algorithm for
practical systems and describe it in a way that is significantly easier to learn than Paxos? Furthermore, we wanted
the algorithm to facilitate the development of intuitions
that are essential for system builders. It was important not
just for the algorithm to work, but for it to be obvious why
it works.
The result of this work is a consensus algorithm called
Raft. In designing Raft we applied specific techniques to
improve understandability, including decomposition (Raft
separates leader election, log replication, and safety) andstate space reduction (relative to Paxos, Raft reduces the
degree of nondeterminism and the ways servers can be inconsistent with each other). A user study with 43 students
at two universities shows that Raft is significantly easier
to understand than Paxos: after learning both algorithms,
33 of these students were able to answer questions about
Raft better than questions about Paxos.
Raft is similar in many ways to existing consensus algorithms (most notably, Oki and Liskov’s Viewstamped
Replication [29, 22]), but it has several novel features:
• Strong leader: Raft uses a stronger form of leadership than other consensus algorithms. For example,
log entries only flow from the leader to other servers.
This simplifies the management of the replicated log
and makes Raft easier to understand.
• Leader election: Raft uses randomized timers to
elect leaders. This adds only a small amount of
mechanism to the heartbeats already required for any
consensus algorithm, while resolving conflicts simply and rapidly.
• Membership changes: Raft’s mechanism for
changing the set of servers in the cluster uses a new
joint consensus approach where the majorities of
two different configurations overlap during transitions. This allows the cluster to continue operating
normally during configuration changes.
We believe that Raft is superior to Paxos and other consensus algorithms, both for educational purposes and as a
foundation for implementation. It is simpler and more understandable than other algorithms; it is described completely enough to meet the needs of a practical system;
it has several open-source implementations and is used
by several companies; its safety properties have been formally specified and proven; and its efficiency is comparable to other algorithms.
The remainder of the paper introduces the replicated
state machine problem (Section 2), discusses the strengths
and weaknesses of Paxos (Section 3), describes our general approach to understandability (Section 4), presents
the Raft consensus algorithm (Sections 5–8), evaluates
Raft (Section 9), and discusses related work (Section 10).


2 Replicated state machines
Consensus algorithms typically arise in the context of
replicated state machines [37]. In this approach, state machines on a collection of servers compute identical copies
of the same state and can continue operating even if some
of the servers are down. Replicated state machines are