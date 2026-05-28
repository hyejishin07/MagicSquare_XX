# 4×4 Magic Square — TDD 설계 보고서

**프로젝트:** Magic_Square1
**문서 유형:** TDD 설계 (Test-Driven Development Design)
**작성일:** 2026-05-28
**선행 문서:** [Magic_Square_Problem_Definition_Report.md](./Magic_Square_Problem_Definition_Report.md) (STEP 1~5)
**범위:** STEP 6-A ~ STEP 8 — *테스트 중심의 설계 명세화*

---

## 문서 목적

본 보고서는 선행 문서에서 정의된 **진짜 문제 정의(STEP 5-2)**, **7층 불변식 I~VII (STEP 5-3)**, **8가지 훈련 능력 (STEP 5-4)**, 그리고 **검증·생성·풀이·열거 활동 분해 (Why #2, Why #3)** 를 그대로 승계하여, *테스트가 곧 명세이며 명세가 곧 설계*라는 원칙 아래 4×4 Magic Square 프로그램의 **TDD 설계**를 외부화한다.

### 범위에서 명시적으로 제외되는 것

| # | 제외 항목 | 사유 |
|---|---|---|
| (a) | **구현 코드 (소스코드)** | 본 문서는 설계 명세 — Red 단계에서 작성될 테스트의 의도까지만 다룬다 |
| (b) | **알고리즘 선택** (백트래킹·제약전파·SAT 등) | "어떻게 만드는가"는 본 단계의 책임이 아님 (Why #3 결정적 한 줄 승계) |
| (c) | **성능 최적화·복잡도 분석** | 명세가 통과된 이후의 별도 단계 |
| (d) | **UI · 입출력 포맷·CLI/GUI** | 본 설계는 *순수 도메인 계약* 만 다룸 |
| (e) | **저장·영속화** | 외부 자원 경계(섹션 10)에서 형식만 언급, 실제 형식은 정의하지 않음 |

---

## 목차

0. [문서 메타](#0-문서-메타) *(상단 머리말)*
1. [TDD 설계 원칙 선언](#1-tdd-설계-원칙-선언)
2. [모듈 경계 (Module Boundaries)](#2-모듈-경계-module-boundaries)
3. [자료 모델 & 도메인 계약](#3-자료-모델--도메인-계약)
4. [테스트 분류 체계 (Test Taxonomy)](#4-테스트-분류-체계-test-taxonomy)
5. [불변식 → 테스트 매핑 (핵심 섹션)](#5-불변식--테스트-매핑-핵심-섹션)
6. [활동별 TDD 시나리오](#6-활동별-tdd-시나리오)
7. [상태 분류 사고 매핑](#7-상태-분류-사고-매핑)
8. [대칭·동치(orbit) 처리 설계](#8-대칭동치orbit-처리-설계)
9. [비결정성·시간·무작위성 격리 전략](#9-비결정성시간무작위성-격리-전략)
10. [Test Doubles & 경계](#10-test-doubles--경계)
11. [테스트 명명 규약](#11-테스트-명명-규약)
12. [CI / 실행 정책](#12-ci--실행-정책)
13. [추적성 매트릭스 (Traceability Matrix)](#13-추적성-매트릭스-traceability-matrix)
14. [다음 단계 (구현 진입 전 마지막 점검)](#14-다음-단계-구현-진입-전-마지막-점검)
15. [부록](#15-부록)

---

## 1. TDD 설계 원칙 선언

### 1.1 Red → Green → Refactor 사이클 정의

| 단계 | 의미 | 본 프로젝트에서의 운용 |
|---|---|---|
| **Red** | 아직 통과하지 못하는 테스트를 **먼저** 작성 — "옳음의 진술"이 코드보다 앞선다 | 불변식 I~VII 중 하나를 **자연어 Given/When/Then**으로 진술. 통과 수단(구현)은 이 시점에 부재해야 한다. |
| **Green** | Red 테스트를 통과시키는 **가장 작은** 변경 | 불변식의 충족이 *관찰 가능*해질 때까지만 변경. 우아함은 다음 단계의 책임. |
| **Refactor** | 통과 상태를 깨지 않으면서 중복·결합·이름·경계를 정리 | 모듈 경계(섹션 2) 위반 여부, 순수성 손상 여부, 불변식 강제 지점 분산 여부를 점검. |

### 1.2 본 프로젝트의 **Spec-First** 정의 (선행 Why #3 승계)

> **Spec-First** 란, 어떤 단위 작업이라도 *"무엇이 옳은가"* 의 진술(테스트·계약·불변식)이 *"어떻게 만드는가"* 보다 시간적으로 먼저 외부화되는 작업 순서를 말한다.

본 프로젝트에서 Spec-First는 다음을 의미한다.

| 요구 | 내용 |
|---|---|
| **시간적 선행** | 모든 모듈은 *공개 계약 (섹션 6.1)* 과 *Red 테스트 목록 (섹션 6.2)* 이 정의된 뒤에만 Green 단계로 진입한다. |
| **표현 매체의 분리** | 명세는 자연어(Given/When/Then)와 술어로, 구현은 코드로 — 두 매체를 섞지 않는다. |
| **공통 기준의 단일화** | 검증·생성·풀이·열거는 모두 **동일한 술어 집합**(불변식 I~VII)을 공유한다. (Why #3 핵심 발견 승계) |

### 1.3 테스트의 4가지 역할

| 역할 | 내용 | 본 문서에서의 위치 |
|---|---|---|
| **(R1) 명세 (Specification)** | "옳음"을 외부화한 1차 진술 | 섹션 5 (불변식 → 테스트 매핑) |
| **(R2) 회귀 방지 (Regression Guard)** | 한 번 통과한 진술이 다시 깨지지 않도록 고정 | 섹션 12 (결정성 회귀 빈도) |
| **(R3) 설계 압력 (Design Pressure)** | 테스트하기 어려운 구조 = 잘못된 구조라는 신호 | 섹션 10 (Doubles 경계) — "필요 없음"을 먼저 진술 |
| **(R4) 문서 (Documentation)** | 의도가 코드 옆에서 항상 최신 상태로 유지되는 산 문서 | 섹션 11 (명명 규약), 섹션 13 (추적성 매트릭스) |

**핵심 선언:**

> *본 프로젝트의 테스트는 "버그를 잡는 그물"이 아니라, **"옳음의 진술 그 자체"** 다.* (선행 문서 Why #3 결정적 한 줄 승계)

---

## 2. 모듈 경계 (Module Boundaries)

선행 문서 Why #2의 활동 분해 — **검증 · 생성 · 풀이 · 열거** — 와 Why #3의 *모듈 분해 가능성* 발견을 6개 모듈로 승격한다.

### 2.1 모듈 책임·계약·의존 표

| 모듈 | 책임 (한 문장) | 입력 계약 | 출력 계약 | 의존 |
|---|---|---|---|---|
| **Grid** (자료 표현) | 4×4 격자와 셀의 표현·접근·동등성을 정의 | 16칸의 셀 값 (값 또는 빈칸 표식) | 불변(immutable) Grid 인스턴스, 셀 접근자, 동등 판정 | — |
| **Validator** (검증기) | 임의 Grid 가 술어 *Magic* 을 어느 분류로 충족하는지 판정 | Grid (완성·부분·빈 격자 모두) | `{Satisfied, PartiallyConsistent, Violated}` 중 정확히 하나 | Grid |
| **Solver** (풀이기) | 부분 격자로부터 *Magic* 을 충족하는 완성 격자(존재 시)를 산출 | 부분 격자 + 도메인 명세 | 완성 Grid 또는 *해 없음* | Validator, Grid |
| **Generator** (생성기) | 외부 시드를 받아 *Magic* 을 충족하는 완성 격자를 만든다 | 시드(seed) + 도메인 명세 | 완성 Grid (동일 시드 → 동일 결과) | Validator, Grid |
| **Enumerator** (열거기) | *Magic* 을 충족하는 격자 전체 또는 그 대표원(orbit) 의 유한 집합을 산출 | 도메인 명세 + 대표원 선택 규칙 | 완성 Grid 의 유한 시퀀스 | Validator, Symmetry, Grid |
| **Symmetry** (대칭/동치) | 회전·반사 변환과 orbit 동등성을 외부화 | Grid, 변환 종류 | 변환된 Grid 또는 orbit 대표원 | Grid |

### 2.2 모듈별 순수성·부수효과·비결정성 위치

| 모듈 | 순수 함수 여부 | 부수효과 발생 위치 | 비결정성 격리 지점 |
|---|---|---|---|
| **Grid** | 완전 순수 (불변 객체) | 없음 | 없음 |
| **Validator** | 완전 순수 | 없음 | 없음 — 같은 Grid 입력은 언제나 같은 분류를 반환 (불변식 **VII** 직접 강제) |
| **Solver** | 완전 순수 (탐색 순서 고정 시) | 없음 | 탐색 순서가 결정성을 가지도록 *정렬 규칙*을 계약에 포함 (섹션 9.2) |
| **Generator** | **불순 입력 → 순수 함수화** | 없음 (시드 주입 후) | **시드 주입 경계** 에서만 비결정성 허용. 함수 본체는 시드에 대해 결정적. (섹션 9.1) |
| **Enumerator** | 완전 순수 (대표원 규칙 고정 시) | 없음 | 출력 시퀀스 순서 결정성은 *대표원 정렬 규칙*에 의해 강제 |
| **Symmetry** | 완전 순수 | 없음 | 없음 |

> **설계 원칙:** *비결정성은 Generator 의 입력 경계(시드)에만 존재한다.* 그 외의 모든 모듈은 **참조 투명(referentially transparent)** 하다.

### 2.3 모듈 의존 그래프 (계약 수준)

```
        ┌─────────┐
        │  Grid   │  (자료 표현, 잎 노드)
        └────┬────┘
             │
   ┌─────────┼─────────┬──────────────┐
   │         │         │              │
┌──▼──┐  ┌───▼────┐  ┌─▼──────┐  ┌───▼──────┐
│Sym. │  │Validator│  │Solver │  │Generator │
└──┬──┘  └───┬────┘  └────────┘  └──────────┘
   │         │
   └────┬────┘
        │
   ┌────▼─────┐
   │Enumerator│
   └──────────┘
```

- 화살표는 *의존*. 모든 활동 모듈(Validator/Solver/Generator/Enumerator)은 Grid 위에서, 그리고 검증 모듈(Validator)을 **공통 기준**으로 삼는다. (선행 문서 Why #3 "공통 기준" 승계, 훈련 능력 **#5 검증과 구성의 분리**, **#7 모듈 분해 사고**)

---

## 3. 자료 모델 & 도메인 계약

### 3.1 `Grid` 표현 규칙

| 속성 | 정의 | 근거 (선행 문서) |
|---|---|---|
| **차원** | 항상 4×4 — 다른 차원은 형태 위반으로 입력 거부 | STEP 5-3 (I), STEP 1 관찰 |
| **셀 수** | 정확히 16 | STEP 1 |
| **셀 도메인 (값)** | 유한 정수 집합 \(D = \{1, 2, \dots, 16\}\) | STEP 5-3 (I), 부록 용어 |
| **AllDifferent 도메인 적용** | 채워진 셀 값들은 서로 다르며 모두 \(D\) 의 원소 | STEP 5-3 (I) |
| **불변성** | Grid 인스턴스는 생성 후 변경 불가. 한 칸 변경은 *새 Grid* 의 산출이다. | 훈련 능력 #2, #5 |
| **동등성** | 같은 16칸 값(같은 위치)을 가지면 동등. (대칭에 의한 동등은 별개 — 섹션 8) | 불변식 VI 와 분리 |

### 3.2 `Cell` / `EmptyCell` 표현 정책

| 표현 후보 | 채택 여부 | 사유 |
|---|---|---|
| **Sentinel `0`** | ❌ | "0" 이 도메인 외부지만 동일한 정수 타입을 사용 → *형태 위반*과 *빈칸*의 구분이 흐려진다 (불변식 I 강제 약화) |
| **음수 / 매직 넘버** | ❌ | 도메인 외부 정수의 의미 과부하 |
| **선택형 (Optional / Maybe)** | ✅ | "값 있음/없음" 을 타입 시스템 차원에서 분리. 빈칸은 *없음*, 값은 \(D\) 의 원소. |
| **별도 `EmptyCell` 토큰 + `FilledCell(value)`** | ✅ (Optional 의 동의어) | 의도가 더 또렷한 경우 채택 가능. 본 설계는 두 표현을 **동형(isomorphic)** 으로 본다. |

> **결정:** `Cell ::= Empty | Filled(value ∈ D)` (언어 중립 의사 표기)

### 3.3 입력 검증 — 형태 위반 시의 동작

| 위반 종류 | 동작 정책 | 근거 |
|---|---|---|
| **차원 위반** (4×4 아님) | **계약 위반 예외** — *판정 이전에* 거부 | 불변식 I 의 전제 조건 |
| **셀 값 타입 위반** | 계약 위반 예외 | 동상 |
| **값 \(\notin D\)** (예: 17, 0, -1) | 계약 위반 예외 | 도메인 외부는 *Violated* 가 아니라 *형태 위반* |
| **AllDifferent 위반** | **결과형 `Violated`** — 형태는 맞으나 술어 위반 | 불변식 I 의 술어 부분 |
| **합 제약 위반** | 결과형 `Violated` | 불변식 II, III, IV |
| **부분 격자 결정 모순** | 결과형 `Violated` | 불변식 V |

> **원칙 (훈련 능력 #3 계약 기반 사고):** *형태(shape)는 예외로, 술어(predicate)는 결과형으로 분리한다.*

### 3.4 불변식 → 타입/계약 강제 지점 매핑

| 불변식 | 자연어 | 강제 위치 | 강제 수단 |
|---|---|---|---|
| **I** 도메인 | 모든 채워진 값은 \(D\) 안, 서로 다름 | Grid 생성자 + Validator | 생성자에서 *값 외부* 즉시 거부 / Validator 에서 AllDifferent 검사 |
| **II** 마법합 \(M=34\) | \(M = n(n^2+1)/2\), n=4 → 34 | Validator (상수 도출) | 상수는 *도출* 되어야지 *고정 입력* 이 되어선 안 됨 |
| **III** 합 균등 | 모든 행·열·주대각선 합 = M | Validator | 10개 합 검사 술어의 동시 충족 |
| **IV** 전체합 일관성 | \(\sum = n \cdot M = 136\) | Validator (필요조건 사전 검사) | III 의 필요조건 — III 충족 시 자동 충족 |
| **V** 부분 일관성 | 부분 격자에서도 결정된 합 제약은 위반 불가 | Validator (`PartiallyConsistent` 분류) | 부분합 ≤ M 또는 *확정 위반* 조건 검사 |
| **VI** 대칭 닫힘 | 회전·반사 후에도 마방진 | Symmetry + Validator | Symmetry 적용 후 Validator 결과 동일성 |
| **VII** 판정 결정성 | 같은 입력 → 같은 결과 (시간·실행·환경 무관) | Validator (그리고 모든 술어 모듈) | 순수성 강제, 시드 외 외부 자원 의존 금지 |

---

## 4. 테스트 분류 체계 (Test Taxonomy)

### 4.1 5계층 정의

| 계층 | 한 줄 정의 | 적용 모듈 |
|---|---|---|
| **L1 Unit** | 외부 의존 없는 순수 함수·술어의 단일 동작 검증 | Grid, Validator(개별 술어), Symmetry(단일 변환) |
| **L2 Contract** | 모듈 공개 계약(입력 도메인 × 출력 분류)의 충족 검증 | Validator, Solver, Generator, Enumerator |
| **L3 Invariant / Property-Based** | 무작위 입력 다수 표본에 대해 불변식 I~VII 충족을 검증 | Validator, Symmetry (전체 불변식), Solver/Generator (출력 측 불변식) |
| **L4 Equivalence (대칭)** | 회전·반사 orbit 내 동등성 / orbit 대표원 단일성 검증 | Symmetry, Enumerator |
| **L5 Integration** | 두 개 이상 모듈의 협력이 공통 기준(Validator)에 합치하는지 검증 | Solver→Validator, Generator→Validator, Enumerator→Validator+Symmetry |

### 4.2 적용 매트릭스 (모듈 × 계층)

| 모듈 / 계층 | L1 | L2 | L3 | L4 | L5 |
|---|:-:|:-:|:-:|:-:|:-:|
| Grid | ● | ● | — | — | — |
| Validator | ● | ● | ● | ● | — |
| Solver | — | ● | ● | — | ● |
| Generator | — | ● | ● | — | ● |
| Enumerator | — | ● | ● | ● | ● |
| Symmetry | ● | ● | ● | ● | — |

> ● = 필수 적용, — = 본 설계 단계에서 비대상

---

## 5. 불변식 → 테스트 매핑 (핵심 섹션)

선행 문서 STEP 5-3 의 7층 불변식 I~VII 각각을 **최소 1개 이상**의 테스트 시나리오로 매핑한다. 모든 진술은 자연어 Given/When/Then 으로만 표현한다. *(훈련 능력 #2 불변식 중심 사고)*

---

### (I) 도메인 — *모든 채워진 값은 \(D=\{1..16\}\), 서로 다름*

| 항목 | 내용 |
|---|---|
| **자연어 진술** | "Grid 안의 채워진 셀 값들은 모두 \(\{1,\dots,16\}\) 의 원소이며, 같은 값이 두 번 등장하지 않는다." |
| **Given** | 16칸이 모두 채워진 Grid, 단 두 셀이 같은 값을 가짐 |
| **When** | Validator 에 해당 Grid 를 전달 |
| **Then** | 분류는 `Violated`. 위반 근거에 *AllDifferent* 가 포함된다. |
| **테스트 계층** | L1 Unit + L3 Property-Based (랜덤 중복 주입) |
| **실패 시 의미** | "도메인 가정이 무너졌다" — 시스템 전체가 불변식 II~VII 를 논할 자격을 잃는다. |

| Test ID | 보조 시나리오 |
|---|---|
| **T-I-1** | 형태 위반: 17 또는 0 포함 Grid → *계약 위반 예외* (결과형 아님) |
| **T-I-2** | AllDifferent 위반: 7이 두 번 → `Violated` |
| **T-I-3** | (PBT) 임의의 부분 격자에서도 채워진 값은 항상 \(D\) 의 원소 |

---

### (II) 마법합 \(M = n(n^2+1)/2\), \(n=4\) → \(M=34\)

| 항목 | 내용 |
|---|---|
| **자연어 진술** | "마법합 M은 도메인 크기 n으로부터 도출되며, 외부에서 고정 주입되지 않는다." |
| **Given** | n=4 도메인 명세만 입력 |
| **When** | 시스템에 M 을 질의 |
| **Then** | M = 34. 동시에 일반식 \(M = n(n^2+1)/2\) 으로 도출됨을 확인. |
| **테스트 계층** | L1 Unit (수식 도출 함수) + L3 (가상의 n 변형에 대한 식 검증) |
| **실패 시 의미** | M 이 매직 넘버로 굳어졌다 — *규칙의 외부화* (훈련 능력 #1) 가 실패. |

| Test ID | 보조 시나리오 |
|---|---|
| **T-II-1** | 수식 도출: n=3 → M=15, n=5 → M=65 (도출식의 일반성 확인, 본 프로젝트의 채택은 n=4) |

---

### (III) 합 균등 — *모든 행·열·주대각선 합 = M*

| 항목 | 내용 |
|---|---|
| **자연어 진술** | "완성된 Grid 가 마방진이려면, 4행·4열·2주대각선 = 총 10개 합이 모두 M과 같아야 한다." |
| **Given** | 마방진으로 알려진 표준 격자 (예: 알브레히트 뒤러 격자) |
| **When** | Validator 호출 |
| **Then** | 분류 = `Satisfied`. 10개 합이 모두 34 임이 부수 정보로 확인 가능. |
| **테스트 계층** | L1 Unit + L2 Contract |
| **실패 시 의미** | 술어 정의가 누락되었다 — 검증과 구성의 공통 기준이 깨졌다. |

| Test ID | 보조 시나리오 |
|---|---|
| **T-III-1** | 한 행 합 = 33 → `Violated` |
| **T-III-2** | 모든 행·열 OK, 한 대각선 = 35 → `Violated` (대각선 누락 회귀 방지) |
| **T-III-3** | (PBT) 임의 마방진을 회전/반사한 결과에 대해서도 동일하게 `Satisfied` (불변식 VI 와 교차) |

---

### (IV) 전체합 일관성 — \(\sum = n \cdot M = 136\)

| 항목 | 내용 |
|---|---|
| **자연어 진술** | "모든 셀 값의 합은 \(n \cdot M\) 과 같다 — III 의 필요조건이며, 불일치는 즉시 `Violated`." |
| **Given** | 16칸 모두 채워졌으나 전체합이 135 또는 137 인 Grid |
| **When** | Validator 호출 |
| **Then** | 분류 = `Violated`. III 검사 이전 단계에서 거부 가능 (단, 결과는 동일). |
| **테스트 계층** | L1 + L2 |
| **실패 시 의미** | 필요조건 검사가 누락 — 검증 비용이 불필요하게 증가, 명세의 *층 구조* 가 무너졌다. |

| Test ID | 보조 시나리오 |
|---|---|
| **T-IV-1** | 완성 격자 합 = 136 인데도 III 위반 → `Violated` (필요조건은 만족, 충분조건 미충족) |

---

### (V) 부분 일관성 — *부분 격자에서도 결정된 합 제약은 위반 불가*

| 항목 | 내용 |
|---|---|
| **자연어 진술** | "부분 격자에서, 어떤 행/열/대각선이 *완전히 채워졌는데 합 ≠ M* 이면 `Violated`. 그 외 부분 충족은 `PartiallyConsistent`." |
| **Given** | 첫 행이 완전히 채워졌고 그 합 = 33, 나머지 12칸은 빈 격자 |
| **When** | Validator 호출 |
| **Then** | 분류 = `Violated` (이미 결정된 행이 M 을 위반) |
| **테스트 계층** | L1 + L2 + L3 |
| **실패 시 의미** | 부분 일관성 분류가 누락 → Solver 가 *살아 있는 가지치기 신호* 를 받지 못한다. |

| Test ID | 보조 시나리오 |
|---|---|
| **T-V-1** | 빈 격자 → `PartiallyConsistent` (어떤 결정도 아직 위반 아님) |
| **T-V-2** | 한 행만 채워졌고 합 = 34 → `PartiallyConsistent` |
| **T-V-3** | 두 칸만 채워졌고 같은 값 → `Violated` (AllDifferent 가 부분 격자에서도 적용) |

---

### (VI) 대칭 닫힘 — *회전·반사에 대해 마방진 집합이 닫혀 있음*

| 항목 | 내용 |
|---|---|
| **자연어 진술** | "임의의 마방진 G 에 대해, 8개 대칭(회전 0°/90°/180°/270° × 반사 ±) 변환의 결과도 모두 마방진이다." |
| **Given** | 표준 마방진 G |
| **When** | Symmetry 모듈로 8개 변환 모두 적용한 \(\{G_1, \dots, G_8\}\) 을 Validator 에 전달 |
| **Then** | 8개 모두 `Satisfied` |
| **테스트 계층** | L4 Equivalence + L5 Integration (Symmetry × Validator) |
| **실패 시 의미** | 대칭 모듈이 술어를 보존하지 않는다 — *동등성 사고*(훈련 능력 #6) 가 코드에 외부화되지 않았다. |

| Test ID | 보조 시나리오 |
|---|---|
| **T-VI-1** | 8개 변환 후 orbit 대표원이 유일하게 결정됨 (정렬 규칙의 안정성) |
| **T-VI-2** | (PBT) 임의 마방진 표본에 대해 변환 결과의 `Satisfied` 비율 = 100% |

---

### (VII) 판정 결정성 — *같은 입력 → 같은 결과 (시간·실행 횟수 무관)*

| 항목 | 내용 |
|---|---|
| **자연어 진술** | "Validator·Symmetry·Solver(고정 정렬)·Generator(고정 시드)는 같은 입력에 대해 언제 어디서 호출되어도 동일한 출력을 낸다." |
| **Given** | 동일한 Grid 또는 동일한 시드 |
| **When** | 동일 호출을 N회 (예: 1000회) 반복 |
| **Then** | 모든 호출 결과가 비트 단위로 동일 |
| **테스트 계층** | L3 Property-Based + L5 Integration |
| **실패 시 의미** | 어딘가에서 *숨은 외부 자원*(시계, 전역 상태, 비결정 자료구조 순회)이 침투했다. *비결정성의 격리* 가 실패. |

| Test ID | 보조 시나리오 |
|---|---|
| **T-VII-1** | Solver: 정렬 규칙을 명시한 부분 격자 입력 → 1000회 호출 모두 동일 완성 격자 |
| **T-VII-2** | Generator: 시드 = 42 → 1000회 모두 동일 격자 |
| **T-VII-3** | Enumerator: 출력 시퀀스의 i 번째 원소가 호출 간 동일 |

---

## 6. 활동별 TDD 시나리오

선행 문서 Why #2 의 활동 4분해 — **Validator / Solver / Generator / Enumerator** — 각각에 대해 (1) 공개 계약, (2) Red 단계 테스트 목록, (3) Green 단계의 통과 조건, (4) Refactor 단계의 관심사를 사전 진술한다.

---

### 6.A Validator (검증기)

#### (1) 공개 계약 (의사 시그니처)

```
validate(grid: Grid) -> ValidationResult

ValidationResult ::= Satisfied
                   | PartiallyConsistent
                   | Violated(reasons: Set<ViolationReason>)

ViolationReason ::= DomainOutOfRange       // 형태 위반은 예외로 별도
                  | NotAllDifferent
                  | RowSumMismatch(row: Int, sum: Int)
                  | ColSumMismatch(col: Int, sum: Int)
                  | DiagSumMismatch(diag: MainOrAnti, sum: Int)
                  | TotalSumMismatch(sum: Int)
                  | PartiallyDecidedViolation(line: LineRef)
```

#### (2) Red 단계 테스트 목록 (최소 5)

| Test ID | 이름 (의도) |
|---|---|
| V-R1 | `validate_emptyGrid_returnsPartiallyConsistent` — 빈 격자는 위반 아님 |
| V-R2 | `validate_completeMagic_returnsSatisfied` — 표준 마방진 → Satisfied |
| V-R3 | `validate_completeNonMagic_returnsViolatedWithReasons` — 합 위반 시 이유 동봉 |
| V-R4 | `validate_partialWithFullyDecidedRowViolation_returnsViolated` — 불변식 V |
| V-R5 | `validate_outOfDomainValue_throwsContractException` — 형태 위반은 예외 |
| V-R6 | `validate_sameGridTwice_returnsIdenticalResult` — 불변식 VII |

#### (3) Green 통과 조건 (불변식 참조)

- V-R1: 불변식 V 의 *어떤 결정도 위반 아닌 부분 격자* 정의 충족
- V-R2: 불변식 III + IV + I 동시 충족
- V-R3: 불변식 III 위반 시 `reasons` 가 *비어 있지 않음*
- V-R4: 불변식 V 의 *완전히 채워진 라인의 합 ≠ M* 조건 식별
- V-R5: 섹션 3.3 형태/술어 분리 정책 충족
- V-R6: 불변식 VII 의 N=2 인스턴스

#### (4) Refactor 관심사

| 관심사 | 점검 |
|---|---|
| **중복** | 행/열/대각선 합 검사가 3개 코드 경로로 분기되지 않는가 (라인 추상화) |
| **결합** | Validator 가 Solver/Generator 자료구조에 의존하지 않는가 |
| **이름** | `Satisfied/PartiallyConsistent/Violated` 가 모듈 경계에서 동일 용어로 통용되는가 |
| **경계** | "형태 위반 = 예외" vs "술어 위반 = 결과형" 의 경계가 한 곳에서만 결정되는가 |

---

### 6.B Solver (풀이기)

#### (1) 공개 계약

```
solve(partial: Grid, ordering: OrderingRule) -> SolveResult

SolveResult ::= Solved(grid: Grid)
              | NoSolution
              | Inconsistent          // 입력이 이미 Violated

// ordering 은 탐색 순서를 결정성으로 만들기 위한 외부 주입 (섹션 9.2)
```

#### (2) Red 단계 테스트 목록 (최소 5)

| Test ID | 이름 (의도) |
|---|---|
| S-R1 | `solve_alreadySatisfied_returnsSameGrid` — 항등성 |
| S-R2 | `solve_inconsistentPartial_returnsInconsistent` — Validator 와의 일치 |
| S-R3 | `solve_partialWithUniqueCompletion_returnsThatCompletion` — 정합성 |
| S-R4 | `solve_emptyGrid_returnsSomeSatisfiedGrid` — 존재성 |
| S-R5 | `solve_sameInputAndOrdering_returnsIdenticalResult` — 결정성 (불변식 VII) |
| S-R6 | `solve_returnedGrid_passesValidator` — Solver→Validator 일치 (L5) |

#### (3) Green 통과 조건

- S-R1, S-R6: 모든 반환 격자는 Validator 결과가 `Satisfied`
- S-R2: 입력이 `Violated` 이면 `Inconsistent` (절대 `Solved` 가 아님)
- S-R3: 유일 완성 가능 부분 격자에 대해 반환은 그 유일 완성과 동등
- S-R5: 동일 (partial, ordering) → 동일 SolveResult — 1000회 검증

#### (4) Refactor 관심사

| 관심사 | 점검 |
|---|---|
| 중복 | 부분 일관성 검사가 Validator 와 분리 구현되어 *두 진실(double truth)* 이 생기지 않았는가 |
| 결합 | 탐색 진행 중 *Validator 만* 술어 판단의 단일 출처가 되는가 |
| 이름 | `NoSolution` / `Inconsistent` 분리가 외부에서 즉시 이해되는가 |
| 경계 | Ordering 주입 경계가 함수 시그니처에 명시적으로 드러나는가 |

---

### 6.C Generator (생성기)

#### (1) 공개 계약

```
generate(seed: Seed, domain: DomainSpec) -> Grid    // 항상 Satisfied 보장
```

#### (2) Red 단계 테스트 목록 (최소 5)

| Test ID | 이름 (의도) |
|---|---|
| G-R1 | `generate_anySeed_returnsSatisfiedGrid` — 출력 측 불변식 (L3) |
| G-R2 | `generate_sameSeed_returnsIdenticalGrid` — 결정성 (불변식 VII) |
| G-R3 | `generate_differentSeeds_mayReturnDifferentGrids` — 시드 의존성 입증 (단 동등은 허용) |
| G-R4 | `generate_resultPassesValidator` — Generator→Validator 일치 (L5) |
| G-R5 | `generate_noGlobalStateRead` — 시드 외 외부 자원 미사용 |
| G-R6 | `generate_resultRespectsDomain` — 출력 값 범위 \(\subseteq D\) (불변식 I) |

#### (3) Green 통과 조건

- G-R1, G-R4, G-R6: 모든 반환 격자에 대해 Validator 가 `Satisfied`
- G-R2: 동일 시드의 N회 호출 결과가 모두 동등
- G-R5: 테스트 환경에서 시계/난수 전역 상태 접근 없음 (Doubles 로 차단, 섹션 10)

#### (4) Refactor 관심사

| 관심사 | 점검 |
|---|---|
| 중복 | Validator 통과 검사가 Generator 내부에 *중복 구현* 되지 않았는가 |
| 결합 | 시드 타입이 함수 외부에서 의미를 가지지 않는가 (불투명한 의사 난수원) |
| 이름 | `Seed`, `DomainSpec` 이 도메인 어휘인가 |
| 경계 | 무작위성의 출처가 *시드 단일 경계* 인가 (섹션 9.1) |

---

### 6.D Enumerator (열거기)

#### (1) 공개 계약

```
enumerate(scope: EnumScope, repr: RepresentativeRule) -> FiniteSequence<Grid>

EnumScope ::= AllSolutions          // 7040 개 (대칭 포함)
            | EssentiallyDistinct   // 880 개 (orbit 대표원)
```

#### (2) Red 단계 테스트 목록 (최소 5)

| Test ID | 이름 (의도) |
|---|---|
| E-R1 | `enumerate_essentiallyDistinct_returnsExactly880` — 본질해 수 (선행 문서 STEP 1) |
| E-R2 | `enumerate_allSolutions_returnsExactly7040` — 8 × 880 (대칭 닫힘) |
| E-R3 | `enumerate_everyElement_isSatisfied` — 출력 측 불변식 (L5) |
| E-R4 | `enumerate_noTwoElements_areOrbitEquivalent` (EssentiallyDistinct) — 대표원 유일성 (L4) |
| E-R5 | `enumerate_sameScope_returnsIdenticalSequence` — 결정성 (불변식 VII) |
| E-R6 | `enumerate_unionOfOrbits_coversAllSolutions` — 대칭과 전체의 일치 (L4 ∪ L5) |

#### (3) Green 통과 조건

- E-R1: 시퀀스 길이 = 880
- E-R2: 시퀀스 길이 = 7040
- E-R3: 모든 원소가 Validator `Satisfied`
- E-R4: 임의 두 원소가 Symmetry 의 어떤 변환으로도 동등이 아님
- E-R6: `EssentiallyDistinct` 각 원소의 orbit 합집합 = `AllSolutions` 의 집합

#### (4) Refactor 관심사

| 관심사 | 점검 |
|---|---|
| 중복 | 대표원 선택 규칙이 Symmetry 와 Enumerator 양쪽에 *복제* 되지 않았는가 |
| 결합 | Enumerator 가 Solver 의 내부 표현에 의존하지 않는가 |
| 이름 | `EnumScope`, `RepresentativeRule` 이 외부 사용자에게 명시적인가 |
| 경계 | "본질해" 와 "전체해" 의 경계가 한 매개변수로만 표현되는가 |

---

## 7. 상태 분류 사고 매핑

선행 문서 STEP 5-2 (c) 의 3분류 `Satisfied / PartiallyConsistent / Violated` 를 세 가지 형식으로 외부화한다. *(훈련 능력 #4 상태 분류 사고)*

### 7.1 경계 사례 (Boundary Cases)

| 경계 | 입력 형태 | 기대 분류 |
|---|---|---|
| **B1** 완전 빈 격자 (16칸 모두 Empty) | 모든 칸 Empty | `PartiallyConsistent` |
| **B2** 1칸만 채워짐 | 임의 값 ∈ D | `PartiallyConsistent` |
| **B3** 15칸 채워짐, 1칸 Empty | 부분 격자 | `PartiallyConsistent` (단, 결정된 라인이 M 위반이 아닐 때) |
| **B4** 16칸 채워짐, 1행 합만 33 | 거의 완성 | `Violated` |
| **B5** 16칸 채워짐, 모든 합 = 34 | 완성 마방진 | `Satisfied` |
| **B6** 1행만 완전히 채워졌고 합 = 35 | 부분 + 라인 결정 | `Violated` (불변식 V) |
| **B7** 2칸 모두 값 7 | 부분 + AllDifferent 위반 | `Violated` |
| **B8** 16칸 채워짐, 합 모두 34, but 한 대각선만 33 | 회귀 함정 | `Violated` (대각선 누락 회귀 방지) |

### 7.2 동치류 (Equivalence Classes)

| EC | 정의 | 대표 사례 |
|---|---|---|
| **EC-S** *Satisfied* | 16칸 모두 채워졌으며 불변식 I·III·IV 동시 충족 | 880개 본질해 + 7개 대칭 동등 = 7040개 |
| **EC-PC** *PartiallyConsistent* | (a) 미완성이며 (b) 어떤 결정도 위반 아님 | 첫 행 합 = 34 인 부분 격자 |
| **EC-V-AllDiff** *Violated · AllDifferent* | 중복 값 존재 (완성 여부 무관) | 두 칸이 같은 값 |
| **EC-V-RowCol** *Violated · 행/열 합 위반* | 완성됐으나 행 또는 열 ≠ M | T-III-1 |
| **EC-V-Diag** *Violated · 대각선 위반* | 행·열 OK, 대각선 위반 | T-III-2 / B8 |
| **EC-V-Total** *Violated · 전체합 위반* | \(\sum \ne 136\) | T-IV (드물지만 가능) |
| **EC-V-PartialDecided** *Violated · 부분 결정 위반* | 결정된 라인이 M 위반 | B6 |
| **EC-CV-Domain** *형태 위반* (계약 위반 예외) | 17 / 0 / -1 / 비4×4 | T-I-1 — *결과형이 아닌 예외 경로* |

### 7.3 결정 트리 (Decision Table)

| 형태 OK? | AllDifferent | 16칸 모두 채워짐? | 결정된 라인 위반? | 모든 합 = M? | 분류 |
|:-:|:-:|:-:|:-:|:-:|---|
| ✗ | — | — | — | — | **예외** (DomainOutOfRange) |
| ✓ | ✗ | — | — | — | **Violated** (AllDifferent) |
| ✓ | ✓ | ✗ | ✓ | — | **Violated** (불변식 V) |
| ✓ | ✓ | ✗ | ✗ | — | **PartiallyConsistent** |
| ✓ | ✓ | ✓ | — | ✗ | **Violated** (불변식 III/IV) |
| ✓ | ✓ | ✓ | — | ✓ | **Satisfied** |

> **원칙:** 모든 입력은 위 표의 *정확히 한 행* 에 매핑된다 (분류의 *전체성* + *상호배타성*). 이 두 성질은 별도 메타 테스트 **T-Class-Total** 과 **T-Class-Disjoint** 로 강제한다.

---

## 8. 대칭·동치(orbit) 처리 설계

### 8.1 대칭군의 구조와 책임 위치

| 변환 | 정의 | 책임 모듈 |
|---|---|---|
| **회전 0°** \(R_0\) (항등) | 변경 없음 | Symmetry |
| **회전 90°** \(R_{90}\) | 시계 방향 90° | Symmetry |
| **회전 180°** \(R_{180}\) | 시계 방향 180° | Symmetry |
| **회전 270°** \(R_{270}\) | 시계 방향 270° | Symmetry |
| **반사 (수평)** \(F\) | 좌우 반전 | Symmetry |
| **반사 ∘ 회전 90°** \(F \circ R_{90}\) | 합성 | Symmetry |
| **반사 ∘ 회전 180°** \(F \circ R_{180}\) | 합성 | Symmetry |
| **반사 ∘ 회전 270°** \(F \circ R_{270}\) | 합성 | Symmetry |

> 위 8원소가 위수 8의 이면체군 \(D_4\) 를 이룬다. (선행 문서 STEP 1, STEP 4)

### 8.2 orbit 와 대표원 (Representative)

- **orbit(G)** \(= \{ \sigma(G) : \sigma \in D_4 \}\), 크기는 1 ~ 8 (자기대칭의 정도에 따라 가변, 4×4 마방진의 경우 대부분 8)
- **대표원 (Canonical Representative)** : 한 orbit 에서 *전순서(total order)* 상 최소 원소 하나를 선택. 본 설계의 채택 순서는:
  1. 좌상단 셀 값 오름차순
  2. 동률이면 행 우선 평탄화 시퀀스의 사전순(lexicographic)

> **선택 기준의 원칙:** 대표원 규칙은 *결정적(deterministic)* 이고 *전순서적(total)* 이어야 한다. 그래야 불변식 VII 와 충돌하지 않는다.

### 8.3 "본질해 880" 의 계수 가능성(countable) 검증 전략

| 전략 | 내용 | 적용 테스트 |
|---|---|---|
| **(C1) 직접 계수** | EnumScope = `EssentiallyDistinct` 출력 길이 = 880 | E-R1 |
| **(C2) 간접 계수** | EnumScope = `AllSolutions` 길이 = 7040, 그리고 \(7040 / 8 = 880\) | E-R2 + 정수 나눗셈 검증 |
| **(C3) orbit 분할 검증** | EssentiallyDistinct 원소의 orbit 합집합이 AllSolutions 와 *집합 동일* | E-R6 |
| **(C4) 대표원 유일성** | EssentiallyDistinct 내 임의 두 원소가 orbit 동등이 아님 | E-R4 |
| **(C5) 자기대칭 처리** | orbit 크기가 8보다 작은 케이스가 본 문제(4×4 마방진 전체해)에서 어떻게 처리되는지 명시적으로 검증 (없으면 *없음* 을 검증) | E-R-AutoSym (보조) |

> *(훈련 능력 #6 대칭과 동치 사고)* — "세는 단위" 의 정의가 곧 테스트의 일부가 된다.

---

## 9. 비결정성·시간·무작위성 격리 전략

### 9.1 Generator 의 시드 주입 계약

| 요구 | 내용 |
|---|---|
| **단일 출처 원칙** | 무작위성은 함수 인자 `seed: Seed` 만을 통해 들어온다. 모듈 내부에서는 시계·전역 PRNG·환경 변수·파일·네트워크를 읽지 않는다. |
| **불투명성** | `Seed` 의 내부 구조는 호출자에게 의미를 갖지 않는다 (블랙박스) — 단 동등 비교는 가능. |
| **재현성** | 동일 `(seed, domain)` → 동일 Grid. 불변식 VII 의 직접 구현. |
| **외부 변경 차단** | 호출 중 시드는 *복사* 되며, 호출 종료 후 호출자의 시드 객체는 변경되지 않는다. |

#### 결정성 회귀 테스트 (G-R2 의 강화)

| 시나리오 | 기대 |
|---|---|
| 같은 시드 1000회 호출 → 모든 결과 동등 | 100% 동등 |
| 같은 시드, 서로 다른 두 환경(예: OS, 컴파일러)에서 호출 → 동등 | 100% 동등 (단, 부동소수 미사용 환경에서) |

### 9.2 Solver 의 탐색 순서 결정성

| 요구 | 내용 |
|---|---|
| **OrderingRule 외부 주입** | 빈 칸 선택, 후보값 시도 순서, 가지치기 우선순위 모두 `OrderingRule` 에 캡슐화 |
| **기본값(default) 의 결정성** | 별도 지정이 없을 때 적용되는 기본 OrderingRule 역시 *입력 부분 격자에 대해 결정적* |
| **자료구조 순회의 결정성** | 해시 기반 컬렉션 등 *순회 순서가 외부 영향을 받는 자료구조* 의 사용을 본 모듈은 금한다 (혹은 사용 시 *명시적 정렬* 을 통과한 결과만 외부 노출) |

### 9.3 시간 의존성 차단

| 종류 | 정책 |
|---|---|
| **현재 시각 (now)** | 본 도메인 모듈에서 호출 금지 (Validator/Solver/Generator/Enumerator/Symmetry/Grid) |
| **타임아웃** | 도메인 결과에 *영향을 미치지 않는* 외곽(예: CI 러너) 에서만 적용. 결과가 동일해야 한다. |
| **로깅 타임스탬프** | 결과 객체의 일부가 되지 않는다 (`equals` 에서 제외) |

---

## 10. Test Doubles & 경계

### 10.1 Doubles 가 **필요 없는** 영역 (먼저 진술)

| 모듈 | Doubles 불요 사유 |
|---|---|
| **Grid** | 불변·순수 — 진짜 인스턴스로 테스트 가능 |
| **Validator** | 외부 의존 없음 — 진짜 Grid 입력으로 충분 |
| **Symmetry** | 순수 변환 — 진짜 Grid 로 충분 |
| **Solver** (OrderingRule 주입형) | OrderingRule 은 *값* 으로 주입 가능 (인터페이스 mock 불필요) |
| **Enumerator** | 출력 시퀀스를 *직접 비교* 가능 |

> **설계 압력 (R3) 의 적극적 활용:** 만약 위 모듈에서 mock 이 *필요해진다면*, 그것은 모듈 경계나 의존 그래프가 잘못되었다는 신호다.

### 10.2 Doubles 가 **허용되는** 경계

| 경계 | 사용되는 Double 유형 | 사유 |
|---|---|---|
| **시드 공급원** (Generator) | Stub (고정 시드 반환) | 외부 난수원 차단, 결정성 회귀 |
| **시각 공급원** (외곽 모듈) | Fake Clock | 도메인 모듈은 영향 받지 않음을 *명시적으로 검증* |
| **CI 환경 (보조)** | 환경 변수 stub | 도메인 결정성에 환경이 영향 없음을 증명 |
| **테스트 보고 출력** | Spy (호출 횟수 검증) | 메타 테스트 — 결과 자체에는 무관 |

### 10.3 금지 사항

| 금지 | 사유 |
|---|---|
| **Validator 의 mock 화** | Validator 는 *공통 기준* — mock 으로 대체되면 Solver/Generator/Enumerator 의 출력 검증이 *자기참조* 가 된다 (Why #2 ② 위배) |
| **부분 격자의 가짜 PartiallyConsistent 응답 강요** | 분류 결정성(섹션 7.3) 이 깨진다 |

---

## 11. 테스트 명명 규약

### 11.1 선택안

| 후보 | 형식 | 예 |
|---|---|---|
| **(A) `subject_state_expected`** | snake_case, 3토큰 | `validate_emptyGrid_returnsPartiallyConsistent` |
| **(B) Given-When-Then 풀어쓰기** | 자연어형 | `given_emptyGrid_when_validate_then_partiallyConsistent` |
| **(C) Specification (camel/space)** | 산문에 가까움 | `"빈 격자는 부분 일관으로 분류된다"` |

### 11.2 채택안과 사유

> **채택: (A) `subject_state_expected` (snake_case)**

| 사유 | 내용 |
|---|---|
| 도구 비종속성 | 거의 모든 테스트 러너에서 그대로 표시·필터링·정렬 가능 |
| 검색성 | `validate_*`, `solve_*`, `generate_*`, `enumerate_*` 접두로 모듈별 일괄 추출 |
| 추적성 매트릭스와의 정합 | 섹션 13의 Test ID(V-R*, S-R*, G-R*, E-R*, T-I-*, T-VII-* 등)와 1:1 매핑 가능 |
| 자연어 진술과의 분리 | Given/When/Then 의 *세부 진술* 은 코드 본문/주석에, 이름은 *압축된 의도* 만 담아 가독성 유지 |

### 11.3 한국어/영어 혼용 정책

| 영역 | 언어 | 이유 |
|---|---|---|
| **테스트 함수 이름** | 영어 (snake_case) | 도구·CI·로그 호환성, IDE 자동완성 |
| **테스트 본문 내 주석·Given/When/Then 문구** | 한국어 허용 | 의도의 정확한 표현 |
| **본 설계 문서·산출물 문서** | 한국어 | 사고 능력 외부화의 매체 일관성 |
| **공개 도메인 어휘 (`Satisfied` 등)** | 영어 식별자, 한국어 부연 | 단일 공식 명칭 + 학습 친화 |

---

## 12. CI / 실행 정책

### 12.1 테스트 그룹 분리

| 그룹 | 포함 계층 | 실행 빈도 |
|---|---|---|
| **fast** | L1 Unit, L2 Contract | 모든 커밋, 로컬 사전훅 |
| **property** | L3 Property-Based | 모든 PR, 메인 머지 |
| **equivalence** | L4 Equivalence | 모든 PR |
| **integration** | L5 Integration | 모든 PR, 야간 회귀 |
| **determinism** | 결정성 회귀 (불변식 VII 전용) | 야간 회귀 (반복 수 ↑) |

### 12.2 속성 테스트(L3) 최소 시도 횟수 권장값

| 시드 정책 | 권장 시도 수 | 근거 |
|---|---|---|
| **고정 시드 (회귀)** | 100 | 회귀 — *재현성* 이 우선, 표본 수는 보조 |
| **무작위 시드 (PR)** | 500 | 분포 커버리지와 CI 시간의 균형 |
| **확장 시드 (야간)** | 5,000 ~ 10,000 | 희소 위반 발견 — 7,040/16! ≈ \(3.4 \times 10^{-10}\) 의 비대칭(선행 문서 STEP 2 ①)에서 *생성된 격자*가 아닌 *변환·부분 격자* 의 PBT 에 적용 |

### 12.3 결정성 회귀 (불변식 VII) 빈도

| 종류 | 빈도 | 비고 |
|---|---|---|
| **Validator 결정성** (T-VII-기본) | 모든 PR | N=100 |
| **Solver 결정성** (T-VII-1) | 모든 PR | N=100 |
| **Generator 결정성** (T-VII-2) | 모든 PR | N=100 |
| **Enumerator 결정성** (T-VII-3) | 모든 PR | N=100 |
| **장기 결정성 회귀** | 야간 | N=10,000 |

### 12.4 실패 시 정책

| 상황 | 정책 |
|---|---|
| L1/L2 실패 | 머지 차단 (필수 게이트) |
| L3 1건 실패 | 머지 차단 + *반례 시드* 자동 회귀 케이스로 승격 |
| L5 실패 | 머지 차단 + Validator 와 활동 모듈 중 *어느 쪽의 진실이 깨졌는지* 우선 식별 |
| 결정성 실패 | **최우선 차단** — 비결정성의 침입 흔적이므로 즉시 원인 격리 |

---

## 13. 추적성 매트릭스 (Traceability Matrix)

### 13.1 매트릭스

| 불변식 | 모듈 | 테스트 계층 | 테스트 시나리오 ID | STEP5 정의 항목 | 훈련 능력(#) |
|---|---|---|---|---|---|
| **I** 도메인 | Grid | L1 Unit | T-I-1 | STEP5-3 (I), STEP5-2 (a) | #1, #3 |
| **I** 도메인 | Validator | L1 Unit | T-I-2 | STEP5-3 (I) | #2 |
| **I** 도메인 | Validator | L3 Property-Based | T-I-3 | STEP5-3 (I), STEP5-2 (b) | #2 |
| **II** 마법합 | Validator | L1 Unit | T-II-1 | STEP5-3 (II) | #1 |
| **III** 합 균등 | Validator | L1 Unit | T-III-1 | STEP5-3 (III), STEP5-2 (b) | #2, #5 |
| **III** 합 균등 | Validator | L1 Unit | T-III-2 | STEP5-3 (III) | #2 |
| **III** 합 균등 | Validator | L3 Property-Based | T-III-3 | STEP5-3 (III)+(VI) | #2, #6 |
| **IV** 전체합 | Validator | L2 Contract | T-IV-1 | STEP5-3 (IV) | #2 |
| **V** 부분 일관 | Validator | L1 Unit | T-V-1 | STEP5-3 (V), STEP5-2 (c) | #4 |
| **V** 부분 일관 | Validator | L1 Unit | T-V-2 | STEP5-3 (V), STEP5-2 (c) | #4 |
| **V** 부분 일관 | Validator | L1 Unit | T-V-3 | STEP5-3 (V)+(I) | #4 |
| **VI** 대칭 닫힘 | Symmetry, Validator | L4 Equivalence + L5 Integration | T-VI-1 | STEP5-3 (VI) | #6, #7 |
| **VI** 대칭 닫힘 | Symmetry, Validator | L3 + L4 | T-VI-2 | STEP5-3 (VI) | #6 |
| **VII** 판정 결정성 | Solver | L3 + L5 | T-VII-1 | STEP5-3 (VII), STEP5-2 (e) | #8 |
| **VII** 판정 결정성 | Generator | L3 + L5 | T-VII-2 | STEP5-3 (VII) | #8 |
| **VII** 판정 결정성 | Enumerator | L3 + L5 | T-VII-3 | STEP5-3 (VII), STEP5-2 (d) | #5, #8 |
| I+III+IV | Validator | L2 Contract | V-R2 / V-R3 | STEP5-2 (b), (d) | #3 |
| V | Validator | L1 / L2 | V-R1 / V-R4 | STEP5-2 (c) | #4 |
| (형태/술어 분리) | Validator | L2 Contract | V-R5 | STEP5-2 (a), (d) | #3, #7 |
| VII | Validator | L1 (메타) | V-R6 | STEP5-3 (VII) | #8 |
| I+III+IV | Solver | L2 + L5 | S-R1 / S-R3 / S-R4 / S-R6 | STEP5-2 (b) | #5, #7 |
| (입력 분류 일치) | Solver | L2 | S-R2 | STEP5-2 (c) | #4 |
| VII | Solver | L3 | S-R5 | STEP5-3 (VII) | #8 |
| I+III+IV | Generator | L3 + L5 | G-R1 / G-R4 / G-R6 | STEP5-2 (b) | #5 |
| VII | Generator | L3 | G-R2 | STEP5-3 (VII) | #8 |
| (시드 의존) | Generator | L2 | G-R3 | STEP5-2 (e) | #5 |
| (외부 자원 차단) | Generator | L2 (메타) | G-R5 | STEP5-2 (e) | #7 |
| VI (계수) | Enumerator | L4 | E-R1 / E-R2 | STEP5-3 (VI), STEP1 | #6 |
| I+III+IV (출력) | Enumerator | L5 | E-R3 | STEP5-2 (b), (d) | #5 |
| VI (대표원 유일) | Enumerator, Symmetry | L4 | E-R4 | STEP5-3 (VI) | #6 |
| VII (시퀀스) | Enumerator | L3 | E-R5 | STEP5-3 (VII) | #8 |
| VI (orbit 합집합) | Enumerator, Symmetry | L4 + L5 | E-R6 | STEP5-3 (VI) | #6, #7 |
| (분류 전체성/배타성) | Validator | L1 (메타) | T-Class-Total, T-Class-Disjoint | STEP5-2 (c) | #4 |

### 13.2 자체 검증 체크리스트

#### (a) 불변식 I~VII 누락 검사

| 불변식 | 등장 행 수 | 충족 |
|:-:|:-:|:-:|
| I | 3 (+ V-R5, S-R1 계열에서 간접) | ✅ |
| II | 1 | ✅ |
| III | 3 | ✅ |
| IV | 1 (+ S/G/E 의 출력 측 간접) | ✅ |
| V | 3 (+ V-R1, V-R4) | ✅ |
| VI | 5 (T-VI-1/2, E-R1/2/4/6) | ✅ |
| VII | 6 (T-VII-1/2/3, V-R6, S-R5, G-R2, E-R5) | ✅ |

#### (b) 모듈 6종 누락 검사

| 모듈 | 등장 | 충족 |
|---|:-:|:-:|
| Grid | T-I-1 | ✅ |
| Validator | T-I-2/3, T-II-1, T-III-*, T-IV-1, T-V-*, V-R1~R6, T-Class-* | ✅ |
| Solver | S-R1 ~ S-R6, T-VII-1 | ✅ |
| Generator | G-R1 ~ G-R6, T-VII-2 | ✅ |
| Enumerator | E-R1 ~ E-R6, T-VII-3 | ✅ |
| Symmetry | T-VI-1/2, E-R4, E-R6 | ✅ |

#### (c) 훈련 능력 1~8 누락 검사

| # | 능력 | 등장 위치 | 충족 |
|:-:|---|---|:-:|
| 1 | 규칙의 외부화 | T-I-1, T-II-1 | ✅ |
| 2 | 불변식 중심 사고 | T-I-2/3, T-III-*, T-IV-1 | ✅ |
| 3 | 계약 기반 사고 | T-I-1, V-R2/3/5 | ✅ |
| 4 | 상태 분류 사고 | T-V-*, V-R1/R4, S-R2, T-Class-* | ✅ |
| 5 | 검증과 구성의 분리 | T-III-1, T-VII-3, S-R1/3/4/6, G-R1/4/6, G-R3, E-R3 | ✅ |
| 6 | 대칭과 동치 사고 | T-III-3, T-VI-1/2, E-R1/2/4/6 | ✅ |
| 7 | 모듈 분해 사고 | T-VI-1, V-R5, S-R1/3/4/6, G-R5, E-R6 | ✅ |
| 8 | Spec-First 사고 | T-VII-1/2/3, V-R6, S-R5, G-R2, E-R5 | ✅ |

> **자체 검증 결과: 모든 항목(I~VII × 모듈 6종 × 훈련 능력 1~8) 누락 0건.**

---

## 14. 다음 단계 (구현 진입 전 마지막 점검)

### 14.1 게이트 체크리스트

| # | 게이트 | 충족 기준 |
|---|---|---|
| G1 | 모든 모듈의 *공개 계약* 이 섹션 6.1 의사 시그니처 수준으로 합의됨 | 합의 서명(또는 PR 승인) |
| G2 | 모든 불변식 I~VII 에 대해 최소 1개의 Red 테스트 시나리오가 자연어로 외부화됨 | 섹션 5, 13 완료 |
| G3 | 분류(`Satisfied/PartiallyConsistent/Violated`) 의 *전체성·배타성* 메타 테스트가 정의됨 | 섹션 7.3 / T-Class-* |
| G4 | 비결정성의 단일 출처(시드)가 합의됨 | 섹션 9.1 |
| G5 | Validator 의 mock 화 금지 정책이 합의됨 | 섹션 10.3 |
| G6 | 테스트 명명 규약·언어 정책 합의 | 섹션 11 |
| G7 | CI 그룹·실행 빈도 합의 | 섹션 12 |
| G8 | 추적성 매트릭스의 누락 0건 자체 검증 통과 | 섹션 13.2 |

> 위 G1~G8 미충족 항목이 있으면 **구현 진입 금지.**

### 14.2 본 설계로 *막을 수 없는* 위험과 대응

| 위험 | 대응 |
|---|---|
| **본질해 880이라는 수치 자체의 신뢰성** | 외부 출처(이산수학 표준 문헌)와의 *교차 검증* 단계를 별도 회귀 케이스로 보존. 일치 실패 시 *정의 자체* 를 재검토. |
| **부동소수·플랫폼 차이** | 본 도메인은 정수 산술만 사용 → 차단됨. 새로운 의존성 도입 시 *결정성 게이트* 재검토. |
| **OrderingRule 기본값 변경이 사용자에게 미치는 영향** | 기본값을 *공개 계약의 일부* 로 명시하고, 변경 시 *의미적 버전* 상향. |
| **PBT 의 표본 분포 편향** | 시드 다양화 + 야간 확장 시도(섹션 12.2). |
| **대표원 선택 규칙의 미묘한 비결정성** | 섹션 8.2 의 전순서·결정성 요구를 메타 테스트로 강제. |
| **설계 문서·구현·테스트의 *세 진실* 표류** | 추적성 매트릭스(섹션 13)를 *코드 PR 의 필수 갱신 산출물* 로 지정. |

---

## 15. 부록

### 15.1 용어 정리 (선행 문서 용어 + TDD 용어)

| 용어 | 정의 |
|---|---|
| **마방진 (Magic Square)** | n×n 격자에 \(1 \sim n^2\) 을 한 번씩 배치하여 모든 행·열·주대각선 합이 같은 배치 (선행 문서 부록 승계) |
| **마법합 (Magic Constant)** | \(M = n(n^2+1)/2\). n=4 일 때 M=34 (승계) |
| **술어 (Predicate)** | 격자가 마방진인지 참/거짓으로 판정하는 명시적 규칙 (승계) |
| **불변식 (Invariant)** | 모든 유효 상태에서 항상 참인 명제 (승계) |
| **본질적으로 다른 해** | 회전·반사로 변환할 수 없는 서로 다른 마방진. 4×4 에서 880개 (승계) |
| **TDD** | Test-Driven Development. 판정 기준(테스트)을 먼저 작성하고 구현을 뒤따르게 하는 방식 (승계) |
| **Spec-First** | 명세(규칙·계약·불변식)를 구현보다 먼저 진술하는 사고 방식 (승계) |
| **Red / Green / Refactor** | TDD 의 3박자 사이클 — 실패하는 테스트 작성 → 통과시키는 최소 변경 → 깨지 않으면서 정리 |
| **Property-Based Test (PBT)** | 임의 입력에 대해 *불변식이 항상 참* 임을 다수 표본으로 검증하는 테스트 |
| **Equivalence Class** | 같은 *처리 결과* 를 보이는 입력들의 묶음. 테스트 케이스를 *분류 단위* 로 설계할 때의 기본 단위 |
| **Boundary Value** | 동치류의 경계에 위치한 값 — 결함이 가장 자주 출몰하는 지점 |
| **Test Double** | 테스트를 위해 진짜 협력자를 대체하는 가짜 객체의 총칭 (Stub, Mock, Fake, Spy, Dummy) |
| **Contract** | 입력 도메인과 출력 분류, 그리고 *지켜져야 할 사후 조건* 의 명시적 약속 |
| **Pure Function** | 같은 입력에 항상 같은 출력을 내며 부수효과가 없는 함수 (불변식 VII 의 구현 수단) |
| **Referential Transparency** | 어떤 표현식을 그 값으로 치환해도 프로그램 의미가 변하지 않는 성질 |
| **orbit** | 군 작용 아래 한 원소가 도달하는 원소들의 집합. 본 문서에서는 8개 대칭 \(D_4\) 의 작용 |
| **Canonical Representative** | 한 orbit 에서 *전순서적·결정적* 으로 선택되는 유일 대표 원소 |
| **Decision Table** | 입력 조건의 조합을 행으로, 기대 결과를 열로 두어 분류의 *전체성·배타성* 을 시각화하는 도구 |

### 15.2 알고리즘 개념적 참고 (Hard Constraints 에 따라 *언급만*)

본 설계는 알고리즘을 선택하지 않는다. 다만 구현 단계에서 다음 *카테고리* 가 후보가 될 수 있음을 *참고로만* 기록한다.

| 카테고리 | 본 설계와의 접점 |
|---|---|
| 백트래킹 | Solver 의 OrderingRule 가 명시적으로 외부화되어 있다는 점에서 정합 |
| 제약 전파 (Constraint Propagation) | Validator 의 부분 일관성 분류(V)와 정합 |
| 군론적 정규화 | Symmetry / 대표원 선택과 정합 |

> *어느 카테고리를 선택하더라도, 본 문서의 테스트는 변하지 않는다.* 이것이 Spec-First 의 작동 증거다.

---

*본 문서는 구현 코드·알고리즘 선택을 포함하지 않습니다.*
