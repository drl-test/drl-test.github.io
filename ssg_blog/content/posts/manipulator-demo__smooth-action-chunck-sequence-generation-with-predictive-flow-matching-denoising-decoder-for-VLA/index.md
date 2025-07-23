+++
title = "Manipulator Demo: Smooth action chunck sequence generation with predictive flow matching denoising decoder for VLA"
date = "2025-07-06"

[taxonomies]
tags=["VLA", "IL", "Flow-Matching", "SmolVLM", "SmolVLA", "Robot-Arm", "Manipulator", "LIBERO"]

[extra]
comment = true
+++



## Introduction

VLA (Vision-Language-Action) models have shown promising results for robots to perform tasks in complex environments based on visual input and textual instructions. Specifically, chunk-based action generation is crucial for its long-horizon planning and short-term per-chunk smooth motion. However, naive chunk-by-chunk action execution strategy may result in jerky robot motion when switching chunks. In this blog, a predictive flow-matching-based action decoder design is proposed to generate smooth action sequences, which forms a key component of our lightweight VLA variant implementation.

<div style="display:flex;justify-content:center;text-align:center;align-items:center;vertical-align:center;">

<div style="display:inline-flex;width:40%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;">
    <figure>
    <img src="{{ url(path="imgs/Panda-robot-with-labeled-joints.min.jpg") }}" width=60%/>
    <!-- <img src="{{ url(path="imgs/Panda-robot-with-labeled-joints.jpg") }}" style="clip-path:inset(130px 0px 290px 0x); margin:-100px;"/> -->
    <figcaption style="line-height:1.1em;">
    <span style="font-size:0.8em;color:indigo;"><strong>Franka Panda Emika Robot</strong></span>
    <br/>
    <span style="font-size:0.6em;">
    <em>
    (<a href=https://www.researchgate.net/figure/Panda-robot-with-labeled-joints_fig1_361659008 style="pointer-events:none;">RoboGroove</a><!--
    --><span style="color:red;">&copy;</span>)
    </em>
    </span>
    </figcaption>
    </figure>
</div>

<div style="display:inline-flex;width:55%;justify-content:center;flex-wrap:wrap;margin-top:0.5%;margin-bottom:0.5%;">
    <div>
    <video width="90%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
        <source src="{{ url(path="vids/task_2_episode_3_turn_on_the_stove_and_put_the_moka_pot_on_it.mp4") }}" type="video/mp4">
        Your browser does not support the video tag.
    </video>
    </div>
    <p style="font-size:80%;margin-top:0;">
    <b><span style="color:blue">Task:</span> <span style="color:red">turn on the stove and put the moka pot on it</span></b>
    </p>
</div>

</div>

<!-- more -->


## Method

VLA conceptually has three logical components: visual perception, language comprehension, and action generation. Current (as of 2025) SOTA VLA models typically employ either a dual-core system that consists of a VLM scene/task understander and an action generator, or an end-to-end general policy that directly maps observations to actions.

Here, we try to implement a lightweight VLA model of the first kind (i.e., a VLM + Expert architecture) for low-cost consumer devices (e.g., RTX3060). Additionally, we expect this VLA agent to output actions that lead to smooth motion as much as possible.

To achieve the first goal, we use the pretrained SmolVLM2 as the lightweight backbone for scene perception and task planning, which has about 0.35B parameters (with some layers removed from pretrained model). As for the second goal, TE(temporal ensemble) in ACT is a commonly used method to alleviate/circumvent this discontinuity issue between adjacent chunks, despite potentially outputting invalid actions. Here we propose a new smoothness-oriented action generation method, which has model parameters about 0.13B. To elaborate a bit, we adopt the following improvements:

- new state feature representation: two perpendicular axes are used to represent the robot arm's attitude feature instead of the Rodrigues (AA) form.
- predictive action chunk generation: briefly speaking, we implement a 2D flow-matching based action expert with formula $p(A_{t} - A_{t-E} {\ }|{\ } O_{t}, A_{t-E})$ instead of $p(A_{t} {\ }|{\ } O_{t})$.



## Experiments

| Task | |
| -- | -- |
| put the black bowl in the bottom drawer of the cabinet and close it | <video width="90%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata src="vids/task_3_episode_1_put_the_black_bowl_in_the_bottom_drawer_of_the_cabinet_and_close_it.mp4" type="video/mp4"/> |
| put both moka pots on the stove | <video width="90%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata src="vids/task_8_episode_3_put_both_moka_pots_on_the_stove.mp4" type="video/mp4"/> |
| put the wine bottle on top of the cabinet | <video width="90%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata src="vids/rollout_task_2_episode_0_put_the_wine_bottle_on_top_of_the_cabinet_success.mp4" type="video/mp4"/> |
| put the bowl on the plate | <video width="90%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata src="vids/rollout_task_8_episode_0_put_the_bowl_on_the_plate_success.mp4" type="video/mp4"/> |
| put the wine bottle on the rack | <video width="90%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata src="vids/rollout_task_9_episode_1_put_the_wine_bottle_on_the_rack_success.mp4" type="video/mp4"/> |
| ... | <font color=red><em>TO-ADD-MORE</em></font> |
