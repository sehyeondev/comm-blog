---
title: "Indoor Localization 기술 비교: UWB, Wi-Fi, BLE는 각각 어디에 쓰이는가"
date: 2026-04-02
draft: false
math: true
tags: ["UWB", "Wi-Fi RTT", "BLE", "Indoor Localization", "802.15.4z", "802.11mc"]
categories: ["Localization"]
summary: "UWB, Wi-Fi RTT, BLE — 세 기술 모두 실내 위치를 잡을 수 있지만, 적합한 시나리오는 전혀 다르다. 각 기술의 원리와 트레이드오프를 정리한다."
---

## 왜 이 비교가 필요한가

"실내에서 위치를 잡고 싶다"는 요구사항은 간단한데, 막상 기술을 고르려고 하면 복잡해진다. UWB, Wi-Fi, BLE 세 가지 모두 Indoor Localization에 쓸 수 있다. 그런데 정확도, 인프라 비용, 전력 소모가 전부 다르다.

"뭐가 제일 좋나요?"라는 질문에 대한 답은 — 당연하지만 — **상황에 따라 다르다**. 이 글에서는 각 기술이 어떤 원리로 위치를 추정하는지, 그리고 어떤 시나리오에 어떤 기술이 맞는지를 정리해본다.

## Ranging 원리 비교

Indoor Localization의 출발점은 **기준점(Anchor)과 단말 사이의 거리 또는 방향을 추정**하는 것이다. 이 단계를 Ranging이라고 하고, 기술마다 접근 방식이 다르다.

(각 Ranging 방식의 수학적 디테일은 추후 별도 포스팅에서 다룰 예정이다.)

### UWB: 시간으로 거리를 잰다

