---
title: "Ranging Fundamentals: ToF, TDoA, AoA의 원리와 트레이드오프"
date: 2026-04-02T10:00:00
draft: false
math: true
tags: ["Ranging", "ToF", "TDoA", "AoA", "TWR", "Localization"]
categories: ["Localization"]
summary: "Localization의 기반이 되는 세 가지 Ranging 방식 — ToF, TDoA, AoA — 의 원리를 수식과 함께 정리하고, 각 방식의 트레이드오프와 Hybrid 접근법을 분석합니다."
---

## Introduction

[Indoor Localization 기술 비교](/comm-blog/posts/indoor-localization-uwb-wifi-ble/) 글에서 UWB, Wi-Fi, BLE 각각의 Ranging 원리를 간략히 다루었다. 이 글에서는 그 기반이 되는 세 가지 Ranging 방식 — **ToF, TDoA, AoA** — 의 원리를 수식 레벨에서 정리하고, 각 방식이 가진 트레이드오프를 분석한다.

Ranging은 Localization의 가장 기본적인 단계이다. Anchor와 Target 사이의 **거리** 또는 **방향**을 추정하는 과정이며, 이 추정의 정밀도가 최종 위치 추정 성능을 직접 결정한다.

## ToF (Time of Flight)

### 원리

ToF는 전파가 Anchor와 Target 사이를 이동하는 데 걸리는 시간을 측정하여 거리를 추정한다.

$$d = c \cdot \tau$$

여기서 $c = 3 \times 10^8 \text{ m/s}$는 빛의 속도, $\tau$는 편도 전파 시간이다.

편도 시간을 직접 측정하려면 Anchor와 Target의 Clock이 정밀하게 동기화되어야 한다. ns 단위의 동기 오차가 곧 수십 cm의 거리 오차로 이어지기 때문이다:

$$\Delta d = c \cdot \Delta t = 3 \times 10^8 \times 1 \times 10^{-9} = 0.3\text{ m}$$

**1ns의 Clock 오차가 30cm의 거리 오차**를 만든다. 이것이 ToF 기반 시스템에서 Clock 동기화가 핵심인 이유다.

### Two-Way Ranging (TWR)

Clock 동기화 문제를 우회하는 가장 직관적인 방법이 **TWR(Two-Way Ranging)** 이다. Anchor와 Target이 패킷을 주고받으며 Round Trip Time(RTT)을 측정한다.

```
  Anchor                  Target
    |-------- Tx -------->|  t1
    |                     |  (처리 지연: t_reply)
    |<------- Rx ---------|  t2
```

$$d = \frac{c \cdot (t_{\text{RTT}} - t_{\text{reply}})}{2}$$

TWR의 장점은 **Anchor 간 동기화가 불필요**하다는 것이다. 각 Anchor가 독립적으로 Target과 Ranging을 수행할 수 있다.

단점은 **Clock Drift**에 취약하다는 점이다. Anchor와 Target의 Crystal Oscillator 주파수가 미세하게 다르면, RTT 측정에 오차가 누적된다.

### DS-TWR (Double-Sided TWR)

DS-TWR은 TWR을 두 번 수행하여 Clock Drift의 영향을 상쇄한다.

```
  Anchor                  Target
    |-------- Tx1 ------->|
    |<------- Rx1 ---------|
    |-------- Tx2 -------->|
```

첫 번째 Round에서 Anchor가 먼저 송신하고, 두 번째 Round에서 Target이 먼저 송신한다. 두 RTT를 결합하면:

$$\hat{\tau} = \frac{t_{\text{RTT,1}} \cdot t_{\text{RTT,2}} - t_{\text{reply,A}} \cdot t_{\text{reply,T}}}{t_{\text{RTT,1}} + t_{\text{RTT,2}} + t_{\text{reply,A}} + t_{\text{reply,T}}}$$

이 공식은 양쪽 Clock Drift가 대칭적으로 상쇄되도록 설계되어 있다. UWB(IEEE 802.15.4z)에서 DS-TWR은 사실상 표준 Ranging 프로토콜이다.

### 시간 분해능과 Bandwidth의 관계

ToF 정확도의 물리적 한계는 **Bandwidth**가 결정한다. 신호의 시간 분해능은:

$$\Delta \tau_{\min} = \frac{1}{BW}$$

