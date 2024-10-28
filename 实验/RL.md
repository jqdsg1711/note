#### 贝尔曼最优公式

Optimal policy

![image-20240422171017615](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240422171017615.png)

#### Bellman optimality equation(BOE)

![image-20240422171241422](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240422171241422.png)







#### Stochastic Approximation（随机近似理论）

##### Motivating examples

![image-20240424134720104](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424134720104.png)

![image-20240424134840440](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424134840440.png)

方法一：等到足够的数据收集

方法二：迭代的方法，来一个算一个

**推导：**

![image-20240424135138688](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424135138688.png)

 ![image-20240424135303571](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424135303571.png)

**算法延申：**

![image-20240424135451142](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424135451142.png)

##### Robbins-Monro algorithm

**介绍：**

![image-20240424135821543](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424135821543.png)

![image-20240424135952032](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424135952032.png)



![image-20240424140133365](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424140133365.png)

**RM算法**

![image-20240424140356023](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424140356023.png)

#### 分析

![image-20240424140443484](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424140443484.png)



![image-20240424140533513](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424140533513.png)

**例子证明RM算法可以收敛**

![image-20240424140840061](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424140840061.png)



![image-20240424140931836](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424140931836.png)



![image-20240424141431750](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424141431750.png)



![image-20240424141526753](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424141526753.png)



![image-20240424141619923](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424141619923.png)



![image-20240424141832936](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424141832936.png)

**如果三个条件有不满足的，RM算法就可能失效**

ak可能取值为某个小的常量，因为它需要把后面进来的数据有效化。

![image-20240424142014414](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424142014414.png)



![image-20240424142237651](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424142237651.png)



![image-20240424142411275](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424142411275.png)









#### Stochastic Gradient Descent（随机梯度下降）



##### 算法介绍

![image-20240424142944690](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424142944690.png)

![image-20240424143330427](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424143330427.png)

![image-20240424143349194](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424143349194.png)

##### 示例和应用

![image-20240424143634829](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424143634829.png)

![image-20240424143918156](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424143918156.png)

##### 收敛性分析

 ![image-20240424144207984](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424144207984.png)

![image-20240424144336836](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424144336836.png)

![image-20240424144414932](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424144414932.png)

![image-20240424144503447](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424144503447.png)

##### 收敛模式

![image-20240424144830025](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424144830025.png)

![image-20240424145013351](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424145013351.png)

![image-20240424145056080](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424145056080.png)

![image-20240424145210433](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424145210433.png)

![image-20240424145412932](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424145412932.png)

##### 确定性形式

![image-20240424145520618](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424145520618.png)

![image-20240424145658701](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424145658701.png)

![image-20240424145908408](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424145908408.png)



##### BGD,MBGD and SGD

![image-20240424150335740](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424150335740.png)

![image-20240424150504604](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424150504604.png)

![image-20240424150725498](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424150725498.png)

![image-20240424150812665](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424150812665.png)

![image-20240424150848933](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424150848933.png)

##### Summary

![image-20240424151042394](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424151042394.png)

#### Temporal-Difference Learning（时序差分方法）

##### Motivating example

![image-20240424164305112](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424164305112.png)



![image-20240424164418681](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424164418681.png)



![image-20240424164518350](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424164518350.png)



![image-20240424164613800](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424164613800.png)

##### TD learning of state value

求解给定策略π的state value

![image-20240424165319352](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424165319352.png)

![image-20240424165503222](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424165503222.png)

![image-20240424165843764](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424165843764.png)

![image-20240424170146166](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424170146166.png)

![image-20240424170537117](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424170537117.png)



![image-20240424170625776](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424170625776.png)

![image-20240424170937586](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424170937586.png)

新的贝尔曼公式：

![image-20240424171144594](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424171144594.png)

用RM算法求解该贝尔曼公式：

通过采样

![image-20240424171308677](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424171308677.png)

![image-20240424171503121](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424171503121.png)

![image-20240424171610233](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424171610233.png)

![image-20240424172006873](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424172006873.png)

![image-20240424172213175](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424172213175.png)

![image-20240424172350562](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424172350562.png)



##### TD learning of action values：Sarsa

![image-20240424172758898](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424172758898.png)

![image-20240424172900082](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424172900082.png)

![image-20240424173107969](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424173107969.png)

