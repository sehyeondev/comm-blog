---
title: "6G에서 Localization의 위상은 어떻게 달라지고 있는가"
date: 2026-04-04
draft: true
math: true
tags: ["6G", "Localization", "ISAC", "IMT-2030", "3GPP"]
categories: ["Localization", "Trends"]
summary: "5G까지 부가 기능에 가까웠던 Localization이 6G 논의에서는 설계 단계부터 고려되고 있다. 그 배경을 정리하고 개인적인 생각을 덧붙인다."
---

## 5G까지, Localization은 통신의 부가 기능이었다

5G 이전까지 셀룰러 네트워크에서 Localization은 주요 기능이 아니었다.

LTE에서 단말 위치를 추정하려면 OTDOA 같은 방식을 쓸 수 있었지만, 정확도는 수십 미터 수준이었다. GPS가 있는 상황에서 셀룰러로 위치를 잡을 동기가 크지 않았고, 실내에서는 Wi-Fi Fingerprinting이나 BLE Beacon 같은 별도 기술에 의존했다.

5G NR에서 변화가 시작되었다. 3GPP Release 16부터 NR 기반 Positioning이 도입되었고, DL-TDoA, Multi-RTT, AoA 등의 방식이 정의되었다. Release 17에서는 산업용 시나리오를 위해 수십 cm 정확도를 목표로 개선이 이루어졌고, Release 18에서는 AI/ML 기반 Positioning이 Study Item에 포함되었다 [[2]](#ref-2)[[3]](#ref-3).

하지만 이 모든 건 여전히 **"통신 시스템에 Positioning 기능을 추가한다"** 는 프레임이었다고 생각한다. 통신이 주이고, Positioning은 부가 기능.

6G 논의에서는 이 구도가 달라지고 있는 것으로 보인다.

## 6G 논의에서 무엇이 달라지고 있는가

### Sensing이 6G의 새로운 Capability로 명시되었다

ITU-R의 IMT-2030 Framework(Recommendation M.2160, 2023)는 6G의 새로운 Capability 중 하나로 **Sensing**을 명시했다 [[1]](#ref-1). 여기서 Sensing은 환경 인식, 물체 탐지, 그리고 Localization을 포함한다.

이것이 실제 표준과 제품에 어떤 형태로 반영될지는 아직 확정되지 않았다. 하지만 적어도 설계 목표 단계에서 "통신 시스템이 센싱도 한다"는 방향이 공식적으로 잡혔다는 것은 의미가 있다.

관련 논의에서 자주 언급되는 요구 정확도는 아래와 같다 [[5]](#ref-5). 다만 이 수치들은 Use Case별 목표치이지, 확정된 시스템 요구사항은 아니다:

| 응용 시나리오 | 논의되는 요구 정확도 |
|---|---|
| AR/XR | ~1cm |
| 협동 로봇 | ~10cm |
| 제조/물류 | <20cm (수평+수직, <1초 지연) |
| 자율주행 | ~10cm |

### ISAC: 통신과 센싱을 하나의 시스템에서

6G에서 Localization의 위상 변화를 이해하는 데 가장 중요한 키워드가 **ISAC(Integrated Sensing and Communication)** 이라고 생각한다.

기존에는 통신과 센싱(레이더, 측위 등)이 별도 시스템이었다. 각각 다른 주파수, 다른 하드웨어, 다른 파형을 사용했다. ISAC는 **하나의 시스템이 동일한 하드웨어, 스펙트럼, 파형을 공유하면서 통신과 센싱을 동시에 수행**하는 개념이다 [[4]](#ref-4)[[6]](#ref-6).

이게 왜 주목받는가? 두 가지 관점에서 볼 수 있다.

**첫째, 스펙트럼 효율.** 주파수 자원은 유한하다. 통신용 대역 따로, 레이더용 대역 따로 할당하는 건 비효율적이다. 같은 신호로 데이터도 보내고 환경도 인식하면 스펙트럼을 훨씬 효율적으로 쓸 수 있다.

**둘째, 상호 이득의 가능성.** 센싱으로 얻은 위치/환경 정보가 통신 성능을 개선하고, 통신 인프라가 센싱의 커버리지를 확장할 수 있다. 예를 들어 단말의 정밀한 위치를 알면 Beam Management가 효율적으로 동작할 수 있고, Beam Failure를 예측해서 대응하거나 CSI Feedback 오버헤드를 줄이는 것도 가능할 수 있다. 이런 시나리오가 연구 레벨에서 활발히 논의되고 있다.

다만 ISAC가 실제로 어떤 형태로 구현되고 표준화될지는 아직 초기 단계이다. 개념적으로는 매력적이지만, Sensing과 Communication 사이의 Resource Tradeoff, Waveform Design, Interference Management 등 풀어야 할 문제가 많다.

## 관련 기술 동향

### Sub-THz / Terahertz 대역

6G에서 고려되는 Sub-THz(100GHz~300GHz) 및 THz 대역은 Localization 관점에서 두 가지 이점이 있다 [[7]](#ref-7).

**넓은 Bandwidth.** cm급 ToF 정확도를 위해서는 수 GHz 단위의 Bandwidth가 필요하다. Sub-6GHz에서는 확보가 어려운 수치지만, Sub-THz에서는 가능하다.

$$\Delta d = \frac{c}{2 \cdot BW}$$

예를 들어 1cm의 Range Resolution을 얻으려면 약 15GHz Bandwidth가 필요한데, 이는 Sub-THz 이상에서나 현실적인 수치다. 물론 분해능과 실제 정확도는 다르지만, Bandwidth가 정확도의 물리적 상한을 결정한다는 점은 변하지 않는다.

**짧은 파장.** 파장이 짧아지면 같은 물리적 크기의 Antenna Array에 더 많은 소자를 배치할 수 있고, AoA 추정 정확도와 공간 분해능이 올라간다.

다만 Sub-THz 대역은 전파 감쇄가 크고, 기존 반도체 기술로 구현하기 어려운 부분이 많아서 실용화까지는 상당한 시간이 걸릴 것으로 보인다.

### RIS (Reconfigurable Intelligent Surface)

RIS는 전파의 반사 방향을 능동적으로 제어할 수 있는 Metasurface이다. Localization 관점에서 흥미로운 이유는 **NLOS 환경에서의 가능성** 때문이다.

Indoor Localization에서 가장 어려운 문제 중 하나가 NLOS이다. 직접파가 차단되면 ToF 거리 추정에 Bias가 생기고, AoA도 왜곡된다. RIS를 사용하면 직접파가 없는 상황에서 **제어 가능한 반사 경로**를 만들 수 있고, Phase Shift를 알고 있으므로 반사 경로의 기하학적 정보를 활용한 Localization이 가능하다는 것이 연구 커뮤니티의 기대이다.

솔직히 아직 연구 초기 단계이고, 실제 환경에서의 성능 검증이나 실용화까지는 갈 길이 멀다. 하지만 NLOS라는 근본적인 문제에 대한 새로운 접근법이라는 점에서 주목할 만하다고 생각한다.

### AI/ML 기반 Positioning

3GPP Release 18에서 AI/ML 기반 Positioning이 Study Item으로 포함되었다 [[2]](#ref-2)[[3]](#ref-3). 크게 두 가지 방향이 논의되고 있다:

**Direct AI/ML Positioning.** CSI나 Timing 측정값을 Neural Network에 입력해서 직접 위치 좌표를 출력하는 방식. Fingerprinting의 확장이라고 볼 수 있다.

**AI/ML-Assisted Positioning.** 기존 Model-Based Positioning의 특정 단계(NLOS 식별, Timing 추정, Multipath 분리 등)를 AI로 대체하거나 보완하는 방식.

개인적으로는 두 번째 방향이 더 실용적이지 않을까 생각한다. End-to-End Neural Network는 환경이 바뀌면 재학습이 필요하고 결과 해석이 어렵다. 반면 Model-Based 파이프라인의 병목 지점만 AI로 풀면, 물리적 해석 가능성을 유지하면서 성능을 개선할 수 있다. 다만 이건 연구 커뮤니티에서도 의견이 갈리는 부분이고, 환경이 고정된 시나리오에서는 End-to-End가 더 나은 경우도 있을 수 있다.

## 표준화 타임라인

현재까지의 흐름을 정리하면:

| 시점 | 내용 |
|---|---|
| **Rel-16 (2020)** | NR Positioning 도입. DL-TDoA, Multi-RTT, AoA 등 정의 |
| **Rel-17 (2022)** | 산업용 시나리오 타겟, 수십 cm 정확도 목표 |
| **Rel-18 (2024)** | AI/ML Positioning Study Item, Low-Power High-Accuracy Positioning |
| **Rel-20 (2027 목표)** | 6G Study 본격화. ISAC, Sensing 포함 예정 |
| **Rel-21 (2030 전)** | 첫 6G 기술 규격(Normative). IMT-2030 제출 대상 |

ITU 쪽에서는 2029년 초까지 IMT-2030 기술 제안서를 받고, 2030년 중반까지 규격을 확정하는 타임라인이 잡혀 있다 [[8]](#ref-8).

지금(2026년)은 Rel-20 Study가 진행 중인 시점이다. ISAC와 Sensing이 실제 규격에 어떤 형태로 들어갈지는 아직 구체화되는 단계이다.

## Localization을 연구하면서 느끼는 것

개인적으로 흥미로운 건, Localization이라는 분야가 점점 "독립된 연구 주제"에서 "시스템 설계의 일부"로 확장되고 있다는 느낌이 든다는 것이다.

예전에는 Localization 논문이라고 하면 "이 알고리즘이 RMSE를 얼마나 줄였는가"가 핵심이었다. 지금도 그 부분은 중요하지만, 6G 맥락에서는 더 큰 질문이 붙는다. Localization 결과가 Beam Management에 어떤 영향을 주는가? Sensing과 Communication의 Resource를 어떻게 나눌 것인가?

이건 기회이기도 하고 도전이기도 하다. Localization만 잘 해서는 부족하고, 통신 시스템 전체를 이해해야 하는 방향으로 가고 있다는 느낌이다. 그래서 이 블로그에서 Localization 외의 주제도 다루려는 것이기도 하다.

## 정리

6G에서 Localization은 5G 때보다 분명히 위상이 올라가고 있다. ITU-R IMT-2030이 Sensing을 새로운 Capability로 명시했고 [[1]](#ref-1), ISAC라는 개념을 통해 통신과 센싱이 하나의 시스템에서 동작하는 방향이 논의되고 있다 [6].

다만 이것이 "Localization이 6G의 핵심이다"라고 단정할 수 있는 단계는 아니다. 6G의 주된 동력은 여전히 통신 성능(throughput, latency, coverage, energy efficiency)이고, Sensing/Localization은 여러 새로운 Capability 중 하나이다. ISAC도 아직 연구 단계이며, 표준에 어떤 형태로 반영될지는 앞으로의 논의에 달려 있다.

확실한 건, Localization이 "부가 기능"에서 "설계 단계부터 고려되는 기능"으로 격상되고 있다는 흐름이다. 이 변화를 지켜보는 것 자체가 흥미롭고, 앞으로 ISAC의 기술적 구조나 AI/ML Positioning 동향도 다뤄볼 계획이다.

## References


<span id="ref-1">1.</span> ITU-R, "Recommendation ITU-R M.2160-0: Framework and overall objectives of the future development of IMT for 2030 and beyond," Nov. 2023. [[PDF]](https://www.itu.int/dms_pubrec/itu-r/rec/m/R-REC-M.2160-0-202311-I!!PDF-E.pdf)

<span id="ref-2">2.</span> 3GPP, "Release 18 Specifications," 2024. [[Link]](https://www.3gpp.org/specifications-technologies/releases/release-18)

<span id="ref-3">3.</span> Ericsson, "5G Advanced positioning in 3GPP Release 18," Nov. 2024. [[Blog]](https://www.ericsson.com/en/blog/2024/11/5g-advanced-positioning-in-3gpp-release-18)

<span id="ref-4">4.</span> Huawei, "Integrated Sensing and Communication: From Concept to Practice." [[Link]](https://www.huawei.com/en/huaweitech/future-technologies/integrated-sensing-communication-concept-practice)

<span id="ref-5">5.</span> C. De Lima et al., "6G White Paper on Localization and Sensing," arXiv:2006.01779, 2020. [[arXiv]](https://arxiv.org/abs/2006.01779)

<span id="ref-6">6.</span> H. Wymeersch et al., "Integration of Communication with Sensing, Localization, and Computing: The Role of ISAC in 6G," arXiv:2510.04413, 2025. [[arXiv]](https://arxiv.org/abs/2510.04413)

<span id="ref-7">7.</span> T. S. Rappaport et al., "Terahertz Communications and Sensing for 6G and Beyond," arXiv:2307.10321, 2023. [[arXiv]](https://arxiv.org/abs/2307.10321)

<span id="ref-8">8.</span> Ericsson, "6G standardization timeline and technology principles," Mar. 2024. [[Blog]](https://www.ericsson.com/en/blog/2024/3/6g-standardization-timeline-and-technology-principles)
