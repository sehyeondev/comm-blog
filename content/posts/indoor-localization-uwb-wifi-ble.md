---
title: "Indoor Localization 기술 비교: UWB, Wi-Fi, BLE는 각각 어디에 쓰이는가"
date: 2026-04-02
draft: false
math: true
tags: ["UWB", "Wi-Fi RTT", "BLE", "Indoor Localization", "802.15.4z", "802.11mc"]
categories: ["Localization"]
summary: "UWB, Wi-Fi RTT, BLE 기반 Indoor Localization의 원리, 정확도, 인프라 요건을 비교하고 각 기술이 적합한 응용 시나리오를 분석합니다."
---

## 들어가며

Indoor Localization은 GPS가 동작하지 않는 실내 환경에서 단말의 위치를 추정하는 기술이다. 스마트폰, IoT 기기, 로봇 등 실내에서 위치 정보가 필요한 응용이 늘어나면서, UWB, Wi-Fi, BLE 세 가지 무선 기술이 각각의 강점을 가지고 시장에서 경쟁하고 있다.

이 글에서는 세 기술의 Localization 원리, 정확도, 인프라 요건을 비교하고, 어떤 시나리오에 어떤 기술이 적합한지를 분석한다.

## Ranging 원리 비교

Indoor Localization의 핵심은 **기준점(Anchor)과 단말 사이의 거리 또는 방향을 추정하는 것**이다. 기술마다 이 과정의 원리가 다르다.

### UWB: Time of Flight (ToF)

UWB는 IEEE 802.15.4z 표준에 기반하며, 500MHz 이상의 넓은 Bandwidth를 사용한다. Ranging의 핵심은 **ToF(Time of Flight)** 측정이다.

Two-Way Ranging(TWR) 프로토콜에서 Anchor와 Tag가 패킷을 주고받으며 Round Trip Time을 측정하고, 이를 거리로 환산한다:

$$d = \frac{c \cdot \Delta t}{2}$$

시간 분해능은 Bandwidth에 의존한다. 500MHz Bandwidth에서:

$$\Delta t_{\min} = \frac{1}{BW} = \frac{1}{500 \times 10^6} = 2\text{ns}$$

이는 거리 분해능 약 60cm에 해당한다. 실제로는 Leading Edge Detection 등의 기법으로 **10cm 이하의 정밀도**를 달성한다.

UWB의 또 다른 강점은 Multipath Robustness이다. 짧은 Pulse 덕분에 Direct Path와 Reflected Path를 시간 영역에서 분리할 수 있어, NLOS 환경에서도 First Path를 검출하면 정확한 거리 추정이 가능하다.

**Ranging 방식 세분화:**

| 방식 | 원리 | 특징 |
|---|---|---|
| **ToF / TWR** | 왕복 시간 측정 | Anchor 간 동기화 불필요, 양방향 통신 필요 |
| **TDoA** | 다수 Anchor 도달 시간 차이 | Tag 전력 소모 적음, Anchor 간 ns급 동기화 필요 |
| **AoA** | Antenna Array로 도래각 추정 | Anchor 수 절감 가능, Calibration 필요 |

실무에서는 ToF/TDoA로 거리를, AoA로 방향을 동시에 추정하는 **Hybrid 방식**이 가장 높은 정확도를 보인다.

### Wi-Fi RTT: Fine Timing Measurement

Wi-Fi RTT는 IEEE 802.11mc에 정의된 FTM(Fine Timing Measurement) 프로토콜을 사용한다. 원리는 UWB의 TWR과 동일하게 Round Trip Time 기반이다.

차이는 Bandwidth에서 나타난다. 80MHz Wi-Fi 채널에서:

$$\Delta t_{\min} = \frac{1}{80 \times 10^6} = 12.5\text{ns} \approx 3.75\text{m}$$

UWB 대비 시간 분해능이 6배 이상 낮다. 실제로는 Multipath와 Timestamp 정밀도 개선으로 **1~2m 정확도**를 달성한다.

