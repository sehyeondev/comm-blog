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

"뭐가 제일 좋나요?"라는 질문에 대한 답은 당연하지만 **상황에 따라 다르다**. 이 글에서는 각 기술이 어떤 원리로 위치를 추정하는지, 그리고 어떤 시나리오에 어떤 기술이 맞는지를 정리해본다.

## Ranging 원리 비교

Indoor Localization의 출발점은 **기준점(Anchor)과 단말 사이의 거리 또는 방향을 추정**하는 것이다. 이 단계를 Ranging이라고 하고, 기술마다 접근 방식이 다르다.

(각 Ranging 방식의 수학적 디테일은 [Ranging Fundamentals](/comm-blog/posts/ranging-fundamentals/)에서 다룬다.)

### UWB: 시간 기반 거리 측정

UWB(Ultra-Wideband, IEEE 802.15.4z) [[1]](#ref-1)는 500MHz 이상의 넓은 Bandwidth를 사용한다. 핵심은 **ToF(Time of Flight)** — 전파가 오가는 시간을 측정해서 거리를 계산한다.

$$d = \frac{c \cdot \Delta t}{2}$$

여기서 중요한 건 Bandwidth와 Time Resolution의 관계다. 500MHz Bandwidth에서 Time Resolution은:

$$\Delta t_{\min} = \frac{1}{500 \times 10^6} = 2\text{ns}$$

이 Time Resolution에 대응하는 Range Resolution은 편도 전파 거리 기준으로 계산한다. 왕복 시간을 2로 나누는 Ranging 수식과는 다르게, Resolution은 두 경로(Direct Path와 Reflected Path)를 시간 영역에서 구분할 수 있는 최소 거리를 의미하기 때문이다.

$$\text{Range Resolution} = c \cdot \Delta t_{\min} = 3 \times 10^8 \times 2 \times 10^{-9} = 0.6\text{m}$$

"Resolution이 60cm인데 어떻게 10cm 정확도가 나오지?"라고 생각할 수 있다. Resolution은 두 Multipath를 구분하는 최소 거리이고, Leading Edge Detection 등의 후처리를 거치면 Resolution보다 훨씬 높은 정확도를 달성할 수 있다. Leading Edge Detection은 수신 신호에서 에너지가 처음으로 Threshold를 넘는 지점을 찾아 Direct Path의 도착 시간을 추정하는 기법이다 [[2]](#ref-2).

UWB의 또 다른 강점은 **Multipath Robustness**이다 [[3]](#ref-3). Pulse가 짧으니까 Direct Path와 Reflected Path를 시간 영역에서 분리할 수 있다. 실내처럼 반사가 많은 환경에서 이건 큰 이점이다.

실제 시스템에서는 ToF 외에 TDoA, AoA도 사용하고, 이들을 조합한 **Hybrid 방식**이 가장 높은 정확도를 보인다.

| 방식 | 원리 | 특징 |
|---|---|---|
| **ToF / TWR (Two-Way Ranging)** | 왕복 시간 측정 | Anchor 간 동기화 불필요, 양방향 통신 필요 |
| **TDoA (Time Difference of Arrival)** | 다수 Anchor 도달 시간 차이 | Tag 전력 소모 적음, Anchor 간 ns급 동기화 필요 |
| **AoA (Angle of Arrival)** | Antenna Array로 신호 도달 각도 추정 | Anchor 수 절감 가능, Calibration 필요 |

### Wi-Fi RTT: 기존 인프라 활용

Wi-Fi RTT(Round-Trip Time)는 IEEE 802.11mc의 FTM(Fine Timing Measurement) 프로토콜을 사용한다 [[4]](#ref-4). 원리는 UWB의 TWR과 같다 — 신호의 왕복 시간을 측정해서 거리를 계산한다.

차이는 Bandwidth에서 나온다. 80MHz Wi-Fi 채널이면:

$$\Delta t_{\min} = \frac{1}{80 \times 10^6} = 12.5\text{ns} \quad \rightarrow \quad \text{Range Resolution } \approx 3.75\text{m}$$

UWB 대비 Time Resolution이 6배 이상 떨어진다. 실제 정확도는 Multipath 처리와 Timestamp 정밀도 개선으로 **1~2m** 정도라고 알려져 있다 [[8]](#ref-8).

Wi-Fi 7에서 320MHz 채널이 도입되면 개선될 여지는 있지만, UWB의 500MHz에는 여전히 못 미친다. 그런데 Wi-Fi RTT의 진짜 강점은 정확도가 아니라 **이미 깔려 있는 인프라**다. 전용 Anchor를 새로 설치할 필요 없이, 기존 Wi-Fi AP(Access Point)를 그대로 쓸 수 있다. 이 차이는 도입 비용 면에서 상당히 크다.

### BLE: 저비용, 초저전력

BLE(Bluetooth Low Energy) Localization은 전통적으로 **RSSI(Received Signal Strength Indicator)** 기반이다. 수신 신호 세기를 Path Loss Model에 넣어서 거리를 추정한다:

$$RSSI = RSSI_0 - 10n\log_{10}\left(\frac{d}{d_0}\right)$$

솔직히 RSSI 기반 거리 추정은 정확도가 좋지 않다. Shadowing, Multipath Fading에 크게 흔들리고, **수 미터 수준**이 한계다. 그런데 BLE Beacon은 저렴하고 배터리가 수년간 간다. 정밀한 좌표가 필요 없고 "이 사람이 어느 구역에 있는가" 정도만 알면 되는 시나리오에서는 충분하다.

BLE 5.1에서는 **AoA(Angle of Arrival) / AoD(Angle of Departure)** 가 도입되면서 상황이 좀 달라졌다 [[5]](#ref-5). Antenna Array의 Phase Difference로 신호의 도달 각도(또는 출발 각도)를 추정하는 방식인데, **Sub-meter 정확도**가 가능해졌다 [[6]](#ref-6). 다만 전용 Locator 하드웨어가 필요해서, RSSI 방식의 저비용 장점이 일부 상쇄된다. 개인적으로 궁금해서 찾아봤는데, 일반 BLE Beacon이 개당 10–25 USD 수준인 반면 [[11]](#ref-11), Minew의 BLE 5.1 AoA Kit은 Locator 4대 포함 659 USD에 판매되고 있다(2026년 4월 기준) [[12]](#ref-12). Antenna Array가 들어가는 만큼 비용 차이가 상당하다.

## 비교 테이블

| 항목 | UWB | Wi-Fi RTT | BLE (RSSI) | BLE 5.1 (AoA) |
|---|---|---|---|---|
| **정확도** | ~10cm | 1~2m | 3~5m | 0.5~1m |
| **Bandwidth** | 500MHz+ | 20~160MHz | 1~2MHz | 1~2MHz |
| **Ranging 원리** | ToF / TDoA / AoA | ToF (FTM) | RSSI | AoA / AoD |
| **전용 인프라** | 필요 (UWB Anchor) | 불필요 (기존 AP) | Beacon (저가) | 전용 Locator |
| **단말 전력** | 중간 | 중간 | 매우 낮음 | 낮음 |
| **표준** | 802.15.4z | 802.11mc | BT 4.0+ | BT 5.1 |
| **Multipath Robustness** | 높음 | 중간 | 낮음 | 중간 |
| **보안 (Anti-Relay)** | STS(Scrambled Timestamp Sequence) 기반 | 미지원 | 미지원 | 미지원 |

숫자만 보면 UWB가 압도적으로 좋아 보인다. 하지만 현실에서는 정확도만으로 기술을 선택하지 않는다.

## 그래서 언제 뭘 쓰는가

### cm급이 필요하면: UWB

<div style="display: flex; gap: 24px; align-items: flex-end;">
  <figure style="flex: 2; min-width: 0; margin: 0;">
    <img src="/comm-blog/images/airtag-precision-finding.jpg" alt="Apple AirTag의 Precision Finding UI. UWB를 이용해 방향과 거리를 cm 단위로 안내한다.">
    <figcaption>Apple AirTag의 Precision Finding UI. (Image: Apple)</figcaption>
  </figure>
  <figure style="flex: 3; min-width: 0; margin: 0;">
    <img src="/comm-blog/images/galaxy-smarttag.png" alt="Samsung Galaxy SmartTag2와 SmartThings Find UI. 가방에 부착된 SmartTag2의 위치를 지도에서 확인할 수 있다.">
    <figcaption>Samsung Galaxy SmartTag2와 SmartThings Find UI. (Image: Samsung)</figcaption>
  </figure>
</div>
<figcaption style="margin-top: 8px;">AirTag와 SmartTag2 모두 BLE + UWB를 지원하는 대표적인 Item Tracker다.</figcaption>

- **Digital Key** — 차량 잠금 해제에는 cm급 정확도와 Relay Attack 방지가 필수다. Relay Attack은 공격자가 신호를 중계해서 멀리 있는 키를 가까이 있는 것처럼 속이는 공격인데, 기존 Keyless Entry 시스템은 BLE나 NFC 기반이라 이에 취약했다. UWB는 ToF로 실제 물리적 거리를 측정하기 때문에 Relay Latency를 탐지할 수 있고, STS로 타임스탬프를 암호화해서 위조도 방지한다. CCC(Car Connectivity Consortium) 표준이 이 UWB Secure Ranging을 요구한다 [[7]](#ref-7).
- **Asset Tracking** — 공장, 물류 창고에서 선반 단위로 위치를 구분하려면 10cm급이 필요하다. Wi-Fi RTT의 1–2m 정확도로는 인접한 선반을 구분할 수 없고, BLE RSSI의 3–5m는 더더욱 부족하다.
- **AR Navigation** — 카메라 화면 위에 경로 안내 같은 가상 정보를 겹쳐 보여주는 AR Overlay가 자연스러우려면, 사용자 위치를 정밀하게 추적해야 한다. 위치 오차가 1m만 넘어도 Overlay가 벽을 뚫거나 엉뚱한 곳을 가리키게 된다.

대신 전용 Anchor 인프라를 깔아야 한다. 예를 들어 Pozyx의 UWB Anchor는 개당 약 1,000 USD 수준이고 [[13]](#ref-13), 넓은 공간이면 비용이 만만치 않을 것이다.

### 인프라를 새로 깔 수 없으면: Wi-Fi RTT

<figure style="max-width: 400px;">
  <img src="/comm-blog/images/wifi-rtt.png" alt="Wi-Fi RTT 개념도. 기존 Wi-Fi AP를 활용해 실내 위치를 측정한다.">
  <figcaption>Wi-Fi RTT 개념도. 기존 Wi-Fi AP를 활용해 실내 위치를 측정한다. (Image: Navigine)</figcaption>
</figure>

- **쇼핑몰, 공항** — 이미 Wi-Fi AP가 빼곡하다. 경로 안내에는 1–2m 정확도면 충분하고, 이를 위해 UWB Anchor를 새로 설치하는 건 비용 대비 효과가 맞지 않는다.
- **사무실, 병원** — 기존 Wi-Fi 망을 그대로 활용할 수 있어서, 회의실 찾기나 장비 위치 파악 같은 용도에 빠르게 적용 가능하다.

802.11mc를 지원하는 AP가 필요하지만, 요즘 나오는 AP들은 대부분 지원한다. Android 9부터 WifiRttManager API도 제공되어서 앱 개발도 어렵지 않다고 한다 [[8]](#ref-8). 다만 실내 Navigation이 아직 보편화되지 않은 데는 이유가 있다. 실내 지도(Floor Plan) 데이터가 표준화되어 있지 않고, GPS처럼 하나의 통합 플랫폼이 없어서 건물마다 따로 구축해야 한다. 기술보다 인프라와 생태계의 문제다.

Wi-Fi에서는 **CSI(Channel State Information) 기반 Fingerprinting**도 가능하다. CSI는 각 Subcarrier별 채널 응답 정보로, RSSI보다 훨씬 풍부한 공간 특성을 담고 있다. Radio Map을 미리 만들어두고 실시간 CSI를 매칭하는 방식인데, 환경이 바뀌면 Radio Map도 갱신해야 한다는 한계가 있다. RTT와 결합하면 서로 보완이 된다. (나중에 이 내용도 자세히 다뤄보려 한다.)

### 배터리 교체 없이 오래 가야 하면: BLE

<figure style="max-width: 480px;">
  <img src="/comm-blog/images/ibeacon-assortment.jpg" alt="다양한 벤더의 iBeacon 분해 사진. 코인 배터리 하나와 소형 PCB로 구성되어 있다.">
  <figcaption>다양한 벤더의 iBeacon 분해 사진. 코인 배터리 하나와 소형 PCB로 구성되어 있다. (Source: <a href="https://commons.wikimedia.org/wiki/File:An_assortment_of_iBeacon_from_different_vendors.jpg">Wikimedia Commons</a>, CC BY-SA 3.0)</figcaption>
</figure>

- **Proximity Detection** — 매장 근처에 오면 쿠폰 알림을 보내는 정도. 정확한 좌표가 아니라 "이 사람이 매장 근처에 있는가"만 알면 되므로, RSSI의 3–5m 정확도로 충분하다. Beacon 하나에 10–25 USD면 대량 배치에도 부담이 적다.
- **Item Finding** — Apple AirTag, Samsung SmartTag가 대표적이다. BLE Beacon의 초저전력 덕분에 코인 배터리 하나로 약 1년 이상 동작한다.
- **Wearable / IoT** — 배터리 수명이 최우선인 기기. UWB나 Wi-Fi RTT는 전력 소모가 커서 소형 웨어러블에는 적합하지 않다.

## 실제로는 섞어서 쓴다

실제 제품에서 단일 기술만 쓰는 경우는 드물다. 대부분 여러 기술을 조합하는 방식을 사용한다.

**BLE + UWB 2단계.** AirTag, SmartTag 같은 Item Tracker가 대표적이다. 먼 거리에서는 BLE로 대략적인 위치를 파악하고, 가까이 다가가면 UWB로 전환해서 정밀하게 안내한다. 전력과 정밀도를 모두 챙기는 구조다.

**Wi-Fi + BLE Fingerprinting.** Wi-Fi RTT로 거리 기반 위치를 잡은 뒤, BLE RSSI Fingerprint로 보정하는 방식이다.

**IMU(Inertial Measurement Unit) + Wireless.** IMU는 가속도계와 자이로스코프를 포함하는 관성 센서다. PDR(Pedestrian Dead Reckoning, 보행자 추측 항법)로 단기 궤적을 추정하면서, 무선 Localization으로 누적되는 Drift를 주기적으로 보정한다.

이런 융합에서 핵심은 **각 기술의 추정치를 신뢰도에 따라 가중해서 결합**하는 것이다. 예를 들어 가까운 UWB Anchor의 추정치에는 높은 가중치를, 먼 BLE Beacon의 추정치에는 낮은 가중치를 부여한다. 이를 위해 Covariance 기반 Weighted Fusion이나 Kalman Filter가 쓰인다.

## 앞으로 어떻게 달라질까

**UWB** — IEEE 802.15.4ab에서 차세대 표준이 논의되고 있다. 현재 802.15.4z는 최대 6.8Mbps Data Rate에 머물러 있는데, 802.15.4ab는 이를 수십 Mbps 이상으로 끌어올리면서 Ranging 정확도와 저전력 모드도 개선하는 것이 목표다. FiRa Consortium [[7]](#ref-7)과 CCC를 중심으로 Ecosystem이 확장되고 있다.

**Wi-Fi** — Wi-Fi 7(802.11be)의 320MHz Bandwidth는 현재 80–160MHz 대비 Time Resolution을 2–4배 높여서 RTT 정확도를 개선할 여지를 준다. 또 하나 주목할 만한 건 802.11bf의 **Wi-Fi Sensing** 표준이다 [[9]](#ref-9). 기존 Wi-Fi AP가 데이터 통신뿐 아니라 CSI 변화를 분석해서 사람의 존재 여부(Presence Detection)나 손동작(Gesture Recognition)까지 감지할 수 있게 된다. 이것이 표준화되면 별도 센서 없이 Wi-Fi 인프라만으로 Localization과 Sensing을 동시에 수행하는 구조가 가능해진다.

**BLE** — Bluetooth 6.0에서 **Channel Sounding**이 도입되었다 [[10]](#ref-10). 기존 BLE는 RSSI로만 거리를 추정해서 수 미터 오차가 불가피했는데, Channel Sounding은 여러 주파수 채널에서 Phase를 측정해서 거리를 계산하는 Phase-Based Ranging 방식이다. 이를 통해 cm급 거리 추정이 BLE에서도 가능해진다. 이것이 본격화되면 UWB의 영역이었던 정밀 Localization을 BLE가 대체할 가능성도 열린다.

## 정리

UWB, Wi-Fi, BLE는 경쟁 관계라기보다 **상호 보완적**이다. 정밀도가 필요하면 UWB, 인프라 효율이 중요하면 Wi-Fi RTT, 전력과 비용이 우선이면 BLE. 실제 시스템에서는 이걸 조합해서 쓴다.

각 기술의 표준이 빠르게 진화하고 있어서, 기술 간 경계는 점점 더 모호해질 것이다. 예를 들어 BLE Channel Sounding이 보편화되면, 지금은 UWB가 필수인 Digital Key나 Asset Tracking에서도 BLE만으로 충분한 정확도를 확보할 수 있게 된다. 그러면 UWB Anchor 없이 기존 BLE 인프라로 cm급 Localization이 가능해지면서, 비용과 정밀도 사이의 트레이드오프 자체가 달라질 수 있다. Wi-Fi Sensing과 실내 지도 표준화가 함께 진행되면, 아직 보편화되지 못한 실내 길찾기도 현실이 되지 않을까?

## References


<span id="ref-1">1.</span> IEEE 802.15.4z-2020, "IEEE Standard for Low-Rate Wireless Networks — Amendment: Enhanced Ultra Wideband (UWB) Physical Layers and Associated Ranging Techniques," 2020.

<span id="ref-2">2.</span> I. Guvenc and Z. Sahinoglu, "Threshold-Based TOA Estimation for Impulse Radio UWB Systems," *Proc. IEEE International Conference on Ultra-Wideband (ICU)*, pp. 420–425, Sep. 2005.

<span id="ref-3">3.</span> S. Gezici et al., "Localization via Ultra-Wideband Radios: A Look at Positioning Aspects for Future Sensor Networks," *IEEE Signal Processing Magazine*, vol. 22, no. 4, pp. 70–84, Jul. 2005.

<span id="ref-4">4.</span> IEEE 802.11mc (802.11-2016), "IEEE Standard for Information Technology — Fine Timing Measurement (FTM)."

<span id="ref-5">5.</span> Bluetooth SIG, "Bluetooth Core Specification v5.1 — Direction Finding," 2019. [[Link]](https://www.bluetooth.com/specifications/specs/core-specification-5-1/)

<span id="ref-6">6.</span> G. Pau, F. Arena, Y. E. Gebremariam, and I. You, "Bluetooth 5.1: An Analysis of Direction Finding Capability for High-Precision Location Services," *Sensors*, vol. 21, no. 11, article 3589, May 2021. [[Link]](https://doi.org/10.3390/s21113589)

<span id="ref-7">7.</span> FiRa Consortium, "UWB Technology Overview." [[Link]](https://www.firaconsortium.org/discover-uwb/uwb-technology-overview)

<span id="ref-8">8.</span> Google, "Wi-Fi RTT (IEEE 802.11mc) — Android Developers." [[Link]](https://developer.android.com/develop/connectivity/wifi/wifi-rtt)

<span id="ref-9">9.</span> IEEE 802.11bf, "WLAN Sensing (Wi-Fi Sensing)," Task Group bf. [[Link]](https://www.ieee802.org/11/Reports/tgbf_update.htm)

<span id="ref-10">10.</span> Bluetooth SIG, "Bluetooth Core Specification v6.0 — Channel Sounding," 2024. [[Link]](https://www.bluetooth.com/specifications/specs/core-specification-6-0/)

<span id="ref-11">11.</span> Volpis, "Indoor Positioning Systems Implementation Guide," 2025. [[Link]](https://volpis.com/blog/indoor-positioning-system-implementation/)

<span id="ref-12">12.</span> Minew, "Bluetooth 5.1 AoA Indoor Positioning Kit," MinewStore. [[Link]](https://www.minewstore.com/product/bluetooth-5-1-aoa-indoor-positioning)

<span id="ref-13">13.</span> Pozyx, "UWB Anchors — Enterprise," Pozyx Shop. [[Link]](https://www.pozyx.io/products/hardware/hardware-anchors)
