---
layout: posts
title: proxmox container 에서 gpu 및 docker 사용 설정하기
date: 2025-08-29 00:55:00
categories: 
  - Research
tags:
comments: True
---

# 1. Environment

- Dell workstation T7920 with A6000 GPU
- PROXMOX VE 8.3.5
- ubuntu 
10.0.0.138 : proxmox ve gpu
10.0.1.139 : 내 메인 컨테이너
10.0.1.140 : 정대영씨 메인 컨테이너
10.0.0.140 : service container, wireguard
10.0.1.141 : ATANU 메인 컨테이너

10.0.1.139:8080 : gpt
