+++
title = "Quadruped Demo: CPG Introduction"
date = "2024-10-29"
updated = "2024-11-03"

[taxonomies]
tags=["RL", "DRL", "CPG", "Quadruped", "Quadrupedal", "Four-footed"]

[extra]
comment = true
+++



## Background

A central pattern generator (CPG) is a biological neural circuit that produces rhythmic motor patterns, such as walking, breathing, flying, or swimming. CPGs are found in humans and many kinds of animals.

For control tasks such as locomotion of bipedal or quadrupedal robots, the action space is both high-dimensional and continuous, finding methods to directly control all joints is very difficult. It's natural to think controlling each leg motion rather than each joint torque may reduce the complexity of the solution.

CPG assumes each leg's motion follows some rhythmic or quasi-periodic pattern, and divides one full-joints control task into 2(biped)/4(quadruped) leg-joints control tasks, this strategy simplifies the problem and also improves interpretability/controllability.


<!-- more -->


## Introduction


<div style="display:flex;justify-content:center;align-items:center;flex-wrap:wrap;border:2px solid red;">
    <div style="display:inline-flex;width:48%;justify-content:center;align-items:center;">
    <video width="90%" controls controlslist="nodownload nofullscreen" autoplay loop muted disablepictureinpicture preload=metadata>
        <source src="vids/What’s New in Spot (zIdyhGyXcUg)___crop_w2600xh1500_500x300.webm" type="video/webm">
        Your browser does not support the video tag.
    </video>
    </div>
    <div style="display:inline-flex;width:48%;justify-content:center;align-items:center;">
    <video width="90%" controls controlslist="nodownload nofullscreen" autoplay loop muted disablepictureinpicture preload=metadata>
        <source src="vids/With you, Spot can [VRm7oRCTkjE]___crop_38-47s.webm" type="video/webm">
        Your browser does not support the video tag.
    </video>
    </div>
    <div style="display:flex;width:90%;justify-content:center;align-items:center;text-align:center;vertical-align:center;">
    <p style="margin:0;padding:0;">
    <!-- <span style="font-size:1em;color:HotPink;background-color:yellow;"> -->
    <span style="font-size:1em;color:Violet;background-color:transparent;">
    <b>BD<img style="height:0.7em;" src="https://forgeglobal.com/site/assets/files/1330/boston_dynamics-1.jpg"> Spot-Mini Robot Dog</b>
    <span>
    </p>
    </div>
</div>


<!--
<div>
<iframe width="80%" height="480" src="https://youtube.com/embed/6U-bI3On1Ww?start=106&end=157" title="No Time to Dance | Boston Dynamics" frameborder="0" allow="accelerometer; controls=0; autoplay=0; clipboard-write; encrypted-media; gyroscope; picture-in-picture;" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
<iframe width="80%" height="480" src="https://youtube.com/embed/VRm7oRCTkjE" title="With you, Spot can" frameborder="0" allow="accelerometer; controls=0; autoplay=0; clipboard-write; encrypted-media; gyroscope; picture-in-picture;" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
<iframe width="80%" height="480" src="https://youtube.com/embed/fn3KWM1kuAw" title="Do You Love Me?" frameborder="0" allow="accelerometer; controls=0; autoplay=0; clipboard-write; encrypted-media; gyroscope; picture-in-picture;" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
<iframe width="80%" height="480" src="https://youtube.com/embed/MG4PPkCyJig" title="Meet Sparkles | Boston Dynamics" frameborder="0" allow="accelerometer; controls=0; autoplay=0; clipboard-write; encrypted-media; gyroscope; picture-in-picture;" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
<iframe width="80%" height="480" src="https://youtube.com/embed/2SpNjBI1lu0" title="How Boston Dynamics' Spot Robot Learns to Dance!" frameborder="0" allow="accelerometer; controls=0; autoplay=0; clipboard-write; encrypted-media; gyroscope; picture-in-picture;" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
<iframe width="80%" height="480" src="https://youtube.com/embed/UAG_FBZJVJ8" title="Unbelievable Robot Dance by Boston dynamics" frameborder="0" allow="accelerometer; controls=0; autoplay=0; clipboard-write; encrypted-media; gyroscope; picture-in-picture;" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>
-->


