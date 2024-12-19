+++
title = "BipedalWalker Demo: Simple Experiments with RL"
date = "2024-11-11"

[taxonomies]
tags=["RL", "DRL", "BipedalWalker", "Bipedal", "Biped"]

[extra]
comment = true
+++



<center>
<img src="{{ url(path="imgs/bipedal_walker.gif") }}" width="50%">
</center>


<!-- more -->


## Task Definition

This is a 4-joint walker robot environment, the goal is to gain as more points as possible by moving forward to the far end. There are 2 versions to choose from:

- ***Normal, with slightly uneven terrain***: need to get 300+ points in 1600 time steps.
- ***Hardcore, with ladders, stumps, pitfalls***: need to get 300+ points in 2000 time steps.

### Rewards

Reward is given for moving forward, totaling 300+ points up to the far end. If the robot falls, it gets -100. Applying motor torque costs a small amount of points. A more optimal agent will get a better score.

### Actions

Actions are motor speed values in the [-1, 1] range for each of the 4 joints at both hips and knees.

```code
Box(-1.0, 1.0, (4,), float32)
```

### States/Observations

State consists of hull angle speed, angular velocity, horizontal speed, vertical speed, position of joints and joints angular speed, legs contact with ground, and 10 lidar rangefinder measurements. There are no coordinates in the state vector.

```code
Box([-3.1415927 -5. -5. -5. -3.1415927 -5. -3.1415927 -5. -0. -3.1415927 -5. -3.1415927 -5. -0. -1. -1. -1. -1. -1. -1. -1. -1. -1. -1. ], [3.1415927 5. 5. 5. 3.1415927 5. 3.1415927 5. 5. 3.1415927 5. 3.1415927 5. 5. 1. 1. 1. 1. 1. 1. 1. 1. 1. 1. ], (24,), float32)
```

### Episode

The episode will terminate if the hull gets in contact with the ground or if the walker exceeds the right end of the terrain length.



## Control Methods

In this blog we try to solve this task using 3 different methods, DDPG and PPO and SAC.



### DDPG-based

DQN tries to find a policy using quality function $Q(s,a): \text{State} \times \text{Action} \to \mathbb{R}$,

$$
    \pi(s) = \argmax_a Q(s, a)
$$

DQN is a implicit method which indirectly solves this task by finding $Q(s,a)$ first.

DQN uses deep neural network to parameterize $Q(s,a)$, it takes states $s$ as network input, and outputs action values which is the expected return of taking each action given the current input.

To train a neural network, there should exist a loss for which NN to minimize with SGD(stochastic gradient descent). However CartPole sim env doesn't provide explicit groundtruth label for directly training. However we can construct a kind of pseudo groundtruth data with optimal Bellman equation:

$$
Q^\pi (s, a) = r + \gamma \max_{a'} Q^\pi (s', a')
$$

and the error $\delta$ (TD error) between the two sides of the equality can be used to train Q network:

$$
\delta = Q(s,a) − (r + \gamma \max_{a'} Q(s', a'))
$$

To improve training robustness against outliers, we will use Huber loss instead of un-weighted L2 loss for each data sample, so

<p>$$
L(\delta) = \left\{
\begin{aligned}
    \frac{1}{2} \delta^2 & & \text{for $\delta$ $\le$ 1} \\
    \delta - \frac{1}{2} & & \text{otherwise} \\
\end{aligned}
\right.
$$</p>

and the overall loss is: 
$L = \frac{1}{|\mathbf{B}|} \displaystyle\sum\limits_{(s,a,s',r) \in \mathbf{B}} L(\delta)$.


### PPO-based

Proximal Policy Optimization(PPO) is a policy-gradient(PG) algorithm which tries to directly solve the policy to maximize the expected return given some proximality constraints, and this makes PPO a fast and efficient method for online, on-policy reinforcement learning.

The expected return PG tries to optimize is
<p>$$
\pi(s)
= \argmax_\pi R_t
= \argmax\limits_\pi
    \overbrace{\mathbb{E}_\pi \left[ \sum_{k=0} \gamma^k r_{t + k} \right]}^{\color{red}\mathcal{L}}
$$</p>

and its gradient is

<p>$$
\nabla_\theta \mathcal{L} = \mathbb{E}_{\pi_\theta} \left[
    \left( \sum_{k=0} \gamma^k r_{t + k} \right) \nabla_\theta \ln \pi_\theta
\right]
$$</p>

PPO tries to optimize a clipped loss which is a pessimistic bound of return, where lower return estimates will be favored compared to higher ones. The formula of the loss is:

$$
L_{\theta_{new}} (s, a) = \min_{\theta_{new}}
    \left(
        \frac{\pi_{\theta_{new}}(s, a)}{\pi_{\theta_{old}}(s, a)} A^{\pi_{\theta_{old}}}(s, a)
        ,
        g(\epsilon, A^{\pi_{\theta_{old}}}(s, a))
    \right)
$$

where

<p>$$
g(\epsilon, A) = \left\{
    \begin{aligned}
    (1 + \epsilon)A && A \ge 0 \\
    (1 - \epsilon)A && A \lt 0 \\
    \end{aligned}
\right.
$$</p>

This loss ensures that whether the advantage is positive or negative, policy updates that would produce significant shifts from the previous configuration are being discouraged.



## Results

Here we show some results where the agent is trained using vanilla RL algorithms, this can be used as a baseline when comparing with other improved RL algorithms introduced later.

- **Training results with PPO**

    <div style="display:flex;justify-content:center;flex-wrap:wrap;border:0px solid red;">
        <div style="display:flex-inline;width:70%;border:0px solid yellow;text-align:center;">
            <div style="width:80%;">
            <video src="vids/ppo_exp/run0/ppo_sb3_trial2-episode-0.mp4" type="video/webm" oncontextmenu="return false;" width=90% controls autoplay loop muted/>
            </div>
        </div>
        <div style="display:flex-inline;width:70%;border:0px solid yellow;text-align:center;">
            <div style="width:80%;">
            <video src="vids/ppo_exp/run1/ppo_sb3_trial1-episode-0.mp4" type="video/webm" oncontextmenu="return false;" width=90% controls autoplay loop muted/>
            </div>
        </div>
        <div style="display:flex-inline;width:70%;border:0px solid yellow;text-align:center;">
            <div style="width:80%;">
            <video src="vids/ppo_exp/run3/ppo_sb3_trial2-episode-0.mp4" type="video/webm" oncontextmenu="return false;" width=90% controls autoplay loop muted/>
            </div>
        </div>
    </div>


It can be seen that using vanilla RL algorithm with direct torque controlling is almost stucked on some strange policy corresponding to a local minima of deep policy optimization function, although the agent does get enough rewards when approaching goal using this policy.
