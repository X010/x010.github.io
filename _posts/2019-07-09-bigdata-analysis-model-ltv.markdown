---
layout: post
title:  "数据分析之LTV"
date:   2019-07-09 00:18:23 +0700
categories: [bigdata]
---

#### 概述
  LTV(life time value)生命周期总价值，意为客户终生价值，是公司从用户所有的互动中所得到的全部经济收益的总和。通常被应用于市场营销领域，用于衡量企业客户对企业所产生的价值，被定为企业是否能够取得高利润的重要参考指标。  
  
#### LTV定义  
  LTV（life time value）也就是用户生命周期价值，是产品从用户获取到流失所得到的全部收益的总和。LTV用于衡量用户对产品所产生的价值，是所有用户运营手段为了改善的终极指标，同时LTV也应该是所有运营手段的最终衡量指标。以用户获取为例，一个用户获取渠道的新客成本是否昂贵，并不仅仅取决于这个新客成本的绝对值的高低，还取决于获取到的用户LTV是多少。同样一个产品，A渠道的新客获取成本是150元，B渠道的新客获取成本是300元，直观地感受A渠道效果更好。但是如果后续追踪LTV之后，A渠道的用户平均LTV是100元，B渠道用户平均LTV是400元。在考虑LTV之后发现，A渠道每个新客亏损50元，B渠道赢利100元，虽然B渠道新客价格更贵，但是B渠道更加有效。   
  不管是用户获取、留存还是唤醒，需要投入多少资源，都可以用LTV来进行衡量。统一成公式就是如下所示：  
  
```aidl
rate=LTV/cost
```
  
  
  其中rate代表投入产出比， 代表用户运营活动使得LTV增加的量，cost代表运营活动投入的成本。  
  当投入产出比大于1的时候，则代表这次活动就是有正收益的。这是一个非常简单的公式，但是在实际的用户运营中，却很少用到，除了LTV概念没有深度普及之外，还有一个关键的原因就是，生命周期价值的提升难以在短时间内衡量。实际上这个问题也并非没有解决办法。下面我们就会提到如何计算LTV，从而提升LTV的投入产出比。  
  
  
#### LTV的计算
  在网上目前会看到一些比较通识性的LTV计算方法，使用MMR代表每月用户用户给平台带来的收入，churn rate代表用户的月流失率，那么LTV的计算方法如下所示：  
  
```aidl
LTV=MMR/churn rate
```
  这种简单的计算方法隐含了两个假设：用户结构稳定不变、用户质量稳定不变。这两个假设就意味着，新用户的质量总是长期稳定不变，不管从什么渠道获取到的用户都有一样的流失率和收入情况，同时产品的用户规模不会出现比较大的波动。显然这些假设在实际中就是不存在的。用这种方式计算的LTV仅仅能作为一个宏观数据的参考，并不能真正指导业务。  
  那么，什么样的LTV计算才是有价值的呢？结合我们提到的LTV的应用场景，就是要能够计算用户运营活动的投入产出比。不管是拉新、留存还是召回，本质上都是针对不同用户的活动，每次活动的成本是可以计算的，那么为了计算运营活动的投入产出比，这就意味着需要尽快检测出来不同维度的用户群的LTV变化。  
  要精确的计算每个用户的LTV，意味着需要等用户流失之后才能知道LTV的精确值，这个过程短则几个月，长则数年。显然用户运营活动显然不可能等比较长时间之后，才去看这个精确的LTV结果。为了能及时计算LTV的变化，就需要用一些回归或者预测类的算法。比如最典型的新用户获取问题，一般投放的BD衡量一个渠道的好坏，除了看新客成本，还通过一些短期数据来简单判断某个渠道内用户的整体质量如何以及将来的赢利能力如何。这些短期数据包括新客的次日留存，7日留存，30日留存这些留存数据，以及7日消费额，30日消费额等营收数据。既然BD可以用这些数据可以做出基于人工经验的判断，那么就意味着短期数据中有足够预测用户长期LTV的信息量。  
  相比于人工考虑的短期数据，数据系统记录用户短期内使用产品的全部行为数据包含了更大的信息量，用这些数据作为入参，可以更好的更好地预测LTV的结果，进而检测LTV的变化。利用历史上用户行为数据以及最终的LTV情况作为训练数据，利用用户行为数据中的多个维度的特征作为入参，可以做出准确率相对比较高的LTV预测模型。这其中无论是使用决策树、回归算法还是神经网络，只要数据量满足，预测的准确率是可以基本保证的。一旦拥有了这样的LTV预测模型，那么用户运营的结果就可以有效的监控起来。  
  虽然机器学习关于预测的算法已经非常成熟且越来越普及，但是确实也不是所有的公司都具有开发预测模型的能力。在不具有开发预测模型的能力的情况下，负责用户运营的同学也可以根据历史上用户的短期留存率和短期营收数据作为入参，拟合出来粗略的LTV计算公式。作为用户运营的基础数据模型。  
  
