---
layout: post
title:  "数据分析之全行为路径分析模型"
date:   2019-07-07 00:18:23 +0700
categories: [bigdata]
---

#### 概述
  用户在产品中的行为其实是个黑盒子，全行为路径是用全局视野看用户的行为轨迹，很多时候你会有意想不到的收获，在可视化的过程中有两个模型，一个是树形图、一个是太阳图  
  
#### 行为路径分析  
  单体洞察、用户分群、行为路径分析是用户行为数据分析的三大利器。单体洞察满足了我们对单个用户的特征洞察，用户分群满足了我们对全量用户或某一特征人群的洞察，而行为路径分析是对用户产生的行为数据的可视化分析模型，某一人群交叉行为路径分析模型，可以快速洞察到这一群体的行为特征。常用的行为路径分析模型有漏斗分析模型和全行为路径分析模型。  
  在分析既定的行为路径转化时，我们会采用漏斗分析模型，你会看到用户在我们设定的路径中的每一步转化，比如从查看商品详情到最终支付成功每一步的转化率，从而对既定路径不断调优。  
  
  ![collect_model_7_1](/static/img/post/b_a_m_7_1.jpg){:height="320" width="720"} 
  
  
  但是，用户在产品内的行为路径可以说是个黑盒子，界面内的每一个按钮、信息都会影响用户的下一行为。为此，我们需要拥有一个更高的视野去俯视用户的行为，打开这个黑盒子，而这一分析模型就是全行为路径分析模型。  
  
#### 全行为路径分析模型  
  全行为路径分析是互联网产品特有的一类数据分析方法，它主要根据每位用户在App或网站中的行为事件，分析用户在App或网站中各个模块的流转规律与特点，挖掘用户的访问或浏览模式，进而实现一些特定的业务用途，如对App核心模块的到达率提升、特定用户群体的主流路径提取与浏览特征刻画，App产品设计的优化等。  
  常用的全行为路径分析模型有两种：  
  1、树形图  
  
   ![collect_model_7_2](/static/img/post/b_a_m_7_2.jpg){:height="320" width="720"} 
  
  图2：树形图  
  如上图所示，从会话开始，每一行代表用户的一步。树形图最多展示五步。第一步是会话开始，第二步，用户通常会进行搜索课程、查看课程详情、注册、登录、开始付款。从上图可以看出，在用户的使用中，绝大部分的用户打开app后会进行课程搜索。你可能会问横向相加为什么不等于100%？  
  如图示，转化率计算的是用户的每一次会话，同一个用户，可能上午进入app后进行了搜索，下午可能进入app后直接在首页进行了查看课程详情，同一个人在不同会话可能会有不同的行为。分母是所有使用的用户，那计算每一步的时候分子会出现同一个人。所以百分比相加大于1。  
  2、太阳图 树状图通过用户行为的步骤纵向进行了展示并基于每一步的比例进行了从高到底的排序，相较于树状图，太阳图的全局视野更清晰，你可以用一个平面的视角看用户的使用情况。  
  
  ![collect_model_7_3](/static/img/post/b_a_m_7_3.jpg){:height="320" width="720"} 
  
  图3：太阳图 如上图所示，每一环代表用户的一步，不同的颜色代表不同的行为，同一环颜色占比越大代表在当前步骤中用户行为越统一，环越长说明用户的行为路径越长。  
  你可以把路径设计过程中我们忽略的步骤添加到漏斗进行监控，并对用户的这一路径做用户动机分析，并不断进行优化   
  
#### 场景举例  
  我们以上图中的树形图为例，这是一个教育培训类产品，我们发现75.2%的用户都是从“搜索课程”这一行为开始的，说明“搜索”是这一产品的重要功能之一，搜索优化得越好，购买下单的可能性就越大，同时有助于了解用户的真实需求。  
  但是我们还发现，从第二步之后的数据来看，一次的搜索行为显然没有帮用户找到他所需要的课程，因为，他并没有直接进入“查看课程详情”。  
  对于用户来说最理想的体验是，在输入关键词后，快速找到其所需要的商品/课程/服务，对于产品来说，就是在用户还没有失去耐心前完成搜索转化，那么针对上图的场景，我们该如何提升一次搜索转化率呢？  
  除了识别不利于转化的关键词，通过放置搜索结果顶端或者底部来升级或降级产品外，你还可通过洞察用户行为数据来优化：  
  比如：凡搜索“数据分析”、“数据驱动”、“数据思维”等关键词的用户最终都点开查看了A课程，那么我们即可根据数据相关性将搜索词“数据”与A课程关联到一起。  
  比如：将近期用户高频搜索的关键词同步到前端页面，设置成可点击元素，提高搜索效率  
  比如：通过分析用户的搜索行为，为用户补充商品/课程/服务、优化搜索结果页结构、优化搜索推荐等提供数据支持。  
  总之，透过用户行为数据深挖用户表面行为背后真实、本质的需求，唯有通过数据看透本质需求，才能真正触达用户的“心”。  
  用户运营的本质是精细化运营，而精细化运营的前提是，对可真实还原用户与产品交互过程的用户行为的洞察，全行为路径分析让你更直观的看到用户使用产品的状况，了解用户的来龙去脉，找到用户最有可能完成核心转化的行为，通过产品上以及运营策略上的引导，持续挖掘更多用户的价值。  
  
  
  
  
  
