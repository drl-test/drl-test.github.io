+++
title = "Humanoid Demo: Game2D-to-Sim3D Cross Domain Skill Adaptation From CPG Expert Demonstrations"
date = "2025-03-01"

[taxonomies]
tags=["RL", "DRL", "IL", "GAIL", "AIRL", "CPG", "Bipedal", "Biped", "Humanoid", "Unitree/H1", "Unitree/H1-2", "IsaacGym"]

[extra]
comment = true
+++



## Introduction

In previous blog posts, we have shown successful applications of ***CPG-based RL*** for robot locomotion in both 2D (game) and 3D (world) physical simulation environments. While this approach offers the flexibility to adjust parameters for walking gait on-the-fly, it necessitates heavy reward tuning and prolonged training time. On the other hand, imitation learning from human demonstrations has shown more rapid convergence for natural gait cloning. Here we will try to combine these two techniques and explore the feasibility of using a 2D CPG expert to guide a 3D humanoid robot in learning to walk.

<div style="display:flex;justify-content:center;text-align:center;align-items:center;vertical-align:center;">

<div style="display:inline-flex;width:40%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;">
    <div>
    <video width="80%" oncontextmenu="return false;" nocontrols autoplay loop muted disablepictureinpicture preload=metadata>
        <source src="{{ url(path="/posts/bipedalwalker-demo-cpg-experiments/vids/cpg_rl/f8_cpg.mp4", baseurl=true) }}" type="video/mp4">
        Your browser does not support the video tag.
    </video>
    </div>
    <p style="font-size:80%;margin-top:0;">
        <strong>
        <span style="color:#f80808;font-size:60%;">BipedalWalker:</span>
        <!-- <br/> -->
        <span style="color:#0808f8;font-size:70%;">Finally has learned to walk~</span>
        </strong>
    </p>
</div>

<div style="display:inline-flex;width:20%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;margin-left:-8%;margin-right:-4%;">
    <figure>
    <img src="{{ url(path="imgs/right_arrow.png") }}" width="150%" style="border:0;">
    <figcaption>
    <a href="" style="pointer-events: none"></a>
    </figcaption>
    </figure>
</div>

<div style="display:inline-flex;width:45%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;">
    <div>
    <video width="100%" oncontextmenu="return false;" nocontrols autoplay loop muted disablepictureinpicture preload=metadata>
        <source src="{{ url(path="vids/Apr23_00-42-50_test_mlp_noise0.001_ent0.001_nstps16_nbs16_gl_0.95_0.9_tasklerp0.3_firstN.mp4") }}" type="video/mp4">
        Your browser does not support the video tag.
    </video>
    </div>
    <p style="font-size:80%;margin-top:0;">
        <strong>
        <span style="color:#f80808;font-size:70%;">Unitree/H1-2:</span>
        <!-- <br/> -->
        <span style="color:#0808f8;font-size:80%;">Done copying!</span>
        </strong>
    </p>
</div>

</div>

<!-- more -->


## Methods

We evaluate the idea of using BipedalWalker's 2D CPG demonstrations to train Unitree H1-2's 3D locomotion in two stages:

1. Same Domain: train an IL+RL policy using Unitree H1-2 **3D CPG** expert demonstration data

    Here, tens of demonstration datasets of Unitree H1-2 were collected using a previously trained 3D CPG expert policy. Each dataset contains a full episode trajectory with a maximum length equal to the predefined timeout. To make it easier to compare the result using the 2D BipedalWalker dataset, only a forward speed command was given to the robot, and both lateral speed and heading commands were set to zero.

1. Cross Domain: train an IL+RL policy using Gym BipedalWalker **2D CPG** expert demonstration data

    Two distinct policies were used for generating demonstration data, one is the previously trained 2D CPG expert policy, while the other is Gym BipedalWalker's built-in heuristic policy. There are 3 key domain discrepancies when transferring locomotion experiences:

    * Robot Model: Gym BipedalWalker (leveraging the Box2D physics engine) versus Unitree H1-2 (powered by IsaacGym physics engine)
    * World Dimension: 2D game world versus 3D physical world
    * Terrain: uneven ground versus even surfaces



## Experiments

#### **Imitation from Same Model's CPG Expert Demonstratiton**

- Dataset Details

    A pre-trained CPG+RL expert policy is adopted for Unitree H1-2 to walk, and two kinds of command parameters are contained in the policy's inputs, including environment commands and CPG model parameters. Although all parameters support on-the-fly adjustment, here they are randomly set at the beginning and remain fixed for each episode. Specifically,

    | | **CMD** | **VALUE** |
    | -- | -- | -- |
    | ***ENV*** |
    | | $v_x$ | $\mathcal{U}(0.0, 1.0)$ m/s |
    | | $v_y$ | $0.0$ m/s |
    | | $\omega_z$ | $0.0$ rad/s |
    | ***CPG*** |
    | | $ht$ | $1.85$ |
    | | $stp$ | $0.4$ |
    | | $g_c$ | $0.2$ |
    | | $g_p$ | $0.01$ |

- Evaluations

    <div style="display:flex;justify-content:center;text-align:center;align-items:center;vertical-align:center;row-gap:1%;column-gap:1%;">
    <div style="display:inline-flex;width:45%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;">
        <video src="vids/Apr23_16-31-45_test_amp2d.pos__crop.mp4" width=100% oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata/>
    </div>
    <div style="display:inline-flex;width:45%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;">
        <video src="vids/Apr23_00-42-50_test_mlp_noise0.001_ent0.001_nstps16_nbs16_gl_0.95_0.9_tasklerp0.3_firstN.mp4" width=100% oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata/>
    </div>
    </div>


#### **Inspiration from Other Model's CPG Pattern Extraction**

- Dataset Samples

    <div style="display:flex;justify-content:center;text-align:center;align-items:center;vertical-align:center;row-gap:1%;column-gap:1%;">
    <div style="display:inline-flex;width:45%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;">
        <video src="{{ url(path="/posts/bipedalwalker-demo-cpg-experiments/vids/cpg_rl/f8_cpg.mp4", baseurl=true) }}" width=100% oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata/>
    </div>
    <div style="display:inline-flex;width:45%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;">
        <video src="vids/bipedalwalker_policy_heuristics_20250504.mp4" width=100% oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata/>
    </div>
    </div>

- Evaluations

    <div style="display:flex;justify-content:center;text-align:center;align-items:center;vertical-align:center;row-gap:1%;column-gap:1%;">

    <div style="display:inline-flex;width:45%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;">
        <div>
        <video src="vids/May08_22-50-47_test_norefinit_pp4.webm" width=100% oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata/>
        </div>
        <p style="font-size:95%;color:Orchid;margin-top:0;">
            <strong>
            forward only movement
            </strong>
        </p>
    </div>

    <div style="display:inline-flex;width:45%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;">
        <div>
        <video src="vids/20250603_2D_guided_3D_locomotion.webm" width=100% oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata/>
        </div>
        <p style="font-size:95%;color:Violet;margin-top:0;">
            <strong>
            lifting to omni-movement
            </strong>
        </p>
    </div>

    </div>