| 기술 | Bandwidth | 시간 분해능 | 거리 분해능 |
|---|---|---|---|
| UWB | 500MHz | 2ns | 0.6m |
| Wi-Fi 6 | 80MHz | 12.5ns | 3.75m |
| Wi-Fi 7 | 320MHz | 3.1ns | 0.94m |
| BLE | 1MHz | 1μs | 300m |

여기서 **거리 분해능이 곧 정확도는 아니다**. 분해능은 두 개의 Multipath를 구분할 수 있는 최소 거리이고, Leading Edge Detection, Super-Resolution 등의 후처리로 분해능보다 훨씬 높은 정확도를 달성할 수 있다. UWB가 분해능 60cm에서 10cm 이하의 정확도를 내는 것이 이 때문이다.

### Cramer-Rao Lower Bound (CRLB)

ToF 추정의 이론적 정확도 한계는 CRLB로 표현된다. Bandwidth $BW$와 SNR이 주어졌을 때:

$$\sigma_{\tau} \geq \frac{1}{2\pi \cdot BW_{\text{eff}} \cdot \sqrt{2 \cdot \text{SNR}}}$$

여기서 $BW_{\text{eff}}$는 Effective Bandwidth (RMS Bandwidth)이다. 이 식은 두 가지를 말해준다:

1. **Bandwidth가 넓을수록** 추정 정확도가 향상된다 — UWB가 유리한 근본 이유
2. **SNR이 높을수록** 정확도가 향상된다 — 가까운 Anchor의 추정이 더 정확한 이유

## TDoA (Time Difference of Arrival)

### 원리

TDoA는 Target이 송신한 신호가 **여러 Anchor에 도달하는 시간 차이**를 측정한다. ToF와 달리 절대 시간이 아닌 상대 시간을 사용한다.

Anchor $i$와 Anchor $j$에 대한 시간 차이:

$$\Delta \tau_{ij} = \tau_i - \tau_j = \frac{d_i - d_j}{c}$$

이 시간 차이는 기하학적으로 **쌍곡선(Hyperbola)** 을 정의한다. 2D Localization을 위해서는 최소 3개의 Anchor(2개의 독립적인 TDoA)가 필요하며, 쌍곡선의 교점이 Target 위치가 된다.

### ToF와의 핵심 차이

| 항목 | ToF (TWR) | TDoA |
|---|---|---|
| **Target 동작** | 양방향 통신 (Tx + Rx) | 송신만 (Tx only) |
| **Target 전력** | 높음 | 낮음 |
| **동기화 요구** | Anchor-Target 간 불필요 | **Anchor 간 ns급 동기화 필수** |
| **확장성** | Anchor당 순차 Ranging | 동시에 다수 Target 지원 |

TDoA의 최대 장점은 **Target이 한 번만 송신**하면 된다는 것이다. Target의 전력 소모가 적고, Anchor 측에서 수신만 하면 되므로 수백~수천 개의 Target을 동시에 추적할 수 있다. 물류 창고의 대규모 Asset Tracking에서 TDoA가 선호되는 이유다.

대신 **Anchor 간 시간 동기화**가 핵심 과제이다. 모든 Anchor가 공통 시간 기준을 가져야 하며, ns급 동기 오차가 요구된다. 유선 동기(Ethernet PTP)나 무선 동기 프로토콜이 사용된다.

### 수학적 풀이

TDoA 기반 Localization은 비선형 연립방정식을 풀어야 한다. Target 위치 $\mathbf{p} = [x, y]^T$에 대해:

$$\|\mathbf{p} - \mathbf{a}_i\| - \|\mathbf{p} - \mathbf{a}_j\| = c \cdot \Delta \tau_{ij}, \quad \forall (i, j)$$

여기서 $\mathbf{a}_i$는 Anchor $i$의 위치이다. 비선형이므로 다음과 같은 방법으로 풀 수 있다:

- **선형화**: Taylor 전개로 근사 후 Iterative Least Squares
- **폐쇄형 해(Closed-form)**: Chan's Algorithm, Fang's Algorithm
- **최적화**: Maximum Likelihood Estimation

## AoA (Angle of Arrival)

### 원리

