---
layout: single
title: "2.The basic of AR functionality"
categories: ar_introduction
---

- [1) What makes AR feel real?](#1-what-makes-ar-feel-real)
  - [1.1) The hardware that makes mobile AR work](#11-the-hardware-that-makes-mobile-ar-work)
  - [1.2) Placing and positioning assets](#12-placing-and-positioning-assets)
  - [1.3) Scale and the size of assets](#13-scale-and-the-size-of-assets)
  - [1.4) Occlusion](#14-occlusion)
  - [1.5) Lighting for increased realism](#15-lighting-for-increased-realism)
  - [1.6) Solid augmented](#16-solid-augmented)
  - [1.7) Context awareness](#17-context-awareness)
- [2) The magic of AR: detecting, sensing, and understanding](#2-the-magic-of-ar-detecting-sensing-and-understanding)
  - [2.1) Tracking in AR](#21-tracking-in-ar)
  - [2.2) Outside-in tracking](#22-outside-in-tracking)
  - [2.3) Inside-out tracking](#23-inside-out-tracking)
- [3) Inside ARCore: the building blocks of augmented reality](#3-inside-arcore-the-building-blocks-of-augmented-reality)
  - [3.1) Motion tracking](#31-motion-tracking)
  - [3.2) Environmental understanding](#32-environmental-understanding)
  - [3.3) Light estimation](#33-light-estimation)
  - [3.4) Anchors](#34-anchors)
- [4) The current challenges facing AR today](#4-the-current-challenges-facing-ar-today)
- [4.1) Interface issues and lack of UI metaphors](#41-interface-issues-and-lack-of-ui-metaphors)
- [4.2) AR's technical constraints: size, power, heat](#42-ars-technical-constraints-size-power-heat)
- [4.3) The 3D barrier](#43-the-3d-barrier)
- [4.4) Computer vision limitations](#44-computer-vision-limitations)
- [4.5) Constraints of occlusion and shading](#45-constraints-of-occlusion-and-shading)

---

# 1) What makes AR feel real?
## 1.1) The hardware that makes mobile AR work
* motion tracking for AR
  * 가속도계(Accelerometer): 가속도를 측정
  * 자이로스코프(Gyroscope): 방향과 각속도를 측정 및 유지
  * 카메라
* location-based AR
  * 마그네토미터(Magnetometer): 지구 자기장에 의하여 방향 제공
  * GPS
* view of real world with AR
  * 디스플레이

## 1.2) Placing and positioning assets
* Placing: 디지털 물체가 실제 세계에 고정(앵커)되었을 경우

## 1.3) Scale and the size of assets
* Scaling: AR 기기의 위치에 따라 크기가 변하는 것(가까이 다가오는 경우 커짐)


## 1.4) Occlusion
* 물체가 다른 물체에 의해 가려졌을 경우

## 1.5) Lighting for increased realism
## 1.6) Solid augmented
## 1.7) Context awareness

---

# 2) The magic of AR: detecting, sensing, and understanding
## 2.1) Tracking in AR
* 기기 또는 사용자의 위치와 방향을 추적하는 방법
  * Outside-in tracking
  * Inside-out tracking

## 2.2) Outside-in tracking
* 외부의 카메라 또는 센서로 감지
* 정확도 높음
* 이동성 낮음

## 2.3) Inside-out tracking
* 기기 내부의 카메라로 감지
* AR 기기에 많은 하드웨어 필요
* 이동성 높음

---

# 3) Inside ARCore: the building blocks of augmented reality
## 3.1) Motion tracking
* 카메라, 자이로스코프, 가속도계를 사용하여 실시간으로 3차원 공간에서 pose를 알아내는 것
* COM(Concurrent Odometry and Mapping): ARCore의 모션 트래킹 프로세스, 스마트폰의 실세계에서의 위치를 추적

## 3.2) Environmental understanding
* 물체들을 인식하여 적절하게 사용하는 것

## 3.3) Light estimation
* 디지털 물체의 밝기를 실제 세계와 어울리게 조절하여 현실감을 주는 것

## 3.4) Anchors
* 물체를 특정 위치에서 잡고있음
* 모션트래킹을 보완하여 에러가 날 경우 수정

---

# 4) The current challenges facing AR today
# 4.1) Interface issues and lack of UI metaphors
* pc의 마우스, 키보드와 같은 것이 명확하지 않음

# 4.2) AR's technical constraints: size, power, heat
* 기기가 작아야 함, 장비가 전력을 많이 먹음, 배터리의 발열

# 4.3) The 3D barrier
* 대부분이 3D라서 적응이 힘듦

# 4.4) Computer vision limitations

# 4.5) Constraints of occlusion and shading