UWB(IEEE 802.15.4z) [[1]](#ref-1)는 500MHz 이상의 넓은 Bandwidth를 사용한다. 핵심은 **ToF(Time of Flight)** — 전파가 오가는 시간을 측정해서 거리를 계산한다.

$$d = \frac{c \cdot \Delta t}{2}$$

여기서 중요한 건 Bandwidth와 시간 분해능의 관계다. 500MHz Bandwidth에서 시간 분해능은:

$$\Delta t_{\min} = \frac{1}{500 \times 10^6} = 2\text{ns} \quad \rightarrow \quad \text{거리 분해능 } \approx 60\text{cm}$$

"분해능이 60cm인데 어떻게 10cm 정확도가 나오지?"라고 생각할 수 있다. 분해능은 두 Multipath를 구분하는 최소 거리이고, Leading Edge Detection 등의 후처리를 거치면 분해능보다 훨씬 높은 정확도를 달성할 수 있다.

UWB의 또 다른 강점은 **Multipath Robustness**이다 [[8]](#ref-8). Pulse가 짧으니까 Direct Path와 Reflected Path를 시간 영역에서 분리할 수 있다. 실내처럼 반사가 많은 환경에서 이건 큰 이점이다.

실무에서는 ToF 외에 TDoA, AoA도 사용하고, 이들을 조합한 **Hybrid 방식**이 가장 높은 정확도를 보인다.

| 방식 | 원리 | 특징 |
|---|---|---|
| **ToF / TWR** | 왕복 시간 측정 | Anchor 간 동기화 불필요, 양방향 통신 필요 |
| **TDoA** | 다수 Anchor 도달 시간 차이 | Tag 전력 소모 적음, Anchor 간 ns급 동기화 필요 |
| **AoA** | Antenna Array로 도래각 추정 | Anchor 수 절감 가능, Calibration 필요 |

### Wi-Fi RTT: 이미 깔려 있는 인프라를 활용한다

Wi-Fi RTT는 IEEE 802.11mc의 FTM(Fine Timing Measurement) 프로토콜을 사용한다 [[2]](#ref-2). 원리는 UWB의 TWR과 같다 — Round Trip Time을 측정해서 거리를 계산한다.

차이는 Bandwidth에서 나온다. 80MHz Wi-Fi 채널이면:

$$\Delta t_{\min} = \frac{1}{80 \times 10^6} = 12.5\text{ns} \quad \rightarrow \quad \text{거리 분해능 } \approx 3.75\text{m}$$

UWB 대비 시간 분해능이 6배 이상 떨어진다. 실제 정확도는 Multipath 처리와 Timestamp 정밀도 개선으로 **1~2m** 정도.

Wi-Fi 7에서 320MHz 채널이 도입되면 개선될 여지는 있지만, UWB의 500MHz에는 여전히 못 미친다. 그런데 Wi-Fi RTT의 진짜 강점은 정확도가 아니라 **이미 깔려 있는 인프라**다. 전용 Anchor를 새로 설치할 필요 없이, 기존 Wi-Fi AP를 그대로 쓸 수 있다. 이 차이가 실무에서는 꽤 크다.

### BLE: 싸고, 전력을 거의 안 먹는다

BLE Localization은 전통적으로 **RSSI** 기반이다. 수신 신호 세기를 Path Loss Model에 넣어서 거리를 추정한다:

$$RSSI = RSSI_0 - 10n\log_{10}\left(\frac{d}{d_0}\right)$$

솔직히 RSSI 기반 거리 추정은 정확도가 좋지 않다. Shadowing, Multipath Fading에 크게 흔들리고, **수 미터 수준**이 한계다. 그런데 BLE Beacon은 저렴하고 배터리가 수년간 간다. 정밀한 좌표가 필요 없고 "이 사람이 어느 구역에 있는가" 정도만 알면 되는 시나리오에서는 충분하다.

BLE 5.1에서 **AoA/AoD**가 도입되면서 상황이 좀 달라졌다 [[3]](#ref-3). Antenna Array의 Phase Difference로 방향을 추정하는 방식인데, **Sub-meter 정확도**가 가능해졌다. 다만 전용 Locator 하드웨어가 필요해서, RSSI 방식의 저비용 장점이 일부 상쇄된다.

## 한눈에 비교

| 항목 | UWB | Wi-Fi RTT | BLE (RSSI) | BLE 5.1 (AoA) |
|---|---|---|---|---|
| **정확도** | ~10cm | 1~2m | 3~5m | 0.5~1m |
| **Bandwidth** | 500MHz+ | 20~160MHz | 1~2MHz | 1~2MHz |
| **Ranging 원리** | ToF / TDoA / AoA | ToF (FTM) | RSSI | AoA / AoD |
| **전용 인프라** | 필요 (UWB Anchor) | 불필요 (기존 AP) | Beacon (저가) | 전용 Locator |
| **단말 전력** | 중간 | 중간 | 매우 낮음 | 낮음 |
| **표준** | 802.15.4z | 802.11mc | BT 4.0+ | BT 5.1 |
| **Multipath Robustness** | 높음 | 중간 | 낮음 | 중간 |
| **보안 (Anti-Relay)** | STS 기반 | 미지원 | 미지원 | 미지원 |

숫자만 보면 UWB가 압도적으로 좋아 보인다. 하지만 현실에서는 정확도만으로 기술을 선택하지 않는다.

## 그래서 언제 뭘 쓰는가

### cm급이 필요하면: UWB

- **Digital Key** — 차량 잠금 해제에는 cm급 정확도와 Relay Attack 방지가 필수다. CCC 표준이 UWB Secure Ranging을 요구한다 [[5]](#ref-5).
- **Asset Tracking** — 공장, 물류 창고에서 선반 단위로 위치를 구분하려면 10cm급이 필요하다.
- **AR Navigation** — 자연스러운 AR Overlay를 위해서는 사용자 위치를 정밀하게 추적해야 한다.

대신 전용 Anchor 인프라를 깔아야 한다. 넓은 공간이면 비용이 만만치 않다.

### 인프라를 새로 깔 수 없으면: Wi-Fi RTT

- **쇼핑몰, 공항** — 이미 Wi-Fi AP가 빼곡하다. 추가 인프라 없이 1~2m 정확도면 경로 안내에 충분하다.
- **사무실, 병원** — 기존 Wi-Fi 망을 그대로 활용.

802.11mc를 지원하는 AP가 필요하지만, 요즘 나오는 AP들은 대부분 지원한다. Android 9부터 WifiRttManager API도 제공되어서 앱 개발도 어렵지 않다 [[7]](#ref-7).

Wi-Fi에서는 **CSI 기반 Fingerprinting**도 가능하다. Radio Map을 미리 만들어두고 실시간 CSI를 매칭하는 방식인데, 환경이 바뀌면 Radio Map도 갱신해야 한다는 한계가 있다. RTT와 결합하면 서로 보완이 된다.

### 배터리가 수년은 가야 하면: BLE

- **Proximity Detection** — 매장 근처에 오면 알림을 보내는 정도. 정확한 좌표보다 "어느 구역에 있는가"가 중요하다.
- **Item Finding** — SmartTag가 대표적이다. BLE로 대략적 위치를 파악하고, 가까이 가면 UWB로 정밀 안내하는 2단계 구조.
- **Wearable / IoT** — 배터리 수명이 최우선인 기기.

## 실제로는 섞어서 쓴다

제품 레벨에서 단일 기술만 쓰는 경우는 드물다. **Multi-Technology Fusion**이 일반적이다.

**BLE + UWB 2단계.** SmartTag 류가 이 구조다. BLE로 대략적 위치를 잡고, 근거리에서 UWB로 전환해서 정밀 안내. 전력도 아끼고 정밀도도 챙기는 설계.

**Wi-Fi + BLE Fingerprinting.** Wi-Fi RTT로 거리 기반 위치를 잡고, BLE RSSI Fingerprint로 보정.

**IMU + Wireless.** PDR(Pedestrian Dead Reckoning)로 단기 궤적을 추정하면서, 무선 Localization으로 Drift를 주기적으로 보정하는 Sensor Fusion.

이 융합의 핵심은 **각 기술의 추정치를 신뢰도에 따라 가중해서 결합**하는 것이다. 가까운 UWB Anchor의 추정치에는 높은 가중치를, 먼 BLE Beacon의 추정치에는 낮은 가중치를. 이를 위해 Covariance 기반 Weighted Fusion이나 Kalman Filter가 쓰인다.

## 앞으로 어떻게 달라질까

**UWB** — IEEE 802.15.4ab에서 차세대 표준이 논의 중이다. 더 높은 Data Rate, 개선된 Ranging 정확도, 저전력 모드가 목표. FiRa [[5]](#ref-5)와 CCC를 중심으로 Ecosystem이 확장되고 있고, 스마트폰 탑재율도 계속 올라가고 있다.

**Wi-Fi** — Wi-Fi 7(802.11be)의 320MHz Bandwidth는 RTT 정확도 개선 여지를 준다. 더 흥미로운 건 802.11bf의 **Wi-Fi Sensing** 표준 [[6]](#ref-6). CSI 기반 Presence Detection, Gesture Recognition이 표준화되면 Localization과 Sensing의 경계가 흐려진다.

**BLE** — Bluetooth 6.0에서 **Channel Sounding**이 도입되었다 [[4]](#ref-4). Phase-Based Ranging으로 cm급 거리 추정이 BLE에서도 가능해지는 것. 이게 본격화되면 UWB의 영역이었던 정밀 Localization에 BLE가 진입하게 된다. 개인적으로 가장 주목하고 있는 변화다.

## 정리

UWB, Wi-Fi, BLE는 경쟁 관계라기보다 **상호 보완적**이다. 정밀도가 필요하면 UWB, 인프라 효율이 중요하면 Wi-Fi RTT, 전력과 비용이 우선이면 BLE. 실제 시스템에서는 이걸 조합해서 쓴다.

각 기술의 표준이 빠르게 진화하고 있어서, 기술 간 경계는 점점 더 모호해질 것이다. 특히 BLE Channel Sounding은 기존의 기술 선택 기준을 꽤 흔들 수 있는 변화라고 생각한다.

## References


<span id="ref-1">1.</span> IEEE 802.15.4z-2020, "IEEE Standard for Low-Rate Wireless Networks — Amendment: Enhanced Ultra Wideband (UWB) Physical Layers and Associated Ranging Techniques," 2020.

<span id="ref-2">2.</span> IEEE 802.11mc (802.11-2016), "IEEE Standard for Information Technology — Fine Timing Measurement (FTM)."

<span id="ref-3">3.</span> Bluetooth SIG, "Bluetooth Core Specification v5.1 — Direction Finding," 2019. [[Link]](https://www.bluetooth.com/specifications/specs/core-specification-5-1/)

<span id="ref-4">4.</span> Bluetooth SIG, "Bluetooth Core Specification v6.0 — Channel Sounding," 2024. [[Link]](https://www.bluetooth.com/specifications/specs/core-specification-6-0/)

<span id="ref-5">5.</span> FiRa Consortium, "UWB Technology Overview." [[Link]](https://www.firaconsortium.org/discover-uwb/uwb-technology-overview)

<span id="ref-6">6.</span> IEEE 802.11bf, "WLAN Sensing (Wi-Fi Sensing)," Task Group bf. [[Link]](https://www.ieee802.org/11/Reports/tgbf_update.htm)

<span id="ref-7">7.</span> Google, "Wi-Fi RTT (IEEE 802.11mc) — Android Developers." [[Link]](https://developer.android.com/develop/connectivity/wifi/wifi-rtt)

<span id="ref-8">8.</span> S. Gezici et al., "Localization via Ultra-Wideband Radios: A Look at Positioning Aspects for Future Sensor Networks," *IEEE Signal Processing Magazine*, vol. 22, no. 4, pp. 70–84, Jul. 2005.