AoA는 Antenna Array에 도달하는 신호의 **방향(각도)** 을 추정한다. $N$개의 안테나 소자가 간격 $d$로 배열된 ULA(Uniform Linear Array)에서, 도래각 $\theta$인 신호가 인접 소자 사이에 만드는 Phase Difference는:

$$\Delta \phi = \frac{2\pi d}{\lambda} \sin\theta$$

여기서 $\lambda$는 파장이다. $d = \lambda/2$로 설정하면:

$$\Delta \phi = \pi \sin\theta$$

$N$개 소자의 수신 신호를 벡터로 표현하면:

$$\mathbf{a}(\theta) = \begin{bmatrix} 1 \\ e^{j\Delta\phi} \\ e^{j2\Delta\phi} \\ \vdots \\ e^{j(N-1)\Delta\phi} \end{bmatrix}$$

이 **Steering Vector** $\mathbf{a}(\theta)$와 수신 신호의 관계를 분석하여 $\theta$를 추정한다.

### 추정 알고리즘

**Beamforming (Bartlett)**: 가장 직관적인 방법. 모든 방향에 대해 Beamforming 출력을 계산하고 최대값의 방향을 취한다.

$$P(\theta) = \mathbf{a}^H(\theta) \mathbf{R} \mathbf{a}(\theta)$$

여기서 $\mathbf{R}$은 수신 신호의 Covariance Matrix이다. 분해능이 Beamwidth에 제한된다.

**MUSIC (Multiple Signal Classification)**: Covariance Matrix의 Eigenvalue Decomposition을 통해 Signal Subspace와 Noise Subspace를 분리하고, Noise Subspace와의 직교성을 이용하여 추정한다.

$$P_{\text{MUSIC}}(\theta) = \frac{1}{\mathbf{a}^H(\theta) \mathbf{E}_n \mathbf{E}_n^H \mathbf{a}(\theta)}$$

Beamforming보다 훨씬 높은 분해능을 제공하지만, Snapshot 수가 충분하고 신호 수를 사전에 알아야 한다는 가정이 있다.

**ESPRIT**: Steering Vector의 Shift-Invariance 구조를 이용하여 Spectral Search 없이 직접 각도를 추정한다. MUSIC보다 연산량이 적으면서 유사한 성능을 낸다.

### AoA의 정확도 한계

AoA 추정의 CRLB는:

$$\sigma_\theta \geq \frac{1}{\pi \cos\theta} \cdot \sqrt{\frac{6}{N(N^2-1) \cdot \text{SNR}}}$$

이 식에서 읽어낼 수 있는 것:

1. **안테나 수 $N$이 많을수록** 정확도가 향상된다 — $N^3$에 비례하여 분산이 감소
2. **Broadside($\theta = 0$)에서 가장 정확**하고, Endfire($\theta = \pm 90°$)로 갈수록 성능 저하
3. **SNR이 높을수록** 정확도가 향상된다 — ToF와 동일한 경향

실무적으로 AoA는 **거리가 멀어지면 위치 오차가 증가**한다는 특성이 있다. 각도 오차 $\sigma_\theta$가 일정하더라도, 거리 $d$에서의 위치 오차는 $d \cdot \sigma_\theta$ (라디안)에 비례하기 때문이다. 이것이 AoA 단독보다 ToF와 결합하는 것이 유리한 이유다.

## Hybrid Ranging

### 왜 결합하는가

세 방식의 트레이드오프를 정리하면:

| 항목 | ToF | TDoA | AoA |
|---|---|---|---|
| **추정 대상** | 거리 (Scalar) | 거리 차이 (Scalar) | 방향 (Angle) |
| **최소 Anchor 수 (2D)** | 3 | 3 (기준 포함 4) | 2 |
| **Anchor 동기화** | 불필요 (TWR) | 필수 | 불필요 |
| **Target 전력** | 높음 (양방향) | 낮음 (단방향) | 수동 수신 가능 |
| **원거리 성능** | 안정적 | 안정적 | 열화 ($\propto d$) |
| **Multipath 영향** | Bandwidth 의존 | Bandwidth 의존 | Angular Spread |

단일 방식으로는 모든 상황을 커버하기 어렵다. 예를 들어:

- **ToF만 사용**: Anchor가 2개뿐이면 위치가 고유하게 결정되지 않는다 (Ambiguity 발생)
- **AoA만 사용**: 먼 Target에서 위치 오차가 크게 증가한다
- **TDoA만 사용**: Anchor 동기화 인프라가 반드시 필요하다