#### LTV的应用  
  当LTV的数据计算方法被各方认同之后，利用LTV可以做用户运营效果的检测，并沉淀为后续用户运营活动的经验。  
  在用户获取和用户召回的时候，利用不同渠道获取到了用户的短期行为数据作为基础预测出的LTV。在计算出LTV后，就可以同时综合考虑投放成本，确定不同渠道的价值，从而确定怎样的投放组合在用户获取中是最高效的方法。在用户留存时，不论是做活动还是发送优惠券，都需要衡量这些用户运营活动之后LTV的预测值的变化。根据LTV预测值的提升结果，可以了解到不同活效果的好坏，从而总结后续以留存为目的的运营活动到底该如何改进。  
  不仅仅日常的用户运营活动需要看LTV，一些特殊阶段也不例外，比如早期增长或者产所在行业面临激烈竞争的时候。在这些特殊阶段，团队决策层的注意力可能会仅仅放在用户运营结果的绝对量上，比如活跃用户数，新增用户数。即使在这种情况下，用户运营的投入产出比可以为负数，单并不意味这LTV可以放弃去考虑。资源有限的情况下，总是找到最优解。使用LTV来提前预估不同的投资组合的效果之后，在产品早期或者竞争期会更有优势。  
  
#### LTV与CAC  
  一个用户从注册到卸载，整个生命周期为你带来的全部利润（注意是利润不是收入），是LTV（Life Time Value）；而获取一名用户的成本，是CAC（Customer Acquisition Cost）。可以说，LTV与CAC，是衡量一门生意的标尺。  
  生意分为两种：  
  （1）LTV>CAC  
  这类产品，一般有着非常明确的变现方式，这时运营只需要持续不断的洗入用户就可以了，越多越好。  
  （2） LTV<CAC  
  短期内无法盈利，一般缺乏明确的变现方式。但由于移动互联网的特性（人人皆自媒体），只要产品是真的解决了某个问题。那么一旦用户达到一定规模，比如几十万、几百万的真实注册用户、甚至活跃用户，就一定可以实现自增长。所以此时的推广只有一个目的，就是更快的达到这个点。在这个时候，评判推广的最好标准，就是是不是有利于你更快的达到这个点，像刚才说的在线教育的case，就对这个点并没有什么帮助。  
  
