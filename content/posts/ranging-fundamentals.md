---
title: "Ranging Fundamentals: ToF, TDoA, AoA의 원리와 트레이드오프"
date: 2026-04-02T10:00:00
draft: false
math: true
tags: ["Ranging", "ToF", "TDoA", "AoA", "TWR", "Localization"]
categories: ["Localization"]
summary: "Localization의 출발점인 Ranging — 거리를 재는 ToF, 시간 차이를 쓰는 TDoA, 방향을 잡는 AoA — 의 원리와 트레이드오프를 수식과 함께 정리한다."
---

## 왜 Ranging부터 알아야 하는가

[Indoor Localization 기술 비교](/comm-blog/posts/indoor-localization-uwb-wifi-ble/)에서 UWB, Wi-Fi, BLE를 비교했는데, 결국 이 기술들이 하는 일의 출발점은 같다. **Anchor와 Target 사이의 거리 또는 방향을 추정**하는 것. 이 단계를 Ranging이라고 하고, 여기서의 정밀도가 최종 위치 추정 성능을 직접 결정한다.

이 글에서는 ToF, TDoA, AoA 세 가지 Ranging 방식을 수식 레벨에서 정리한다. "이 수식이 뭘 말하려는 건지"를 풀어쓰는 데 초점을 맞추었다.

## ToF (Time of Flight)

### 기본 아이디어

전파가 Anchor에서 Target까지 가는 데 걸리는 시간을 재서 거리를 계산한다. 가장 직관적인 방식이다.

$$d = c \cdot \tau$$

$c$는 빛의 속도($3 \times 10^8$ m/s), $\tau$는 편도 전파 시간.

여기서 바로 문제가 하나 생�다. 편도 시간을 재려면 Anchor와 Target의 **Clock이 정확하게 동기화**되어 있어야 한다. 얼마나 정확해야 하냐면:

$$\Delta d = c \cdot \Delta t = 3 \times 10^8 \times 1 \times 10^{-9} = 0.3\text{ m}$$

**1ns 동기 오차 = 30cm 거리 오차.** 저가 Crystal Oscillator로는 이 수준의 동기화가 어렵다.

### TWR: 동기화 없이 거리를 재는 방법

그래서 나온 게 **TWR(Two-Way Ranging)**이다. 동기화 대신 왕복 시간을 잰다.

```
  Anchor                  Target
    |-------- Tx -------->|  t1
    |                     |  (처리 지연: t_reply)
    |<------- Rx ---------|  t2
```

$$d = \frac{c \cdot (t_{\text{RTT}} - t_{\text{reply}})}{2}$$

Anchor가 자기 Clock으로 보낸 시각과 받은 시각의 차이만 재면 되니까, **Anchor 간 동기화가 필요 없다**. 각 Anchor가 독립적으로 Target과 Ranging을 수행할 수 있다.

문제는 **Clock Drift**다. 양쪽 Crystal Oscillator의 주파수가 미세하게 다르면 RTT 측정에 오차가 쌓인다. 이걸 해결하는 게 DS-TWR이다.

### DS-TWR: Clock Drift를 상쇄한다

DS-TWR(Double-Sided TWR)은 TWR을 두 번 수행한다. 첫 번째는 Anchor가 먼저 송신, 두 번째는 Target이 먼저 송신. 두 RTT를 조합하면:

$$\hat{\tau} = \frac{t_{\text{RTT,1}} \cdot t_{\text{RTT,2}} - t_{\text{reply,A}} \cdot t_{\text{reply,T}}}{t_{\text{RTT,1}} + t_{\text{RTT,2}} + t_{\text{reply,A}} + t_{\text{reply,T}}}$$

이 식의 핵심은 **양쪽 Clock Drift가 대칭적으로 상쇄**된다는 것이다. 수식이 좀 복잡해 보이지만, 아이디어는 "두 번 재서 평균 내면 편향이 줄어든다"와 비슷하다. UWB(802.15.4z)에서 DS-TWR은 사실상 표준 프로토콜이다.