### ToF + AoA Joint Estimation

가장 일반적인 Hybrid 구조이다. 하나의 Anchor에서 거리와 방향을 동시에 추정하면, **Anchor 1개로 2D 위치 결정**이 가능하다:

$$\hat{\mathbf{p}} = \mathbf{a}_k + \hat{d}_k \begin{bmatrix} \cos\hat{\theta}_k \\ \sin\hat{\theta}_k \end{bmatrix}$$

여기서 $\hat{d}_k$는 ToF 기반 거리 추정, $\hat{\theta}_k$는 AoA 기반 각도 추정이다.

다수의 Anchor에서 각각 ToF + AoA를 측정하면, 독립적인 위치 추정치 여러 개를 얻는다. 이를 Covariance 기반의 Weighted Fusion으로 결합하면 단일 방식보다 높은 정확도를 달성할 수 있다.

### 실제 시스템의 Hybrid 사례

- **UWB (802.15.4z)**: DS-TWR + AoA (Phase Difference of Arrival). Apple AirTag, Samsung SmartTag2가 이 구조를 사용한다.
- **5G NR Positioning**: DL-TDoA + UL-AoA + Multi-RTT를 결합하는 Hybrid Positioning이 3GPP Rel-16부터 정의되어 있다.
- **BLE 5.1**: AoA/AoD + RSSI. 방향을 AoA로, 대략적 거리를 RSSI로 추정한다.

## 정리

Ranging은 Localization의 기초 단계이며, ToF, TDoA, AoA 각각은 고유한 강점과 한계를 가진다. **어떤 방식이 최적인지는 시스템의 제약 조건** — Anchor 수, 동기화 가능 여부, Target 전력 제한, 요구 정확도 — 에 따라 달라진다.

실제 시스템은 단일 방식보다 Hybrid 구조를 채택하는 경향이 뚜렷하다. 하나의 Anchor에서 거리와 방향을 동시에 추정하고, 여러 Anchor의 결과를 최적으로 결합하는 구조가 정확도와 Robustness 모두에서 유리하다. 다음 포스팅에서는 이 결합 과정 — 즉 여러 추정치를 최적으로 융합하는 **Sensor Fusion**의 원리를 다룰 예정이다.

## References

1. IEEE 802.15.4z-2020, "IEEE Standard for Low-Rate Wireless Networks — Amendment: Enhanced Ultra Wideband (UWB) Physical Layers and Associated Ranging Techniques," 2020.
2. D. Dardari, A. Conti, U. Ferner, A. Giorgetti, and M. Z. Win, "Ranging With Ultrawide Bandwidth Signals in Multipath Environments," *Proceedings of the IEEE*, vol. 97, no. 2, pp. 404–426, Feb. 2009.
3. S. Gezici et al., "Localization via Ultra-Wideband Radios: A Look at Positioning Aspects for Future Sensor Networks," *IEEE Signal Processing Magazine*, vol. 22, no. 4, pp. 70–84, Jul. 2005.
4. G. C. Carter, "Time Delay Estimation for Passive Sonar Signal Processing," *IEEE Transactions on Acoustics, Speech, and Signal Processing*, vol. 29, no. 3, pp. 463–470, Jun. 1981.
5. R. Schmidt, "Multiple Emitter Location and Signal Parameter Estimation," *IEEE Transactions on Antennas and Propagation*, vol. 34, no. 3, pp. 276–280, Mar. 1986.
6. R. Roy and T. Kailath, "ESPRIT — Estimation of Signal Parameters via Rotational Invariance Techniques," *IEEE Transactions on Acoustics, Speech, and Signal Processing*, vol. 37, no. 7, pp. 984–995, Jul. 1989.
7. Y. T. Chan and K. C. Ho, "A Simple and Efficient Estimator for Hyperbolic Location," *IEEE Transactions on Signal Processing*, vol. 42, no. 8, pp. 1905–1915, Aug. 1994.
8. 3GPP TS 38.305, "NG Radio Access Network (NG-RAN); Stage 2 functional specification of User Equipment (UE) positioning in NG-RAN."
9. H. L. Van Trees, *Optimum Array Processing: Part IV of Detection, Estimation, and Modulation Theory*, Wiley, 2002.
