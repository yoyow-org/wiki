
## 简介
## Introduction
见证人可以理解为YOYOW网络的『矿工』，见证人需要架设矿机（云服务器即可），打包区块，来获得一定的YOYOW奖励。

Witnesses can be understood as the "miners" of the YOYOW network. Witnesses need to set up mining machines (cloud server is okay), and pack blocks to get certain YOYOW rewards.

## 见证人资格
## Witness Qualification

任意账户抵押一定数量的YOYO ，即可成为见证人。
抵押有最少为10000YOYO， 此外此操作手续费为1000YOYO，因此要想成为见证人，需准备大于11000YOYO。

Any account with collateral over a certain amount of YOYO can become a witness. The minimum value for collateral is 10,000 YOYO and the operation fee costs 1000 YOYO so at least 11,000 YOYO are required to become witnesses.


## 见证人选举细节
## Witness Election Details

在YOYOW网络，见证人之间共有2种方式获得出块机会：

In the YOYOW network, there are 2 ways of getting chances to produce blocks among witnesses.

* 获得投票： 获得的投票支持越多，就有更多的出块机会。票数最多的前X名，是每一轮的『主力见证人』；X名之后的其他所有见证人，根据得票数来竞选Y名『备选见证人』。
* Getting votes: The more votes you get, the more chances to produce blocks you will have. The top X witnesses with the highest number of votes are the "top witnesses" of each round; all other witnesses after the X witness are selected for the Y  "standby witnesses" according to the number of votes.

> 注： X名之外的见证人，当选『备选见证人』的概率正比与**有效得票数**

> Note：The probability of the witnesses outside the X being elected as a "standby witness" is proportional to the number of **valid votes**.

* 进行抵押： 自身抵押的YOYO越多，就有更多的出块机会。每轮会选出Z名『矿工见证人』

* Collateral: The more YOYO you deposit as collateral, the more opportunities you have to produce blocks. Z "miner witnesses" will be selected in each round.

> 注： 当选『矿工见证人』的概率正比与**有效抵押**

> Note: The probability of being elected as a "miner witness" is proportional to the **valid collateral**.

目前 X = 11, Y=3 ,Z=7 ，意味着：每轮共选举出21名见证人，其中主力见证人11名，备选见证人3名，矿工见证人7名。

At present, X = 11, Y=3, Z=7, which means: 21 witnesses are elected in each round, including 11 top witnesses, 3 standby witnesses and 7 miner witnesses.

该轮次中，每位都出块完毕后，会按如上方式重新选举，周而复始。

In this cycle, after each block-production is completed, re-election will be carried out as above and will be repeated.

## 投票
## Voting
* 当前，每人可同时支持101位见证人
* Currently, each person can support 101 witnesses at the same time.
* 若自己不想投票，可指定代理人
* If you do not want to vote, you can specify a proxy.

## 抵押
## Collateral
* 见证人可随时调整抵押金额
* Witnesses can adjust the amount of collateral at any time.
* 降低抵押时，需一定的延迟才会到账。当前延迟：56小时
* When the collateral is lowered, a certain delay will be required for the arrival. Current delay time: 56 hours.


## 奖励
## Rewards
* 主力见证人每块工资： 0.3 YOYO
* 备选见证人每块工资： 0.3 YOYO
* 矿工见证人每块工资： 0.5 YOYO

* Top witnesses' rewards for each block: 0.3 YOYO
* Standby witnesses' rewards for each block: 0.3 YOYO
* Miner witnesses' rewards for each block: 0.5 YOYO

见证人可以随时将 待领取的工资，领到自己钱包余额内

Witnesses can withdraw "available" rewards into their balance at any time.

## 进阶FAQ
## Advanced FAQ

抵押了还可以投票吗？
> 可以

Can a person vote after depositing collateral?
> Yes

每人最多同时支持多少张票？
> 101 ， 参数为`max_witnesses_voted_per_account`

How many votes can each person vote for at the same time?
> 101, the parameter is `max_witnesses_voted_per_account`

什么是所谓的**有效票数** ， **有效抵押**？

What are the so-called **valid votes** and **valid collateral**?

> 这两者计算方法是类似的。为『过去7天的平均值』与『当前抵押量』两者的较小值。

> The calculation methods of these two are similar. It is the smaller value of the "average value over the past 7 days" and the "current collateral amount".

> 如A账号，8月1日0时抵押了70000 个YOYO,8月11日0时撤销了50000个YOYO ，那么，他每天0时有效抵押量大约为：

> For example, the account A has a collateral of 70,000 YOYO at 0:00 on August 1 and revokes 50,000 YOYO at 0:00 on August 11. Then, his valid collateral at 0:00 every day is:

Time（0:00 every day） | Valid collateral  |
-----------|-------|
August 1st | 0    |
August 2nd | 10000|
August 3rd | 20000|
August 4th | 30000|
August 5th | 40000|
August 6th | 50000|
August 7th | 60000|
August 8th | 70000|
August 9th | 70000|
August 10th | 70000|
August 11th | 20000|
August 12th | 20000|

当前YOYO见证人年化收益率多少？

What is the current annualized rate of returns of YOYO witnesses?

> 由于不同时间段用户抵押数，投票率等等不同，年化收益率在实时变化。粗略估计来讲，抵押并投票的收益在年化10%

> Since customer collateral, voting rates, etc. are different in different time periods, the annualized rate of returns changes in real time. In a rough estimate, the annualized rate of returns from collateral and voting is 10%.

**有效票数**的激活条件是什么？

What are the activating conditions for **valid votes**?

> 持票人未开始投票的时候，并未开始计算**有效票数**，当持票人开始投票后，才会逐渐积累**有效票数** 

> When the vote holder has not started voting, it does not start to calculate the number of **valid votes**. When the vote holder starts voting, it will gradually accumulate **valid votes**.