There are different types of math models for CPG, here in this blog we focus on a specific oscillator model as follows: <cite>[Auke.Ijspeert.Science.2007](#refs)</cite>

{% tex() %}
$$
\left\{
\begin{align}
    \ddot{r}_i &= a \left( \frac{a}{4} \left( \mu_i - r_i \right) - \dot{r}_i \right) \\
    \dot{\theta}_i &= \omega_i + \textstyle\sum\nolimits_{j} r_j w_{ij} \sin(\theta_j - \theta_i - \phi_{ij}) \\
\end{align}
\right.
$$
{% end %}

where $r_i$ is the current amplitude of the oscillator, $\theta_i$ is the phase of the oscillator, $\mu_i$ and $\omega_i$ are the intrinsic amplitude and frequency, $a$ is a positive constant representing the convergence factor. Couplings between oscillators are defined by the weights $w_{ij}$ and phase biases $\phi_{ij}$.

There are different choices for how to associate oscillator's output to joint's command, such as:

- **one oscillator per joint:** this directly maps oscillator output to joint command in joint space, which is a $N\text{-}to\text{-}1$ mapping, hence increases action space's dimension and corresponding problem complexity.
- **one oscillator per leg:** this maps oscillator output to leg movement in task space, and then solves leg's joints commands via inverse kinematics.

In this blog we focus on the second case, i.e., implementing CPG with one oscillator per limb, this choice on the one hand doesn't increase control complexity as it's a $3\text{-}to\text{-}3$ mapping per limb, on the other hand it exposes explicit model physical parameters to adjust on the fly.

The oscillators' couplings parameters $w_{ij}$ and $\phi_{ij}$ are strongly related to legged robots' various gaits, like walking, trotting, galloping. However this is not necessary if there is a global controller for the agent to tune all oscillators' intrinsic parameters in real-time <cite>[Robin.Thandiackal.scirobotics.2021](#refs)</cite> <cite>[Dai.Owaki.Nature.2017](#refs)</cite>. Thus a simplified CPG model which ignores explicit couplings between oscillators is used here and the oscillator model of each limb is in the following:

{% tex() %}
$$
\left\{
\begin{align}
\ddot{r}_i &= a \left(\frac{a}{4} \left(\mu_i - r_i \right) - \dot{r}_i \right) \\
\dot{\theta}_i &= \omega_i  \\
\dot{\phi}_i &= \psi_i \\
\end{align}
\right.
$$
{% end %}

To map each oscillator's states to corresponding joints command, 2 steps are included:

1. map leg oscillator's states to corresponding foot position.
2. calculate leg's joints angle from foot position with inverse kinematics.

Although there is no directly physical connection between leg's CPG parameters $\{r_i, \theta_i, \phi_i\}_{i=0,\dots}^{\{ 1,\dots,4 \}}$ and foot's position $\{x_i, y_i, z_i\}_{i=0,\dots}^{\{ 1,\dots,4 \}}$, one commonly used formulation is as follows:

{% tex() %}
$$
\left\{
\begin{align}
x_{i,\text{foot}} &= -d_{step} (r_i-1) \cos(\theta_i) \cos(\phi_i) \\
y_{i,\text{foot}} &= -d_{step} (r_i-1) \cos(\theta_i) \sin(\phi_i) \\
z_{i,\text{foot}} &= \begin{cases}
    -h + g_c\sin(\theta_i) & \text{if } \sin(\theta_i) > 0 \\
    -h + g_p\sin(\theta_i) & \text{otherwise} \\
\end{cases} \\
\end{align}
\right.
$$
{% end %}

where $d_{step}$ is the max step length, $h$ is the robot height, $g_c$ is the max ground clearance during swing, and $g_p$ is the max ground penetration during stance. 



## Refs

- [Auke.Ijspeert.Science.2007]: From swimming to walking with a salamander robot driven by a spinal cord model
- [Dai.Owaki.Nature.2017]: A quadruped robot exhibiting spontaneous gait transitions from walking to trotting to galloping
- [Robin.Thandiackal.scirobotics.2021]: Emergence of robust self-organized undulatory swimming based on local hydrodynamic force sensing
- [Guillaume.Bellegarda.RAL.2022]: CPG-RL: Learning Central Pattern Generators for Quadruped Locomotion
- [Guillaume.Bellegarda.ICRA.2024]: Visual CPG-RL: Learning Central Pattern Generators for Visually-Guided Quadruped Locomotion