![image-20240424173209439](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424173209439.png)

![image-20240424173300880](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424173300880.png)

![image-20240424173641816](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424173641816.png)

举例：

![image-20240424173751154](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424173751154.png)

![image-20240424173949957](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240424173949957.png)





##### TD learning of action values：Expected Sarsa

![image-20240428163419308](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240428163419308.png)

![image-20240428163558557](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240428163558557.png)

##### TD learning of action values：n-step Sarsa

![image-20240428164004482](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240428164004482.png)

![image-20240428164225774](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240428164225774.png)

![image-20240428164657617](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240428164657617.png)



##### TD learning of optimal action values：Q-learning

![image-20240429141132055](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429141132055.png)

![image-20240429141140651](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429141140651.png)

**性质**

![image-20240429141522806](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429141522806.png)



![image-20240429141758786](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429141758786.png)



![image-20240429141911720](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429141911720.png)

 ![image-20240429142332205](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429142332205.png)

![image-20240429142504017](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429142504017.png)

![image-20240429142655103](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429142655103.png)





![image-20240429142831727](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429142831727.png)

![image-20240429143004041](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429143004041.png)

b代表behavior，不需要生成数据，所以使用的greedy策略而不是更具有探索性的ɛ-greedy策略



![image-20240429143836259](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429143836259.png)



![image-20240429143955037](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429143955037.png)



![image-20240429144106868](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429144106868.png)



![image-20240429144215146](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429144215146.png)







##### A unified point of view

![image-20240429144503038](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429144503038.png)

![image-20240429144630706](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240429144630706.png)

##### Summary



#### Value Function Approximation

第一次将神经网络引入强化学习

##### Motivating examples：curve fitting

![image-20240506111513948](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240506111513948.png)

![image-20240506111622259](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240506111622259.png)



![image-20240506111825974](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240506111825974.png)

![image-20240506111949028](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240506111949028.png)

![image-20240506112050852](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240506112050852.png)

![image-20240506112152991](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240506112152991.png)

![image-20240506112253206](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240506112253206.png)



##### Algorithm for state value estimate

![image-20240511155217088](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511155217088.png)

![image-20240511155334701](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511155334701.png)

![image-20240511155441693](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511155441693.png)

实际上不一定所有状态都是平等的，故引入第二个概率分布

dpai扮演的是权重的意思

![image-20240511155726661](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511155726661.png)

Stationary:平稳

![image-20240511155908677](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511155908677.png)

例子：

 ![image-20240511160244197](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511160244197.png)



![image-20240511160419130](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511160419130.png)



###### Objective function

###### Optimization algorithms

![image-20240511162913067](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511162913067.png)

![image-20240511163059293](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511163059293.png)

![image-20240511163157145](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511163157145.png)

![image-20240511163303093](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511163303093.png)

###### Selection of  function approximators(P42)

![image-20240511163632709](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511163632709.png)

![image-20240511163728138](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511163728138.png)

Linear function approximation：

![image-20240511163909437](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511163909437.png)



![image-20240511164109919](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511164109919.png)

![image-20240511164212356](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511164212356.png)







###### Illustrative examples(P42)

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511164342658.png" alt="image-20240511164342658" style="zoom:50%;" />

 ![image-20240511164458540](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511164458540.png)

![image-20240511164553164](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511164553164.png)

![image-20240511164645770](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511164645770.png)

![image-20240511164737810](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511164737810.png)



![image-20240511164836388](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511164836388.png)

![image-20240511164921769](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511164921769.png)

###### Summary of the story

![image-20240511165147333](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511165147333.png)





###### Theoretical analysis

![image-20240511165244359](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511165244359.png)

![image-20240511165334743](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511165334743.png)

![image-20240511165522480](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240511165522480.png)

##### Sarsa with function approximation(P43)

![image-20240513105554909](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513105554909.png)

![image-20240513105626061](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513105626061.png)

![image-20240513105840181](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513105840181.png)



##### Q-learning with function approximation

![image-20240513105956858](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513105956858.png)

![image-20240513110026102](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513110026102.png)

![image-20240513110203653](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513110203653.png)

##### Deep Q-learning（p44-45）

![image-20240513110434533](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513110434533.png)

![image-20240513110602634](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513110602634.png)

把Y中的w当作常数