Wi-Fi 6E/7에서 160MHz, 320MHz 채널이 도입되면 분해능이 개선될 여지가 있지만, UWB의 500MHz에는 여전히 미치지 못한다.

### BLE: RSSI 및 AoA/AoD

BLE(Bluetooth Low Energy) Localization은 전통적으로 **RSSI(Received Signal Strength Indicator)** 기반이다. 수신 신호 세기를 Path Loss Model에 대입하여 거리를 추정한다:

$$RSSI = RSSI_0 - 10n\log_{10}\left(\frac{d}{d_0}\right)$$

여기서 $n$은 Path Loss Exponent, $RSSI_0$는 기준 거리 $d_0$에서의 수신 세기이다.

RSSI는 Shadowing, Multipath Fading에 크게 영향받아 **정확도가 수 미터 수준**으로 제한된다.

BLE 5.1에서 도입된 **AoA(Angle of Arrival) / AoD(Angle of Departure)** 는 이를 개선한다. Antenna Array에서 Phase Difference를 측정하여 신호의 도래각을 추정하며, **Sub-meter 정확도**가 가능해졌다. 다만 Array Calibration과 Multipath에 의한 Angular Spread가 여전히 과제이다.

## 정확도 및 인프라 비교

| 항목 | UWB | Wi-Fi RTT | BLE (RSSI) | BLE 5.1 (AoA) |
|---|---|---|---|---|
| **정확도** | ~10cm | 1~2m | 3~5m | 0.5~1m |
| **Bandwidth** | 500MHz+ | 20~160MHz | 1~2MHz | 1~2MHz |
| **Ranging 원리** | ToF / TDoA / AoA | ToF (FTM) | RSSI | AoA / AoD |
| **전용 인프라** | 필요 (UWB Anchor) | 불필요 (기존 AP 활용) | Beacon 설치 (저가) | 전용 Locator 필요 |
| **단말 전력 소모** | 중간 | 중간 | 매우 낮음 | 낮음 |
| **표준** | IEEE 802.15.4z | IEEE 802.11mc | Bluetooth 4.0+ | Bluetooth 5.1 |
| **Multipath Robustness** | 높음 | 중간 | 낮음 | 중간 |
| **보안 (Anti-Relay)** | STS 기반 Secure Ranging | 미지원 | 미지원 | 미지원 |

## 응용 시나리오별 기술 선택

### 정밀 Localization이 필수인 경우: UWB

- **Digital Key**: 차량 잠금 해제 시 cm급 정확도와 Relay Attack 방지가 필수. CCC(Car Connectivity Consortium) 표준이 UWB Secure Ranging을 요구한다.
- **Asset Tracking**: 공장, 물류 창고에서 고가 자산의 실시간 위치 추적. 10cm급 정확도로 선반 단위 위치 구분이 가능하다.
- **AR Navigation**: 실내 AR 내비게이션에서 사용자 위치와 방향을 정밀하게 추적해야 자연스러운 AR Overlay가 가능하다.

UWB의 단점은 **전용 Anchor 인프라 설치 비용**이다. 대규모 공간에서는 Anchor 수가 많아지고, 아직 Wi-Fi만큼 보편적이지 않다.

### 기존 인프라 활용이 핵심인 경우: Wi-Fi RTT

- **대형 쇼핑몰 / 공항**: 이미 설치된 Wi-Fi AP를 활용하여 추가 인프라 없이 1~2m 정확도의 Localization을 제공한다.
- **Enterprise Indoor Navigation**: 사무실, 병원 등에서 기존 Wi-Fi 망을 그대로 활용하여 경로 안내 서비스를 구현한다.

802.11mc를 지원하는 AP가 필요하지만, 최신 AP들은 대부분 지원한다. Android 9부터 WifiRttManager API를 제공하여 앱 개발도 용이하다.

추가로 Wi-Fi에서는 **CSI(Channel State Information) 기반 Fingerprinting**도 가능하다. 사전에 Radio Map을 구축하고, 실시간 CSI를 매칭하여 위치를 추정하는 방식이다. 환경 변화에 취약하다는 한계가 있지만, RTT와 결합하면 정확도를 보완할 수 있다.

