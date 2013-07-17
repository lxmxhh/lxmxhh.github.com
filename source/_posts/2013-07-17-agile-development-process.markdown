---
layout: post
title: "敏捷开发过程剖析及工具推荐"
date: 2013-07-17 22:48
comments: true
categories: 
---
敏捷开发，要求在开发过程中不断增强，从而提高软件质量，以达到提高商业收入的目的。它是一个迭代的过程，一个不断提高企业投资回报率和服务质量的过程。值得注意的是，成功的敏捷开发，单纯依附于活跃的开发过程和理解敏捷所带来的效益并对此有浓厚兴趣的企业用户。 

本文将介绍敏捷开发的五大过程及这些过程中所要用到的工具。 

{% img center http://placekitten.com/500/160 %} 
<!--more-->

1. **敏捷计划** 

	典型的敏捷开发将整体工作分为一系列的发布过程，每个发布过程都是一个迭代循环，每个迭代循环都会发布一组功能特性。 

	敏捷计划规定了每个循环中所需要完成的工作（发布/迭代）。在该阶段，产品所有者将描述每个循环过程中他希望看到的产品样子。 

	敏捷计划包含发布计划与迭代计划，两者的内容及执行者不同： 

	**发布计划：**包含每次发布的功能组。产品所有者负责在产品发布之前制定发布计划。  
	**迭代计划：**开发团队需要在开发工作及迭代开始前确定需要完成的工作。可以通过每天的站立会议来实现。  
	**工具：**制定敏捷计划，有很多工具可以使用，如：   
	[XPlanner] [url1]  
	[Mingle] [url2]  
	[Extreme Planner] [url3]

[url1]:http://www.xplanner.org/
[url2]:http://www.xplanner.org/
[url3]:http://www.extremeplanner.com/  



2. **创建用户故事** 

	用户故事，是对功能、特性的简单描述。每个特性也可能由很多故事组成。用户故事要简单且容易理解，能在几分钟内通过几行字表述清楚。请注意，用户故事是由项目所有者或主要用户群体来定义的，而非开发者。 

	正如Mike Cohnrn所建议的，用户故事应该遵循下面的格式：  

	*   引用  
    `作为一个（某种角色），我需要（某事）如此如此。`

	例如，作为一个用户，我希望通过姓名来查找我的客户。 

	**工具：**最好的方法是使用索引卡片来记录各个故事。有很多种工具可以帮助完成故事图谱与故事追踪，如   
	[SilverStories] [url4]  
	[Pivotaltracker] [url5]  

[url4]:http://toolsforagile.com/silverstories/
[url5]:http://www.pivotaltracker.com/

	**注意：**故事并不是一次性完成的，它循环往复，且贯穿于整个项目开发周期中。   

3. **评估你的工作** 

	在敏捷中，评估用于预测功能实现的复杂程度，并根据以前完成相似复杂度功能的经验预估所需要的完成时间。它是一个持续的过程，基于之前的经验和模式学习，不断提高评估的准确性。 

	通常，评估故事的复杂程度多基于故事要点，而非所耗费的时间。要点解释了故事的复杂性，并通过数据1，2，3……来体现。 

	评估有助于做出更好的商业决策，定义发布/迭代的范围。例如，我们可以很容易地为每次迭代/发布中的所有故事分配同样的数字。 

	**工具：**[Planning Poker][url6]是定义和改善你评估的最好技术。  

	![agile-estimation-chat](http://techmytalk.files.wordpress.com/2013/07/agile-estimation-chat.jpg) 

[url6]:http://www.planningpoker.com/

4.  **站立会议** 

	站立会议是开发团队每天进行的简短会议。会上每个人需要说明昨天所完成的事，及今天的计划和被分配任务现在的状态。商业用户和领域专家偶尔也会参加，这将给他们更多关于项目的信心。 

	它不是例行会议，仅仅对项目实施情况给出粗略的描述，而是要提供更多关于项目的可视性内容，增强团队间的协作，对当天的计划给出正确指导。 

	**工具：**在站立会议中，白板是非常有效的工具。 

5.  **项目监控技术** 

	**速率：** 

	通过速率，可以精确地测量开发团队发布商业价值的速度。速率是对生产力的测量。通过计算一定间隔内完成工作的单元数来计算速率。 

	在每次迭代的最后，为了计算速率，敏捷团队会查看该过程所完成的工作要求，并累加与这些要求相关联的故事点。所完成故事点的总数便是团队的速率。首次小小的迭代之后，你会逐渐发现某种趋势，且能计算出平均速率。 

	下面一些工具可以帮助追踪速率。 

	[TargetProcess] [url7]  
	[Pivotaltracker] [url5]  
	[Timetracking] [url9]  
	[VersionOne] [url10]  

	**Burndown Reports：** 

	Burndown  Report是追踪项目进度的另一个标尺。它用来追踪完成故事点的个数，监控简单的迭代、发布和整个项目积压的工作。它可以显示进度，反映产品交付的价值和团队的速率。 

	以下一些工具可用于测量Burndown Reports： 

	[TargetProcess] [url7]  
	[XPlanner] [url1]  
	[Pivotal Tracker] [url5]  
	[ScrumWorks] [url11]


原文来自：[TechMyTalk][url12] / 译：[CSDN][url8]

[url7]:http://www.targetprocess.com/
[url8]:http://www.csdn.net/article/2013-07-16/2816244
[url9]:http://slimtimer.com/
[url10]:http://www.versionone.com/
[url11]:http://danube.com/scrumworks/basic
[url12]:http://techmytalk.com/2013/07/14/agile-software-development-process/