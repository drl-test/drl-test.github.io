+++
title = "Humanoid Demo: Porting CPG to Unitree H1"
date = "2025-01-26"

[taxonomies]
tags=["RL", "DRL", "Bipedal", "Biped", "Humanoid", "Unitree-H1", "IsaacGym"]

[extra]
comment = true
+++



## Introduction

In this Blog we will adapt CPG to a real bipedal Humanoid model [Unitree H1-2](https://www.unitree.com/h1).


<!-- more -->


## UniTree H1-2 Spec

<div style="display:flex;justify-content:center;align-items:center;text-align:center;flex-wrap:wrap;flex-wrap:wrap;">

<div style="display:flex-inline;width:29%;justify-content:center;align-items:center;text-align:center;">
<img src="imgs/unitree_h1-2.png" width="64%">
</div>

<div style="display:flex-inline;width:69%;justify-content:center;align-items:center;text-align:center;">
<figure>
<img src="imgs/unitree_h1-2_sketch.png" width="70%">
<figcaption>
<a href="https://doc-cdn.unitree.com/static/2024/9/4/9e873852c7e24cb7bbae7f19868483f8_2730x1784.png" style="pointer-events: none">
Unitree H1-2 Components
</a>
</figcaption>
</figure>
</div>
</div>

<a href="https://support.unitree.com/home/en/H1_developer/About_H1-2" style="pointer-events: none">Unitree H1-2</a> is a humanoid robot based on <a href="https://support.unitree.com/home/en/H1_developer/About_H1" style="pointer-events: none">Unitree H1</a> model, boasting enhanced motion capability and degrees of freedom. The comparison of concerned parameters between H1 and H1-2 is shown as follows:

| | H1 | H1-2 |
| -- | -- | -- |
| Key Dimensions | (1520+285)mm × 570mm × 220mm | (1503+285)mm × 510mm × 287mm |
| Thigh and Calf Length | 400mm × 2 | 400mm × 2 |
| Total Arm Length | 338mm × 2 | 685mm |
| DOF of Each Leg | 5（Hip × 3 + Knee × 1 + Ankle × 1） | 6（Hip x 3 + Knee x 1 + Ankle x 2）|
| DOF of Each Arm | 4（Expandable） | 7（Shoulder x 3 + Elbow x 1 + Wrist x 3）|
| Core Joint motor | Low inertia high-speed internal rotor PMSM <br/> (permanent magnet synchronous motor,better response speed and heat dissipation) | Low inertia high-speed internal rotor PMSM <br/> (permanent magnet synchronous motor,better response speed and heat dissipation) |
| Ultimate Torque of Joint Unit | Knee Torque About 360N.m <br/> Hip Joint Torque About 220N.m <br/> Ankle Torque About 59N.m <br/> Arm Joint Torque About 75N.m <br/> Arm joint approximately 75N.m | Knee Torque About 360N.m <br/> Hip Joint Torque About 220N.m <br/> Waist Joint About 220N.m <br/> <font color=red>TBD:</font> ~~Ankle Joint About 75x2N.m~~ ~~(Ankle joint approximately 45N.m)~~ <br/> Arm joint approximately 75N.m |
| Dexterous Hand | Optional | Optional RH56 or other ambidextrous hands |
| Arm joint performance <br/> (peak torque) | / | Shoulder: About 120N.m <br/> Elbow: About 120N.m <br/> Wrist: About 30N.m |
| Mobility | Moving speed of 3.3m/s <br/> Potential mobility > 5m/s | Moving speed of 3.3m/s, Potential mobility > 5m/s |
| Sensor Configuration | 3D LIDAR + Depth Camera | 3D LIDAR + Depth Camera |
| Total Weight | ~ 47kg | ~ 70kg |


### Joints

Each arm of the H1-2 possesses 7 degrees of freedom, each leg has 6, and the waist features 1 degree of freedom, culminating in a total of 27 degrees of freedom across the entire machine.

<div style="display:flex;justify-content:center;align-items:center;text-align:center;flex-wrap:wrap;flex-wrap:wrap;">

<div style="display:flex-inline;width:74%;justify-content:center;align-items:center;text-align:center;">
<figure>
<img src="imgs/unitree_h1-2_joints_3876x3035.png" width="60%">
<figcaption>
<a href="https://doc-cdn.unitree.com/static/2024/9/3/3a458dbbd548408fbf66099cca4e1f66_3876x3035.png" style="pointer-events: none">
H1-2 Joint Numbering
</a>
</figcaption>
</figure>
</div>

<div style="display:flex-inline;width:24%;-justify-content:center;align-items:center;-text-align:center;">
<figure>
<img src="imgs/unitree_h1-2_joint_coord_450x1190.jpg" width="80%">
<figcaption>
<div style="display:block;">
<a href="https://doc-cdn.unitree.com/static/2024/9/5/928eea81692d486b82ae056f2a7865f4_450x1190.jpg" style="pointer-events: none">
<span style="font-size:50%;">H1-2 Joint Coordinate System</span>
</a>
<br/>
<div style="font-size:50%;line-height:100%;">
<span style="color:red;font-size:100%;"><b>red axis -> x-axis</b></span>
<br/>
<span style="color:green;font-size:100%;"><b>green axis -> y-axis</b></span>
<br/>
<span style="color:blue;font-size:100%;"><b>blue axis -> z-axis</b></span>
</div>
</figcaption>
</figure>
</div>
</div>

**Joint Numbering and Joint Limits**

| Joint Name | Limits | Joint Number |
| -- | -- | -- |
| {left,right}_hip_yaw | -0.43~+0.43 rad | left: 0, right: 6 |
| {left,right}_hip_pitch | -3.14~+2.5 rad | left: 1, right: 7 |
| left_hip_roll | -0.43~+3.14 rad | 2 |
| right_hip_roll | -3.14~+0.43 rad | 8 |
| {left,right}_knee | -0.26~+2.05 rad | left: 3, right: 9 |
| {left,right}_ankle_pitch | -0.897334~+0.523598 rad | left: 4, right: 10 |
| {left,right}_ankle_roll | -0.261799~+0.261799 rad | left: 5, right: 11 |
| torso | -3.14~+1.57 rad | 12 |
| left_shoulder_pitch | -3.14~+1.57 rad | 13 |
| right_shoulder_pitch | -1.57~+3.14 rad | 20 |
| left_shoulder_roll | -0.38~+3.4 rad | 14 |
| right_shoulder_roll | -3.4~+0.38 rad | 21 |
| left_shoulder_yaw | -3.01~+2.66 rad | 15 |
| right_shoulder_yaw | -2.66~+3.01 rad | 22 |
| left_elbow_pitch | -2.53~+1.6 rad | 16 |
| right_elbow_pitch | -1.6~+2.53 rad | 23 |
| {left,right}_elbow_roll | -2.967~+2.967 rad | left: 17, right: 24 |
| {left,right}_wrist_pitch | -0.471~+0.349 rad | left: 18, right: 25 |
| {left,right}_wrist_yaw | -1.012~+1.012 rad | left: 19, right: 26 |


