---
title: "1. 프로젝트 설계 목표"
date: 2026-05-20 15:00:00 +0900
categories: [Electronics, Circuit Design]
tags: [amplifier, electronics, circuit-design]
---
## Final Project: Single-Ended Neural Signal Amplifier Design
전자회로1 과목의 Final Project 과정을 기록하기 위해 블로그를 씀. 
우선 교수님이 주신 프로젝트의 설계 명세를 분석하고 설계 방향을 잡는 과정을 기록해보고자 함.
***
## 1. 프로젝트 목표
본 프로젝트의 목표는 주어진 소자만을 사용하여 single-ended 신경신호 증폭기를 설계하는 것.
설계 대상은 작은 생체/신경 신호를 안정적으로 증폭하면서, 지정된 주파수 응답과 PPA 조건을 만족하는 analog front-end amplifier.

입력 신호는 DC common-mode voltage 2.5 V 위에 작은 AC 신호가 실린 형태로 인가됨. 따라서 본 설계는 단순한 gain stage가 아니라, 다음 요소를 함께 고려하는 회로 설계 문제임.

- 작은 신호 증폭
- 목표 대역폭 구현
- Out-of-band noise 및 signal 억제
- 10 pF load 조건에서의 안정성
- Power / area 효율 최적화
- AC response 및 transient response 유사도 확보
***
위 내용은 교수님이 주신 설계 명세의 첫번째 항목이고 정리하자면 아주 작은 생체신호를 증폭하는 amplifier를 설계하는 문제임. 다음 항목의 목표사양을 보면 그림이 그려질 것 같음
***
![alt text](<스크린샷 2026-05-20 오후 2.21.05.png>)