### Bandwidth가 정확도의 상한을 결정한다

ToF의 물리적 한계는 결국 **Bandwidth**가 결정한다.

$$\Delta \tau_{\min} = \frac{1}{BW}$$

숫자로 보면 체감이 된다:

| 기술 | Bandwidth | 시간 분해능 | 거리 분해능 |
|---|---|---|---|
| UWB | 500MHz | 2ns | 0.6m |
| Wi-Fi 6 | 80MHz | 12.5ns | 3.75m |
| Wi-Fi 7 | 320MHz | 3.1ns | 0.94m |
| BLE | 1MHz | 1μs | 300m |

BLE의 거리 분해능 300m을 보면 왜 BLE가 RSSI 기반으로 갈 수밖에 없는지 이해가 된다. ToF로는 아무것도 못 한다.

한 가지 주의할 점 — **분해능 ≠ 정확도**다. 분해능은 두 Multipath를 구분하는 최소 거리. Leading Edge Detection이나 Super-Resolution 같은 후처리를 거치면 분해능보다 훨씬 높은 정확도를 달성할 수 있다. UWB가 분해능 60cm에서 10cm 이하 정확도를 내는 것이 이 때문이다.

### CRLB: 이론적 한계

ToF 추정 정확도의 이론적 하한은 CRLB(Cramer-Rao Lower Bound)로 표현된다:

$$\sigma_{\tau} \geq \frac{1}{2\pi \cdot BW_{\text{eff}} \cdot \sqrt{2 \cdot \text{SNR}}}$$

$BW_{\text{eff}}$는 Effective Bandwidth(RMS Bandwidth). 이 식이 말하는 건 두 가지다:

1. **Bandwidth가 넓을수록** 정확도 향상 — UWB가 유리한 근본적인 이유
2. **SNR이 높을수록** 정확도 향상 — 가까운 Anchor의 추정이 더 정확한 이유

연구할 때 CRLB를 자주 보게 되는데, 이게 좋은 이유는 "이 조건에서 어떤 알고리즘을 쓰든 이 이상은 못 간다"는 레퍼런스 라인을 제공하기 때문이다. 내 알고리즘이 CRLB에 가까우면 잘 설계된 것이고, 한참 떨어져 있으면 개선 여지가 있다는 뜻이다.

## TDoA (Time Difference of Arrival)

### 절대 시간 대신 상대 시간을 쓴다

TDoA는 Target이 송신한 신호가 **여러 Anchor에 도달하는 시간 차이**를 측정한다:

$$\Delta \tau_{ij} = \tau_i - \tau_j = \frac{d_i - d_j}{c}$$

이 시간 차이는 기하학적으로 **쌍곡선(Hyperbola)**을 정의한다. "Anchor $i$와 Anchor $j$까지의 거리 차이가 일정한 점들의 궤적"이 쌍곡선이니까. 2D에서 위치를 결정하려면 최소 3개 Anchor(2개의 독립 TDoA)가 필요하고, 쌍곡선의 교점이 Target 위치가 된다.

### ToF와 뭐가 다른가

| 항목 | ToF (TWR) | TDoA |
|---|---|---|
| **Target 동작** | 양방향 통신 (Tx + Rx) | 송신만 (Tx only) |
| **Target 전력** | 높음 | 낮음 |
| **동기화 요구** | Anchor-Target 간 불필요 | **Anchor 간 ns급 동기화 필수** |
| **확장성** | Anchor당 순차 Ranging | 동시에 다수 Target 지원 |

TDoA의 킬러 피처는 **Target이 한 번만 Tx하면 된다**는 것이다. Target은 그냥 신호를 한 번 보내면 되고, Anchor 쪽에서 수신 시각을 비교한다. 덕분에 Target 전력 소모가 적고, 수백~수천 개 Target을 동시에 추적할 수 있다. 물류 창고에서 수천 개 자산을 동시에 트래킹해야 하는 시나리오에서 TDoA가 선호되는 이유다.