### 저전력 / 저비용이 핵심인 경우: BLE

- **Proximity Detection**: 매장 근접 알림, Beacon 기반 마케팅. 정밀한 좌표보다 "어떤 구역에 있는가"가 중요한 시나리오.
- **물건 찾기(Item Finding)**: SmartTag 같은 Tracker에서 BLE로 대략적 위치를 확인하고, 근거리에서 UWB로 정밀 안내하는 2단계 방식이 실제 제품에 적용되어 있다.
- **Wearable / IoT**: 배터리 수명이 최우선인 기기에서 BLE Beacon 기반 Zone-level Localization을 사용한다.

BLE 5.1 AoA/AoD는 정확도를 Sub-meter로 끌어올리지만, 전용 Locator 하드웨어가 필요하여 RSSI 방식의 저비용 장점이 일부 상쇄된다.

## 기술 융합: Multi-Technology Localization

실제 제품에서는 단일 기술이 아닌 **Multi-Technology 융합**이 일반적이다.

대표적인 사례:

1. **BLE + UWB 2단계**: BLE로 대략적 위치를 파악한 후, 근거리에서 UWB로 정밀 Localization. SmartTag 류의 Item Finder가 이 구조를 사용한다.

2. **Wi-Fi + BLE Fingerprinting**: Wi-Fi RTT로 거리를 추정하고, BLE RSSI Fingerprint로 보정하여 정확도를 개선한다.

3. **IMU + Wireless**: PDR(Pedestrian Dead Reckoning)로 단기 궤적을 추정하고, 무선 기반 Localization으로 Drift를 주기적으로 보정하는 Sensor Fusion 구조.

융합의 핵심은 **각 기술의 추정치를 신뢰도에 따라 적절히 가중하여 결합**하는 것이다. 가까운 UWB Anchor의 추정치에는 높은 가중치를, 먼 BLE Beacon의 추정치에는 낮은 가중치를 부여해야 최적의 결과를 얻는다. 이를 위해 Covariance 기반의 Weighted Fusion이나 Kalman Filter가 널리 사용된다.

## 향후 전망

### UWB
IEEE 802.15.4ab에서 차세대 UWB 표준이 논의 중이다. 더 높은 Data Rate, 개선된 Ranging 정확도, 그리고 저전력 모드가 목표이다. FiRa Consortium과 CCC를 중심으로 Ecosystem이 확장되고 있으며, 스마트폰 탑재율이 지속적으로 증가하고 있다.

### Wi-Fi
Wi-Fi 7(802.11be)의 320MHz Bandwidth는 RTT 정확도를 개선할 여지를 제공한다. 또한 802.11bf에서 **Wi-Fi Sensing** 표준이 개발 중이며, CSI 기반의 Presence Detection, Gesture Recognition 등이 표준화되면 Localization과 Sensing의 경계가 흐려질 전망이다.

### BLE
Bluetooth 6.0에서 Channel Sounding이 도입되었다. Phase-Based Ranging으로 **cm급 거리 추정**이 BLE에서도 가능해지며, UWB의 영역이었던 정밀 Localization에 BLE가 진입할 수 있는 기반이 마련되었다.

## 마치며

UWB, Wi-Fi, BLE는 경쟁 기술이라기보다 **상호 보완적인 기술**이다. 정밀도가 필요하면 UWB, 인프라 효율이 중요하면 Wi-Fi RTT, 전력과 비용이 우선이면 BLE가 적합하다. 실제 시스템에서는 이들을 조합한 Multi-Technology Fusion이 최적의 Indoor Localization 성능을 제공한다.

각 기술의 표준이 빠르게 진화하고 있어, 앞으로 기술 간 경계는 더 모호해질 것이다. 특히 BLE Channel Sounding과 Wi-Fi Sensing의 등장은 기존의 기술 선택 기준을 재정의할 가능성이 있다. 이러한 변화를 지속적으로 추적하는 것이 중요하다.
