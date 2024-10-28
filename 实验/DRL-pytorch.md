### REINFORCE

#### CartPole 环境

以图 7-1 中所示的所示的车杆（[CartPole](https://github.com/openai/gym/wiki/CartPole-v0)）环境为例，它的状态值就是连续的，动作值是离散的。

![img](https://hrl.boyuai.com/static/cartpole.e4a03ca5.gif)

图 7-1 CartPole环境示意图

在车杆环境中，有一辆小车，智能体的任务是通过左右移动保持车上的杆竖直，若杆的倾斜度数过大，或者车子离初始位置左右的偏离程度过大，或者坚持时间到达 200 帧，则游戏结束。智能体的状态是一个维数为 4 的向量，每一维都是连续的，其动作是离散的，动作空间大小为 2，详情参见表 7-1 和表 7-2。在游戏中每坚持一帧，智能体能获得分数为 1 的奖励，坚持时间越长，则最后的分数越高，坚持 200 帧即可获得最高的分数。

表 7-1 CartPole环境的状态空间

| 维度 | 意义         | 最小值   | 最大值  |
| ---- | ------------ | -------- | ------- |
| 0    | 车的位置     | -2.4     | 2.4     |
| 1    | 车的速度     | -Inf     | Inf     |
| 2    | 杆的角度     | ~ -41.8° | ~ 41.8° |
| 3    | 杆尖端的速度 | -Inf     | Inf     |

表7-2 CartPole环境的动作空间

| 标号 | 动作         |
| ---- | ------------ |
| 0    | 向左移动小车 |
| 1    | 向右移动小车 |



![image-20240624095207476](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240624095207476.png)



这是一个基于PyTorch的神经网络模型,名为"Policy"。这个模型用于学习如何输出一个概率分布,代表智能体在给定状态下采取不同动作的概率。我来逐步解释代码:

```python
class Policy(nn.Module):
    def __init__(self, state_dim, action_dim, hidden_width):
        super(Policy, self).__init__()
        self.l1 = nn.Linear(state_dim, hidden_width)
        self.l2 = nn.Linear(hidden_width, action_dim)

    def forward(self, s):
        s = F.relu(self.l1(s))
        a_prob = F.softmax(self.l2(s), dim=1)
        return a_prob

```



1. `__init__(self, state_dim, action_dim, hidden_width)`:
   - 这是类的构造函数,用于初始化模型参数。
   - `state_dim`表示状态的维度大小,`action_dim`表示动作的维度大小,`hidden_width`表示隐藏层的宽度。
   - 在构造函数中,我们定义了两个全连接层:
     - `self.l1`是输入状态到隐藏层的全连接层。
     - `self.l2`是从隐藏层到输出动作概率的全连接层。

2. `forward(self, s)`:
   - 这是模型的前向传播函数,接收状态`s`作为输入,输出动作概率分布。
   - 首先,我们使用ReLU激活函数处理第一个全连接层的输出。
   - 然后,我们使用Softmax函数处理第二个全连接层的输出,得到一个概率分布。
   - 最终,我们返回这个动作概率分布。

总的来说,这个"Policy"模型是一个简单的神经网络,用于根据输入的状态预测智能体在该状态下采取不同动作的概率分布。这种模型通常用于强化学习算法,如策略梯度法,来学习最优的行为策略。



```py
class REINFORCE(object):
    def __init__(self, state_dim, action_dim):
        self.state_dim = state_dim
        self.action_dim = action_dim
        self.hidden_width = 64  # 隐藏层神经元数量
        self.lr = 4e-4  # 学习率
        self.GAMMA = 0.99  # 折扣因子
        self.episode_s, self.episode_a, self.episode_r = [], [], []

        self.policy = Policy(state_dim, action_dim, self.hidden_width)
        self.policy_optimizer = torch.optim.Adam(self.policy.parameters(), lr=self.lr)

```

这段代码定义了一个REINFORCE类,它是一个强化学习算法的实现。

1. `__init__(self, state_dim, action_dim)`: 这是类的构造函数,用于初始化类的属性。
   - `state_dim`: 状态空间的维度
   - `action_dim`: 动作空间的维度
   - `hidden_width`: 策略网络的隐藏层神经元数量,设置为64
   - `lr`: 策略网络的学习率,设置为4e-4
   - `GAMMA`: 折扣因子,设置为0.99
   - `episode_s`, `episode_a`, `episode_r`: 用于存储单个episode中的状态、动作和奖励

2. `self.policy = Policy(state_dim, action_dim, self.hidden_width)`: 创建一个Policy类的实例,它是一个神经网络模型,用于表示智能体的策略。
3. `self.policy_optimizer = torch.optim.Adam(self.policy.parameters(), lr=self.lr)`: 使用Adam优化算法创建一个优化器,用于更新策略网络的参数。

总的来说,这个REINFORCE类定义了一个强化学习算法的基本结构,包括策略网络的定义、优化器的设置,以及用于存储单个episode数据的变量。这个类可以作为强化学习任务的基础框架,需要根据具体的问题和任务进行进一步的实现和扩展。



```python
    def choose_action(self, s, deterministic):
        s = torch.unsqueeze(torch.tensor(s, dtype=torch.float), 0)
        prob_weights = self.policy(s).detach().numpy().flatten()  # 转换为概率分布（numpy）
        if deterministic:  # 评估时使用确定性策略
            a = np.argmax(prob_weights)  # 选择概率最大的动作
            return a
        else:  # 训练时使用随机策略
            a = np.random.choice(range(self.action_dim), p=prob_weights)  # 根据概率分布采样动作
            return a

```

```python
    def store(self, s, a, r):
        self.episode_s.append(s)
        self.episode_a.append(a)
        self.episode_r.append(r)

```

这段代码是用于保存强化学习过程中的经验数据的函数。具体解释如下:

1. `self.episode_s.append(s)`: 将当前状态 `s` 追加到 `self.episode_s` 列表中,用于记录整个强化学习过程中的状态序列。

2. `self.episode_a.append(a)`: 将当前采取的行动 `a` 追加到 `self.episode_a` 列表中,用于记录整个强化学习过程中的行动序列。

3. `self.episode_r.append(r)`: 将当前获得的奖励 `r` 追加到 `self.episode_r` 列表中,用于记录整个强化学习过程中的奖励序列。

通过保存状态、行动和奖励的序列,我们可以利用这些经验数据来训练强化学习算法,如 Q-learning、策略梯度等,从而学习出最优的决策策略。这种保存经验数据的方式在强化学习中非常常见和重要。

```python
    def learn(self, ):
        G = []
        g = 0
        for r in reversed(self.episode_r):  # 反向计算回报 G
            g = self.GAMMA * g + r
            G.insert(0, g)

        for t in range(len(self.episode_r)):
            s = torch.unsqueeze(torch.tensor(self.episode_s[t], dtype=torch.float), 0)
            a = self.episode_a[t]
            g = G[t]

            a_prob = self.policy(s).flatten()
            policy_loss = -pow(self.GAMMA, t) * g * torch.log(a_prob[a])
            self.policy_optimizer.zero_grad()
            policy_loss.backward()
            self.policy_optimizer.step()

        # 清空缓冲区
        self.episode_s, self.episode_a, self.episode_r = [], [], []

```

这段代码是 reinforcement learning 中的策略梯度更新函数。它用于更新神经网络策略模型的参数。下面是对各部分代码的解释:

1. `def learn(self, ):`
   - 这是一个类的成员函数,用于更新策略模型的参数。

2. `G = []`
   - 初始化一个空列表,用于存储每个时间步的累积回报。

3. `g = 0`
   - 初始化累积回报为 0。

4. `for r in reversed(self.episode_r):` 
   - 反向遍历当前 episode 的回报列表 `self.episode_r`。
   - 计算每个时间步的累积回报 `g`。
   - 将累积回报插入列表 `G` 的开头。

5. `for t in range(len(self.episode_r)):`
   - 遍历当前 episode 的状态、动作和回报。
   - 对于每个时间步 `t`:
     - 从 `self.episode_s[t]` 中获取状态 `s`,并将其转换为 PyTorch 张量。
     - 从 `self.episode_a[t]` 中获取动作 `a`。
     - 从 `G[t]` 中获取累积回报 `g`。
     - 使用当前状态 `s` 计算动作概率 `a_prob`。
     - 计算策略损失 `policy_loss`。
     - 清空梯度,并通过反向传播计算梯度。
     - 更新策略模型的参数。

6. `self.episode_s, self.episode_a, self.episode_r = [], [], []`
   - 清空状态、动作和回报的缓冲区。

总的来说,这个函数实现了策略梯度算法,用于更新神经网络策略模型的参数,以提高智能体在强化学习任务中的表现

```python
def evaluate_policy(env, agent):
    times = 3  # 运行评估测试3次,并计算平均奖励
    evaluate_reward = 0
    for _ in range(times):
        s = env.reset()  # 重置环境,获取初始状态
        done = False
        episode_reward = 0
        while not done:
            a = agent.choose_action(s, deterministic=True)  # 使用确定性策略选择动作
            s_, r, done, _ = env.step(a)  # 执行动作,获得下一状态、奖励和是否结束标志
            episode_reward += r  # 累计当前回合的总奖励
            s = s_  # 更新当前状态
        evaluate_reward += episode_reward  # 累计所有回合的总奖励

    return int(evaluate_reward / times)  # 返回3次评估的平均奖励
```

这段代码定义了一个名为`evaluate_policy`的函数,用于评估智能体(agent)在给定环境(env)中的策略表现。具体过程如下:

1. 设定评估次数`times`为3,表示要执行3次评估测试。
2. 初始化`evaluate_reward`变量,用于累计3次评估的总奖励。
3. 进入循环,执行3次评估测试:
   a. 重置环境,获得初始状态`s`。
   b. 初始化`episode_reward`变量,用于记录单次评估的总奖励。
   c. 在当前状态下,智能体使用确定性策略`agent.choose_action(s, deterministic=True)`选择动作`a`。
   d. 执行动作`a`,获得下一状态`s_`、奖励`r`和是否结束标志`done`。
   e. 累加当前回合的奖励`r`到`episode_reward`中。
   f. 更新当前状态`s`为下一状态`s_`。
   g. 当回合结束(`done`为`True`)时,累加`episode_reward`到`evaluate_reward`中。
4. 循环结束后,计算3次评估的平均奖励,并将其作为函数返回值。

总的来说,这个函数用于评估智能体在给定环境中的策略表现,通过运行3次评估测试并计算平均奖励来获得更加可靠的评估结果。

```py
if __name__ == '__main__':
    env_name = ['CartPole-v0', 'CartPole-v1']
    env_index = 0  # 选择环境的索引
    env = gym.make(env_name[env_index])
    env_evaluate = gym.make(env_name[env_index])  # 评估策略时需要重新构建环境
    number = 1
    seed = 0
    env.seed(seed)
    env_evaluate.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)

    state_dim = env.observation_space.shape[0]
    action_dim = env.action_space.n
    max_episode_steps = env._max_episode_steps  # 每集的最大步数
    print("state_dim={}".format(state_dim))
    print("action_dim={}".format(action_dim))
    print("max_episode_steps={}".format(max_episode_steps))

    agent = REINFORCE(state_dim, action_dim)
    writer = SummaryWriter(log_dir='runs/REINFORCE/REINFORCE_env_{}_number_{}_seed_{}'.format(env_name[env_index], number, seed))  # 构建 tensorboard

    max_train_steps = 1e5  # 最大训练步数
    evaluate_freq = 1e3  # 每隔多少步评估一次策略
    evaluate_num = 0  # 记录评估次数
    evaluate_rewards = []  # 记录评估时的奖励
    total_steps = 0  # 记录训练过程中的总步数

    while total_steps < max_train_steps:
        episode_steps = 0
        s = env.reset()
        done = False
        while not done:
            episode_steps += 1
            a = agent.choose_action(s, deterministic=False)
            s_, r, done, _ = env.step(a)
            agent.store(s, a, r)
            s = s_

            # 每隔 evaluate_freq 步评估一次策略
            if (total_steps + 1) % evaluate_freq == 0:
                evaluate_num += 1
                evaluate_reward = evaluate_policy(env_evaluate, agent)
                evaluate_rewards.append(evaluate_reward)
                print("evaluate_num:{} \t evaluate_reward:{} \t".format(evaluate_num, evaluate_reward))
                writer.add_scalar('step_rewards_{}'.format(env_name[env_index]), evaluate_reward, global_step=total_steps)
                if evaluate_num % 10 == 0:
                    np.save('./data_train/REINFORCE_env_{}_number_{}_seed_{}.npy'.format(env_name[env_index], number, seed), np.array(evaluate_rewards))

            total_steps += 1

        # 每集结束后，更新策略
        agent.learn()

```

这段代码是使用 REINFORCE 算法在 Gym 环境中训练一个强化学习智能体。下面是对代码的逐段解释:

1. `if __name__ == '__main__':` 表示这是程序的入口点。
2. 定义了两个可选的环境名称 `'CartPole-v0'` 和 `'CartPole-v1'`，并选择了其中一个环境。
3. 初始化环境 `env` 和评估环境 `env_evaluate`，并设置随机种子。
4. 获取环境的状态维度和动作维度。
5. 创建 REINFORCE 智能体对象 `agent`。
6. 创建 TensorBoard 记录器 `writer`，用于记录训练过程中的奖励等信息。
7. 设置训练的最大步数 `max_train_steps`、评估频率 `evaluate_freq` 和记录评估奖励的列表 `evaluate_rewards`。
8. 开始训练循环,循环直到达到最大训练步数:
   - 重置环境,获取初始状态。
   - 在当前状态下选择动作,并执行该动作,获取下一状态、奖励和是否结束标志。
   - 将状态、动作和奖励存储在智能体的记忆中。
   - 如果达到评估频率,则评估当前策略,并记录评估奖励。每隔 10 次评估,将评估奖励保存到文件。
   - 更新智能体的参数。

总的来说,这段代码实现了使用 REINFORCE 算法在 Gym 环境中训练一个强化学习智能体的过程,并记录了训练过程中的一些指标。

### A2C

我们将 Actor-Critic 分为两个部分：Actor（策略网络）和 Critic（价值网络），如图 10-1 所示。

- Actor 要做的是与环境交互，并在 Critic 价值函数的指导下用策略梯度学习一个更好的策略。
- Critic 要做的是通过 Actor 与环境交互收集的数据学习一个价值函数，这个价值函数会用于判断在当前状态什么动作是好的，什么动作不是好的，进而帮助 Actor 进行策略更新。

![img](https://hrl.boyuai.com/static/640.5dee5ab0.jpg)

图10-1 Actor 和 Critic 的关系



Actor-Critic 算法的具体流程如下：

![image-20240626103700361](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240626103700361.png)

以上就是 Actor-Critic 算法的流程，接下来让我们来用代码实现它，看看效果如何吧！





```python
def learn(self, s, a, r, s_, dw):
    # 将输入状态s和s_转换为PyTorch张量并在批次维度上增加一个维度
    s = torch.unsqueeze(torch.tensor(s, dtype=torch.float), 0)
    s_ = torch.unsqueeze(torch.tensor(s_, dtype=torch.float), 0)

    # 使用critic网络计算当前状态s和下一状态s_的价值函数输出
    v_s = self.critic(s).flatten()
    v_s_ = self.critic(s_).flatten()

    # 在计算梯度时不跟踪梯度
    with torch.no_grad():
        # 计算时间差目标(TD target)
        td_target = r + self.GAMMA * (1 - dw) * v_s_

    # 计算actor网络的损失函数
    log_pi = torch.log(self.actor(s).flatten()[a])
    actor_loss = -self.I * ((td_target - v_s).detach()) * log_pi
    self.actor_optimizer.zero_grad()
    actor_loss.backward()
    self.actor_optimizer.step()

    # 计算critic网络的损失函数
    critic_loss = (td_target - v_s) ** 2
    self.critic_optimizer.zero_grad()
    critic_loss.backward()
    self.critic_optimizer.step()

    # 更新探索参数I
    self.I *= self.GAMMA
```

这段代码实现了一个强化学习算法的学习过程。具体来说:

1. 首先将输入状态`s`和下一状态`s_`转换为PyTorch张量,并在批次维度上增加一个维度。
2. 使用critic网络计算当前状态`s`和下一状态`s_`的价值函数输出`v_s`和`v_s_`。
3. 在计算梯度时不跟踪梯度,计算时间差目标`td_target`。
4. 计算actor网络的损失函数`actor_loss`,并进行反向传播更新actor网络参数。
5. 计算critic网络的损失函数`critic_loss`,并进行反向传播更新critic网络参数。
6. 更新探索参数`I`,使其随时间逐步减少。

这段代码实现了actor-critic算法的核心部分,即同时学习actor网络(策略网络)和critic网络(价值网络),以便于代理学习如何在环境中采取最优行动。