#### LTV与CAC应用例子  
  对于一门生意，我们可以从产品模式、市场环境和用户特点三个方向去把握它。我们举一个LTV>CAC的case里，相对简单的一款产品：积分墙。  
  
  ![collect_model_ltv_1](/static/img/post/ltv_1.jpg){:height="320" width="720"} 
  
  （1）产品模式：  
  积分墙的生意很简单。广告主（CP）需要下载量；生活中有很多人有零碎时间想赚点钱。但是广告主直接找人，比如论坛发帖：谁来下载我一个游戏，给你2块钱，一是操作成本太高，二是不具有可持续性。  
  于是就有了垂直积分墙，连接广告主和用户。积分墙作为执行方，承接CP的需求。之后通过任务列表的方式展现给用户。用户从上面领取任务，完成下载、注册、试玩等任务，然后拿到积分。积分可以兑换到支付宝或者微信里。  
  同时，大规模的下载行为，对安卓来说，主要的意义在于做数据（为了钱去下载你应用的人，后续留存都会非常差），对iOS则有着更深的意义。因为App Store的闭环生态，CP能做的事情不多。大规模下载可以改变应用在免费榜、付费榜、或者关键字榜单的排名，可以通过积分墙影响榜单，让应用获得自然的下载。  
  （2）市场环境  
  2014和2015年积分墙很火，原因主要是以下两点：  
  1）移动互联网突飞猛进，热钱多；  
  没融到钱的，要做数据融钱；融到了的，要做数据给市场，给竞争对手看。而积分墙，是最廉价的起量平台。   
  2）用户有好奇心。  
  那个时候的用户，自己就会去App Store总榜去看看，有什么自己没玩过的应用，下载看看好不好玩。在这种情况下，用户可用的资源，即任务，分为大型应用（BAT旗下）和小型应用两种。大型应用对量的需求持续的。小型应用则是偶发性的。  
  这样，一个用户进入的时候，可以在很短的时间内，完成几个、十几个大型应用的任务。之后则是偶尔能拿到小型任务。  
  （3）用户特点：  
  用户完成1个任务，大概花费5分钟，一般会得到1-2元奖励。这样，一个新用户可能在1-2个小时内，拿到10-30元奖励。之后，可能1、2天能拿到1元奖励。  
  平台对于用户的价值很简单，做任务赚钱。那么什么样的用户是垂直积分墙的目标用户呢？    
  - 缺钱。缺钱的人才会拿1块钱当钱；  
  - 无聊。有大量闲散时间的人才会花一个小时下载App赚10元钱。  
  两个维度交织在一起，就是我们的目标用户群:  
  1）他们应该是三线城市用户，或者1、2线里没钱的年轻人  
  原因：三线城市物价低，1元钱是钱；三线城市的工作一般较清闲，但缺乏娱乐方式，所以无聊时间较多。  
  2）垂直用户群包括：  
  - 孕妇宝妈（女性，更热衷于省钱；行动不便，宝妈要照顾孩子，时间零碎且无聊）  
  - 工厂工人（收入低；工作极其无聊，晚上也缺乏娱乐方式）  
  - 三线城市学生（缺钱；无聊，除了打游戏没别的消遣）  
  上面三个用户群匹配度依次下降。  
  之后的用户调研和推广经验，证明我们对用户的定位是比较准确的。  
  好，在明确上面三个点之后，我们来推演一下这门生意。  
  假设获取一个积分墙用户的成本是8元。前面说过，垂直积分墙的任务，一般是由若干稳定的大型应用和不定期的小型应用构成的。用户注册第一天，就可以完成十个左右的大型应用的任务。   
  假设每个任务给平台贡献1元利润，此时LTV已经大于CAC了。之后可能偶尔做一个任务，贡献1元。在任务少到超过他的容忍限度时，他流失了。  
  很明显，积分墙满足第一类生意，变现途径异常清晰简洁。所以我们看到，2015年下半年，垂直积分墙经历了一个井喷式的增长，市场里多了很多玩家。但是从2016年Q2开始，积分墙走向萎缩。这个生意不太好做了。  
  因为市场环境在变化。  
  首先热钱少了，改试创业方向的都试的差不多了，连模式很重的O2O都试的差不多了。没有明确变现方案的产品，风投看都不想看了。所以你看，今年直播火了。  
  另外，用户的使用习惯有了很大改变。看看现在周围的人，一个月能更新俩App就不错了。包括App Store的关键字榜单，大家都觉得没有去年效果好了，不吸量！过去可能我搜旅游，看看有啥好App，“哎，有个途牛看着还行，下载下来看看。”现在，是我知道我要下途牛，然后我去了App Store。即使搜索结果第一个是去哪儿，第二个是携程，第三个才是途牛，难道我会去下去哪儿和携程么？  
  
  ![collect_model_ltv_2](/static/img/post/ltv_2.jpg){:height="320" width="720"} 
  
  
  市场变化，导致预算多多的大型应用，一方面数量减少，一方面削减这方面的预算。而新的App本来就少了，预算使用也更加谨慎。这造成了什么呢？就是积分墙的用户上来，发现只有5个应用可以做，做完之后苦等三四天，才又领到一个任务；然后又苦等几天，再领一个。请问会有人为了三天赚1元钱，而留着你这个应用吗？没意思嘛。  
  这造成了两个现象：  
  1.新用户对平台的贡献大大减少；
  2.用户的生命周期大大缩短，最终的结果，就是LTV的下降。  
  但市场本身的流量在变得昂贵，因为经历了井喷式增长，竞争者非常多，而渠道就那么几个，用户被洗来洗去，这就使得CAC不降反增。  
  最终导致CAC>LTV，这门生意自然也就很难做了  
  所以，我们在运营和推广一款产品的时候，一定要对产品的运作模式、市场环境和用户特点有着非常清晰的认识，这三者相互依存，相互影响，共同决定了两个标尺CAC和LTV的变化。  
