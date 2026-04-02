---
title: "왜 6G에서 Localization이 핵심이 되었는가"
date: 2026-04-02T12:00:00
draft: false
math: true
tags: ["6G", "Localization", "ISAC", "IMT-2030", "3GPP"]
categories: ["Localization", "트렌드"]
summary: "5G까지 부가 기능이었던 Localization이 6G에서는 시스템 설계의 핵심 축이 되고 있다. 그 배경과 기술적 변화를 정리한다."
---

## 5G까지, Localization은 "있으면 좋은 것"이었다

솔직히 말해서, 5G 이전까지 셀룰러 네트워크에서 Localization은 핵심 기능이 아니었다.

LTE에서 단말 위치를 추정하려면 OTDOA(Observed Time Difference of Arrival) 같은 방식을 쓸 수 있었지만, 정확도는 수십 미터 수준. GPS가 있는데 굳이 셀룰러로 위치를 잡을 이유가 별로 없었다. 실내에서는 GPS가 안 되니까 Wi-Fi Fingerprinting이나 BLE Beacon 같은 별도 기술을 갖다 붙였다. 셀룰러 네트워크 자체가 Localization을 책임지는 구조는 아니었다.

5G NR에서 상황이 조금 바뀌기 시작했다. 3GPP Release 16부터 NR 기반 Positioning이 본격적으로 도입되었고, DL-TDoA, UL-TDoA, Multi-RTT, AoA 같은 방식이 정의되었다. Release 17에서는 산업용 시나리오를 위해 수십 cm 정확도를 목표로 개선이 이루어졌고, Release 18에서는 AI/ML 기반 Positioning이 처음으로 Study Item에 포함되었다.

하지만 이 모든 건 여전히 **"통신 시스템에 Positioning 기능을 추가한다"** 는 프레임이었다. 통신이 주(主)이고, Positioning은 부가 기능.

6G에서는 이 구도가 근본적으로 달라진다.

## 6G에서 무엇이 달라지는가

### Localization이 시스템 요구사항이 되었다

ITU-R의 IMT-2030 Framework(Recommendation M.2160)는 6G의 새로운 Capability 중 하나로 **Sensing**을 명시했다. 여기서 Sensing은 환경 인식, 물체 탐지, 그리고 **Localization**을 포함한다. 통신 시스템이 "통신도 하고, 위치도 잡고, 주변 환경도 인식한다"는 것이 6G의 설계 목표에 처음부터 들어가 있다.

구체적인 수치도 나오고 있다:

| 응용 시나리오 | 요구 정확도 |
|---|---|
| AR/XR | ~1cm |
| 협동 로봇 | ~10cm |
| 제조/물류 | <20cm (수평+수직, <1초 지연) |
| 자율주행 | ~10cm |

**cm급 정확도**가 시스템 레벨에서 요구되고 있다. 이건 "있으면 좋은 것"이 아니라, 이 정확도가 안 나오면 서비스 자체가 성립하지 않는다는 뜻이다.

### ISAC: 통신과 센싱의 경계가 사라진다

6G에서 Localization의 위상이 달라진 가장 큰 기술적 배경은 **ISAC(Integrated Sensing and Communication)** 이다.

기존에는 통신과 센싱(레이더, 측위 등)이 별도의 시스템이었다. 각각 다른 주파수, 다른 하드웨어, 다른 파형을 사용했다. ISAC는 **하나의 시스템이 동일한 하드웨어, 스펙트럼, 파형을 공유하면서 통신과 센싱을 동시에 수행**하는 개념이다.

이게 왜 중요한가? 두 가지 관점에서 보면 된다.

**첫째, 스펙트럼 효율.** 주파수 자원은 유한하다. 통신용 대역 따로, 레이더용 대역 따로 할당하는 건 비효율적이다. 같은 신호로 데이터도 보내고 환경도 인식하면 스펙트럼을 훨씬 효율적으로 쓸 수 있다.

**둘째, 상호 이득.** 센싱으로 얻은 위치/환경 정보가 통신 성능을 개선하고, 통신 인프라가 센싱의 커버리지를 확장한다. 예를 들어, 단말의 정밀한 위치를 알면 Beam Management가 훨씬 효율적으로 동작한다. Beam Failure가 발생하기 전에 예측해서 대응할 수 있다. CSI Feedback 오버헤드도 줄일 수 있다. Localization이 통신 시스템의 성능 향상에 직접 기여하는 구조가 되는 것이다.

## 이것을 가능하게 하는 기술들

### Sub-THz / Terahertz 대역

6G에서 고려되는 Sub-THz (100GHz~300GHz) 및 THz 대역은 Localization에 두 가지 이점을 준다.

**넓은 Bandwidth.** cm급 ToF 정확도를 달성하려면 수 GHz 단위의 Bandwidth가 필요하다. 1cm 정확도를 Delay Domain에서 달성하려면 2~4GHz Bandwidth가 요구된다. Sub-6GHz에서는 불가능한 수치지만, Sub-THz에서는 자연스럽게 확보된다.

$$\Delta d = \frac{c}{2 \cdot BW} \quad \Rightarrow \quad BW = \frac{c}{2 \cdot 0.01} = 15\text{GHz}$$

물론 이론적 분해능과 실제 정확도는 다르다. 하지만 Bandwidth가 근본적인 물리적 한계를 결정한다는 점에서, 넓은 대역은 Localization 정확도의 상한을 끌어올린다.

