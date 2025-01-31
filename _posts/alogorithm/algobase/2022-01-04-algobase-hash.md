---
layout: single
title: "해시"
categories: algobase

toc: true
toc_sticky: true
---

# 기본 개념
![해싱](https://upload.wikimedia.org/wikipedia/commons/thumb/7/7d/Hash_table_3_1_1_0_1_0_0_SP.svg/1280px-Hash_table_3_1_1_0_1_0_0_SP.svg.png)
## 필요성
* 기존의 탐색은 시간복잡도가 큼
  * 선형: O(N)
  * 바이너리 서치: O(logN)
* 해싱은 O(1)

## 비유
* 해싱은 정리정돈을 잘하는 것, 물건이 필요하면 바로 찾음
* 영어사전

## 용어
* 해시 테이블 = 해시 맵 = 딕셔너리
* 해싱(hashing): key - value 쌍을 매핑하는 것
* 해시 함수(hash function) : key를 입력받아 hash code를 생성함
* 해시 코드: 해시 함수에 의해 생성됨, 해시 테이블의 인덱스로 사용
* 해시 테이블: key의 연산을 통해 데이터가 실제 저장되는 배열, M개의 버킷으로 구성됨
* 버킷(bucket): 해시 테이블의 원소, N개의 슬롯을 가질 수 있음
* 충돌(collision): 서로 다른 두개의 키가 같은 해시 코드를 가지게 되는 것, h(k1) = h(k2)

## 문제점
* 해시 테이블의 크기가 제한됨 -> 충돌 발생
* 버킷 오버플로우: 버킷의 슬롯을 모두 사용하는 경우 새로운 키 입력을 받을 수 없음
* 충돌이 발생할수록 O(1)보다 성능 떨어짐

## 해결과제
* 해시함수를 어떻게 구성할 것인가?
* 충돌이 발생했을 때 어떻게 해결할 것인가?

---

# 해시함수
## 좋은 해시함수의 조건
1. 충돌 적어야함
2. 해시코드가 골고루 분포되어야 함
3. 계산이 빨라야 함

## 해시함수의 종류
1. mod
2. 폴딩: 비트별 XOR, 키를 부분으로 나누어 더함
3. 키를 제곱한 후 중간의 몇 비트를 취함
4. 키가 문자열일 경우
    * 첫 문자의 아스키 코드값
    * 각 문자 아스키 코드값 모두 더하기
    * 각 아스키 코드값 * 자리수

---

# 충돌 해결 방법
## 개방 주소법 vs 체이닝
* 개방 주소법(Open Addressing): 충돌 발생시 다른 위치에 저장
* 체이닝(Chaining): 각 버킷을 연결리스트로 구현, 충돌 발생시 새 노드 생성

## 선형 조사법(Linear Probing)
* 충돌 발생시 그 다음 버킷이 비어있으면 그곳에 저장
* h(k), h(k)+1, h(k)+2, h(k)+3, ...
* 클러스터링(clustering) 발생 -> 성능 저하

## 이차 조사법(Quadratic Probing)
* Quadratic Equation: 이차 방정식
* h(k), h(k)+1, h(k)+4, h(k)+9, ...
* 클러스터링 완화

## 이중 해싱법(Double probing)
* 이중 해싱법 = 재해싱(rehasing)
* 충돌 발생시 별개의 해시 함수를 이용하는 방법
* h(k), h(k)+h2(k), h(k)+2*h2(k), ...

---

# 구현