대신 **Anchor 간 ns급 시간 동기화**가 필수다. 유선(Ethernet PTP)이든 무선이든 동기화 인프라를 구축해야 한다. TWR의 "동기화 불필요"와는 정반대의 트레이드오프.

### 풀어야 하는 수학

TDoA 기반 Localization은 비선형 연립방정식이다:

$$\|\mathbf{p} - \mathbf{a}_i\| - \|\mathbf{p} - \mathbf{a}_j\| = c \cdot \Delta \tau_{ij}, \quad \forall (i, j)$$

$\mathbf{p}$는 Target 위치, $\mathbf{a}_i$는 Anchor 위치. 비선형이라 직접 풀 수가 없고, 보통 세 가지 접근법을 쓴다:

- **선형화** — Taylor 전개로 근사 후 Iterative Least Squares. 초기값이 필요하다.
- **폐쇄형 해** — Chan's Algorithm 같은 방식. 초기값 없이 풀 수 있지만 noise에 더 민감할 수 있다.
- **Maximum Likelihood** — 가장 정확하지만 연산량이 크다.

## AoA (Angle of Arrival)

### 위상 차이로 방향을 잡는다

AoA는 ToF, TDoA와 접근이 다르다. 거리가 아니라 **방향(각도)** 을 추정한다.

$N$개의 안테나 소자가 간격 $d$로 나열된 ULA(Uniform Linear Array)를 생각해보자. 도래각 $\theta$인 신호가 인접 소자 사이에 만드는 Phase Difference는:

$$\Delta \phi = \frac{2\pi d}{\lambda} \sin\theta$$

$d = \lambda/2$로 설정하면 $\Delta \phi = \pi \sin\theta$. 이 Phase Difference를 측정하면 $\theta$를 역으로 구할 수 있다.

$N$개 소자의 수신 신호를 벡터로 쓰면:

$$\mathbf{a}(\theta) = \begin{bmatrix} 1 \\ e^{j\Delta\phi} \\ e^{j2\Delta\phi} \\ \vdots \\ e^{j(N-1)\Delta\phi} \end{bmatrix}$$

이걸 **Steering Vector**라고 한다. 수신 신호와 Steering Vector의 관계를 분석해서 $\theta$를 추정하는 것이 AoA의 핵심이다.

### 추정 알고리즘들

**Beamforming (Bartlett).** 가장 직관적이다. 모든 방향으로 빔을 돌려보고, 출력이 가장 큰 방향을 취한다.

$$P(\theta) = \mathbf{a}^H(\theta) \mathbf{R} \mathbf{a}(\theta)$$

$\mathbf{R}$은 수신 신호의 Covariance Matrix. 간단하지만 분해능이 Beamwidth에 제한된다. 비유하자면 손전등을 360도 돌려서 가장 밝은 방향을 찾는 것과 비슷한데, 손전등 빔이 넓으면 정확한 방향을 못 잡는다.

**MUSIC.** Covariance Matrix를 Eigenvalue Decomposition해서 Signal Subspace와 Noise Subspace를 분리한다. Noise Subspace와의 직교성을 이용하면 Beamwidth 한계를 넘어서는 분해능을 얻을 수 있다.

$$P_{\text{MUSIC}}(\theta) = \frac{1}{\mathbf{a}^H(\theta) \mathbf{E}_n \mathbf{E}_n^H \mathbf{a}(\theta)}$$

분해능은 뛰어나지만, Snapshot 수가 충분해야 하고 신호 수를 사전에 알아야 한다.

**ESPRIT.** Steering Vector의 Shift-Invariance 구조를 이용해서 Spectral Search 없이 직접 각도를 계산한다. MUSIC보다 연산량이 적으면서 비슷한 성능.

### AoA의 정확도 한계

AoA 추정의 CRLB:

$$\sigma_\theta \geq \frac{1}{\pi \cos\theta} \cdot \sqrt{\frac{6}{N(N^2-1) \cdot \text{SNR}}}$$