![image-20240513110804090](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513110804090.png)

先假设前面的Wt保持不动，

![image-20240513110916597](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513110916597.png)

![image-20240513111035408](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513111035408.png)

![image-20240513111309973](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513111309973.png)

![image-20240513111443843](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513111443843.png)

![image-20240513111702962](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513111702962.png)

![image-20240513111800601](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513111800601.png)

![image-20240513112102411](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513112102411.png)

![image-20240513112456500](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513112456500.png)

- off-policy
- 算法比较底层，使用伸进网络得高效性

![image-20240513112846009](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513112846009.png)

![image-20240513113027445](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513113027445.png)

![image-20240513113143338](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513113143338.png)

只需要很少得数据量就可以达到目标



![image-20240513113320214](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513113320214.png)

再强大的算法也需要足够的数据

#### Policy Gradient Methods(策略梯度方法)

##### Basic idea of policy gradient

![image-20240513191415173](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513191415173.png)

![image-20240513191629856](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513191629856.png)

![image-20240513191733782](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513191733782.png)

![image-20240513191812318](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513191812318.png)

![image-20240513191859054](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513191859054.png)

![image-20240513192056532](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240513192056532.png)

##### Metric to define optimal policies

![image-20240514151333821](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514151333821.png)

![image-20240514151410670](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514151410670.png)

![image-20240514151830379](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514151830379.png)

![image-20240514152238766](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514152238766.png)

![image-20240514152529195](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514152529195.png)

![image-20240514152652860](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514152652860.png)

跑无穷多步的 reward平均



S0在无穷多步时，不起任何作用即不重要

![image-20240514152906657](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514152906657.png)



![image-20240514153001312](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514153001312.png)

![image-20240514153125737](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514153125737.png)

![image-20240514153220492](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514153220492.png)



![image-20240514153413229](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514153413229.png)

![image-20240514153546391](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514153546391.png)



##### Gradients of the metrics

![image-20240514153706745](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514153706745.png)

![image-20240514153749628](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514153749628.png)

所有的情况求出来的梯度都是大同小异的，故只给出一个公式

![image-20240514153939448](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514153939448.png)



![image-20240514154036274](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514154036274.png)



![image-20240514154203495](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514154203495.png)

![image-20240514154306892](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514154306892.png)



![image-20240514154518859](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514154518859.png)

h可以用神经网络

![image-20240514154635174](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514154635174.png)

##### Gradient-asent algorithm(REINFORCE)

![image-20240514154810802](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514154810802.png)

![image-20240514154917808](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514154917808.png)

![image-20240514155139642](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514155139642.png)



![image-20240514155317981](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514155317981.png)

优化

![image-20240514155647294](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514155647294.png)



![image-20240514160002251](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514160002251.png)

上面是利用，下面是探索

当分子大时，就会给它更大的权重

分子小时，在迭代后，赋值更大的权重



![image-20240514194850454](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514194850454.png)



![image-20240514195009512](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514195009512.png)



##### summary



#### Actor-Critic方法

![image-20240514202326745](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240514202326745.png)

##### The simplest actor-critic（QAC）

![image-20240515191411533](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240515191411533.png)

qt就是对当前策略的评估，计算当前的action value 



![image-20240515191718293](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240515191718293.png)

![image-20240515192134365](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240515192134365.png)

![image-20240515192644297](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240515192644297.png)

##### Advantage actor-critic（A2C)

![image-20240515192812959](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240515192812959.png)

###### Baseline invariance

![image-20240515193359879](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240515193359879.png)

![image-20240516153935377](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240516153935377.png)

![image-20240516154122577](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240516154122577.png)

![image-20240516154408249](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240516154408249.png)

![image-20240516154505236](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240516154505236.png)

###### The algorithm of advantage actor-critic

![image-20240516154701050](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240516154701050.png)

![image-20240516154942289](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240516154942289.png)

![image-20240516155109641](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240516155109641.png)

![image-20240516155200764](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240516155200764.png)

![image-20240516155525924](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240516155525924.png)

##### off-policy actor-critic

###### Illustrative examples

###### Importance sampling

###### The theorem of off-policy policy gradient

###### The algorithm of off-policy actor-critic

##### Deterministic actor-critic(DPG)

###### The theorem of deterministic policy gradient

###### The algorithm of deterministic actor-critic











