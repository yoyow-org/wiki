
## Introduction

Witnesses can be understood as the "miners" of the YOYOW network. Witnesses need to set up mining machines (cloud server is okay), and pack blocks to get certain YOYOW rewards.

## Witness Qualification

Any account with collateral over a certain amount of YOYO can become a witness. The minimum value for collateral is 10,000 YOYO and the operation fee costs 1000 YOYO so at least 11,000 YOYO are required to become witnesses.


## Witness Election Details

In the YOYOW network, there are 2 ways of getting chances to produce blocks among witnesses.

* Getting votes: The more votes you get, the more chances to produce blocks you will have. The top X witnesses with the highest number of votes are the "top witnesses" of each round; all other witnesses after the X witness are selected for the Y  "standby witnesses" according to the number of votes.

> Note：The probability of the witnesses outside the X being elected as a "standby witness" is proportional to the number of **valid votes**.

* Collateral: The more YOYO you deposit as collateral, the more opportunities you have to produce blocks. Z "miner witnesses" will be selected in each round.

> Note: The probability of being elected as a "miner witness" is proportional to the **valid collateral**.

At present, X = 11, Y=3, Z=7, which means: 21 witnesses are elected in each round, including 11 top witnesses, 3 standby witnesses and 7 miner witnesses.

In this cycle, after each block-production is completed, re-election will be carried out as above and will be repeated.

## Voting
* Currently, each person can support 101 witnesses at the same time.
* If you do not want to vote, you can specify a proxy.

## Collateral
* Witnesses can adjust the amount of collateral at any time.
* When the collateral is lowered, a certain delay will be required for the arrival. Current delay time: 56 hours.


## Rewards
* Top witnesses' rewards for each block: 0.3 YOYO
* Standby witnesses' rewards for each block: 0.3 YOYO
* Miner witnesses' rewards for each block: 0.5 YOYO

Witnesses can withdraw "available" rewards into their balance at any time.

## Advanced FAQ
Can a person vote after depositing collateral?
> Yes

How many votes can each person vote for at the same time?
> 101, the parameter is `max_witnesses_voted_per_account`

What are the so-called **valid votes** and **valid collateral**?
> The calculation methods of these two are similar. It is the smaller value of the "average value over the past 7 days" and the "current collateral amount".

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

What is the current annualized rate of returns of YOYO witnesses?
> Since customer collateral, voting rates, etc. are different in different time periods, the annualized rate of returns changes in real time. In a rough estimate, the annualized rate of returns from collateral and voting is 10%.

What are the activating conditions for **valid votes**?
> When the vote holder has not started voting, it does not start to calculate the number of **valid votes**. When the vote holder starts voting, it will gradually accumulate **valid votes**.