이 식에서 읽어낼 수 있는 것:

1. **안테나 수 $N$이 늘어나면** 분산이 $N^3$에 반비례하여 감소한다. 안테나를 두 배로 늘리면 정확도가 크게 올라간다.
2. **Broadside($\theta = 0°$)에서 가장 정확**하고, Endfire($\theta = \pm 90°$)로 가면 $\cos\theta$가 0에 가까워져서 성능이 급격히 나빠진다.
3. **SNR이 높을수록** 정확도 향상.

그런데 AoA에는 ToF와 다른 특성이 하나 있다. 각도 오차 $\sigma_\theta$가 일정하더라도, 거리가 $d$이면 위치 오차는 $d \cdot \sigma_\theta$에 비례한다. **멀어지면 위치 오차가 선형으로 커진다.** ToF는 거리에 관계없이 오차가 비교적 일정한데, AoA는 그렇지 않다. 이것이 AoA를 단독으로 쓰기 어려운 이유이고, ToF와 결합하면 서로 보완이 되는 이유다.

## Hybrid Ranging: 왜 섞어 쓰는가

세 방식의 트레이드오프를 보면, 어느 하나가 모든 면에서 최적이 아니다:

| 항목 | ToF | TDoA | AoA |
|---|---|---|---|
| **추정 대상** | 거리 | 거리 차이 | 방향 |
| **최소 Anchor 수 (2D)** | 3 | 3 (기준 포함 4) | 2 |
| **Anchor 동기화** | 불필요 (TWR) | 필수 | 불필요 |
| **Target 전력** | 높음 | 낮음 | 수동 수신 가능 |
| **원거리 성능** | 안정적 | 안정적 | 열화 ($\propto d$) |

단일 방식의 한계는 명확하다:
- **ToF만** — Anchor가 2개뿐이면 Ambiguity가 생긴다
- **AoA만** — 먼 Target에서 위치 오차가 커진다
- **TDoA만** — 동기화 인프라가 반드시 필요하다

### ToF + AoA: Anchor 하나로 위치를 잡는다

가장 대표적인 Hybrid 구조. 하나의 Anchor에서 거리와 방향을 동시에 추정하면, **Anchor 1개만으로 2D 위치 결정**이 가능하다:

$$\hat{\mathbf{p}} = \mathbf{a}_k + \hat{d}_k \begin{bmatrix} \cos\hat{\theta}_k \\ \sin\hat{\theta}_k \end{bmatrix}$$

여러 Anchor에서 각각 ToF + AoA를 측정하면 독립적인 위치 추정치가 여러 개 나온다. 이걸 Covariance 기반 Weighted Fusion으로 결합하면 단일 방식보다 높은 정확도를 달성할 수 있다.

### 실제로 쓰이는 Hybrid 사례

- **UWB (802.15.4z)** — DS-TWR + AoA. Apple AirTag, Samsung SmartTag2가 이 구조.
- **5G NR Positioning** — DL-TDoA + UL-AoA + Multi-RTT를 결합하는 Hybrid가 3GPP Rel-16부터 정의되어 있다.
- **BLE 5.1** — AoA/AoD + RSSI. 방향은 AoA로, 대략적 거리는 RSSI로.

## 정리

Ranging은 Localization의 기초다. ToF, TDoA, AoA 각각은 고유한 강점과 한계를 가지고 있고, **어떤 방식이 최적인지는 시스템 제약 조건** — Anchor 수, 동기화 가능 여부, Target 전력 제한, 요구 정확도 — 에 따라 달라진다.

실제 시스템은 거의 예외 없이 Hybrid를 쓴다. Anchor 하나에서 거리와 방향을 동시에 추정하고, 여러 Anchor의 결과를 최적으로 결합하는 구조. 이 "결합" 부분 — 여러 추정치를 신뢰도에 따라 융합하는 **Sensor Fusion** — 은 별도 포스팅에서 다룰 예정이다.

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
