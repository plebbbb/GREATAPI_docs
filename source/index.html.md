---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - cpp
 
toc_footers:
  - <a href='https://www.stlrobotics.org/'>Go to our website</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - Odometry
  - Control Loops
  - errors

search: true

code_clipboard: true
---

# Introduction

Welcome to GREATAPI, a Vex Robotics library for the PROS API. 

# Installation

To install GREATAPI, head to our [releases page](https://github.com/plebbbb/GREATAPI/releases) and download the lastest GREATAPI@{something}.zip.

Open up a console on the folder you've downloaded GREATAPI to, and enter the following command:

`
prosv5 conduct fetch greatapi@[whatever version we are on].zip
`

From here, GREATAPI should be addable on the upgrade tab of the PROS editor. You may have to manually type in the name for it to be detected. If GREATAPI isn't there, open up a console in the directory of your PROs project(the folder with project.pros) and enter the following:

`
prosv5 conduct apply greatapi
`