**짧은 파장.** 파장이 짧아지면 같은 물리적 크기의 Antenna Array에 더 많은 소자를 배치할 수 있다. 안테나 수가 많아지면 AoA 추정 정확도가 향상되고, 공간 분해능도 올라간다. Massive MIMO와 결합하면 매우 정밀한 Angular 추정이 가능해진다.

### RIS (Reconfigurable Intelligent Surface)

RIS는 전파의 반사 방향을 능동적으로 제어할 수 있는 메타서피스이다. Localization 관점에서 RIS가 흥미로운 이유는 **NLOS 환경에서의 가능성** 때문이다.

Indoor Localization의 가장 큰 적은 NLOS(Non-Line-of-Sight)이다. 직접파가 차단되면 ToF 기반 거리 추정에 Bias가 발생하고, AoA도 왜곡된다. RIS를 사용하면 직접파가 없는 상황에서도 **제어 가능한 반사 경로**를 만들어낼 수 있다. RIS의 Phase Shift를 알고 있으므로, 반사 경로의 기하학적 정보를 활용한 Localization이 가능해진다.

아직 연구 초기 단계이고 실용화까지는 갈 길이 멀지만, 개념적으로는 Localization의 가장 근본적인 문제(NLOS)에 대한 새로운 접근법이라 주목할 만하다.

### AI/ML 기반 Positioning

3GPP Release 18에서 AI/ML 기반 Positioning이 Study Item으로 포함된 것은 의미 있는 변화다. 두 가지 방향이 논의되고 있다:

**Direct AI/ML Positioning.** CSI나 Timing 측정값을 Neural Network에 입력해서 직접 위치 좌표를 출력하는 방식. Fingerprinting의 확장이라고 볼 수 있다.

**AI/ML-Assisted Positioning.** 기존 Model-Based Positioning의 특정 단계(예: NLOS 식별, Timing 추정, Multipath 분리)를 AI로 대체하거나 보완하는 방식.

개인적으로는 두 번째 방향이 더 실용적이라고 생각한다. End-to-End로 Neural Network에 맡기는 건 환경이 바뀌면 재학습이 필요하고, 왜 그런 결과가 나왔는지 해석이 어렵다. 반면 Model-Based 파이프라인의 병목 지점만 AI로 풀면, 물리적 해석 가능성을 유지하면서 성능을 개선할 수 있다.

물론 이건 아직 연구 커뮤니티에서도 의견이 갈리는 부분이다. 데이터가 충분하고 환경이 고정된 시나리오에서는 End-to-End가 더 나은 경우도 분명 있다.

## 표준화는 어디까지 왔는가

정리하면 이런 흐름이다:

| 시점 | 내용 |
|---|---|
| **Rel-16 (2020)** | NR Positioning 도입. DL-TDoA, Multi-RTT, AoA 등 정의 |
| **Rel-17 (2022)** | 산업용 시나리오 타겟, 수십 cm 정확도 목표 |
| **Rel-18 (2024)** | AI/ML Positioning Study Item, Low-Power High-Accuracy Positioning |
| **Rel-20 (2027 목표)** | 6G Study 본격화. ISAC, Sensing 포함 |
| **Rel-21 (2030 전)** | 첫 6G 기술 규격(Normative). IMT-2030 제출 대상 |

ITU 쪽에서는 2029년 초까지 IMT-2030 기술 제안서를 받고, 2030년 중반까지 규격을 확정하는 타임라인이 잡혀 있다.

지금(2026년)은 Rel-20 Study가 진행 중인 시점이다. ISAC와 Sensing이 어떤 형태로 규격에 반영될지가 구체화되고 있는 단계.

## Localization 연구자로서 느끼는 것

Localization을 연구하면서 흥미로운 건, 이 분야가 점점 "독립된 연구 주제"에서 "시스템 설계의 일부"로 바뀌고 있다는 것이다.

예전에는 Localization 논문이라고 하면 "이 알고리즘이 RMSE를 얼마나 줄였는가"가 핵심이었다. 지금도 물론 그 부분은 중요하지만, 6G 맥락에서는 더 큰 질문이 생긴다. Localization 결과가 Beam Management에 어떤 영향을 주는가? Sensing과 Communication의 Resource Allocation을 어떻게 최적화하는가? 이런 시스템 레벨의 문제가 Localization과 엮이기 시작한다.

이건 연구자 입장에서 보면 기회이기도 하고 도전이기도 하다. Localization만 잘 해서는 부족하고, 통신 시스템 전체를 이해해야 하는 방향으로 가고 있다. 그래서 이 블로그에서 Localization 외의 주제도 다루려는 것이기도 하다.

## 정리

6G에서 Localization은 더 이상 부가 기능이 아니다. ITU-R IMT-2030은 Sensing을 6G의 새로운 Capability로 명시했고, ISAC는 통신과 센싱의 경계를 허물고 있다. cm급 정확도가 시스템 요구사항이 되면서, Sub-THz 대역의 넓은 Bandwidth, RIS를 통한 NLOS 극복, AI/ML 기반 Positioning이 이를 뒷받침하는 핵심 기술로 부상하고 있다.

3GPP Rel-20에서 6G Study가 본격화되고 있는 지금이, 이 흐름을 파악하기에 좋은 시점이라고 생각한다. 앞으로 ISAC의 구체적인 기술 구조나, AI/ML Positioning의 최신 연구 동향도 개별 포스팅으로 다뤄볼 계획이다.
