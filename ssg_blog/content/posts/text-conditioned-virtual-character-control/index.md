+++
title = "Text Conditioned Physics-based Character Control"
date = "2025-08-09"

[taxonomies]
tags=["RL", "DRL", "IL", "BERT", "GAIL", "AIRL", "Bipedal", "Biped", "Humanoid", "IsaacGym"]

[extra]
comment = true
+++



## Introduction

In this blog, we try to train a text-conditioned robot control policy. It extends a naive adversarial imitation policy by supporting user text instructions to generate motions with different styles. Specifically, we integrate a BERT language model into an adversarial imitation RL pipeline.

<div style="display:flex;justify-content:center;text-align:center;align-items:center;vertical-align:center;">
<video width="60%" oncontextmenu="return false;" nocontrols autoplay loop muted disablepictureinpicture preload=metadata>
    <source src="{{ url(path="vids/vid_random_envs25.webm") }}" type="video/webm">
    Your browser does not support the video tag.
</video>
</div>

<!-- more -->


## Results

<table>
  <!-- <caption></caption> -->
  <thead>
    <tr>
      <th>Text</th>
      <th></th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td><i>CMD:</i> <b>Atk Kick</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_atk-kick.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
    <tr>
      <td><i>CMD:</i> <b>Atk Jump</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_atk-jump.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
    <tr>
      <td><i>CMD:</i> <b>Atk Shield Charge</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_atk-shieldcharge.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
    <tr>
      <td><i>CMD:</i> <b>Atk Shield Swipe</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_atk-shieldswipe1.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
    <tr>
      <td><i>CMD:</i> <b>Dodge Backward</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_dodge-backward.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
    <tr>
      <td><i>CMD:</i> <b>Atk Shield Charge</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_atk-shieldcharge.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
    <tr>
      <td><i>CMD:</i> <b>Atk Slash Up</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_atk-slashup.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
    <tr>
      <td><i>CMD:</i> <b>Atk Slash Down</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_atk-slashdown.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
    <tr>
      <td><i>CMD:</i> <b>Atk Slash Left</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_atk-slashleft.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
    <tr>
      <td><i>CMD:</i> <b>Atk Slash Right</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_atk-slashright.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
    <tr>
      <td><i>CMD:</i> <b>Atk Combo</b></td>
      <td style="text-align:center;">
        <video width="60%" oncontextmenu="return false;" controls autoplay loop muted disablepictureinpicture preload=metadata>
            <source src="vids/vid_atk-2xcombo1.webm" type="video/webm">
            Your browser does not support the video tag.
        </video>
      </td>
    </tr>
  </tbody>

  <tfoot>
    <tr>
      <td>
      </td>
    </tr>
  </tfoot>
</table>
