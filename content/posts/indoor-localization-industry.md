---
title: "Indoor Localization 산업: 사례, 생태계, 그리고 경쟁 구도의 변화"
date: 2026-04-04
draft: false
math: false
tags: ["Indoor Localization", "RTLS", "UWB", "BLE", "Asset Tracking", "Smart Factory"]
categories: ["Localization"]
summary: "Indoor Localization이 실제 산업에서 어떻게 쓰이는지, 물류·제조·리테일·헬스케어·스포츠의 배포 사례와 Qorvo, Apple, Samsung, NXP, Nordic 등 벤더별 구현 차이를 정리한다."
---

## Indoor Localization은 어디에 쓰이는가

[이전 포스팅](/comm-blog/posts/indoor-localization-uwb-wifi-ble/)에서 UWB, Wi-Fi RTT, BLE의 기술적 차이를 정리했다. 그런데 실제 산업 현장에서는 "어떤 기술이 더 정밀한가"보다 **"이 환경에서 무엇이 가장 효과적인가"**가 더 중요한 질문이다.

같은 UWB라도 물류 창고에서 자산을 추적하는 것과 병원에서 환자를 추적하는 것은 요구사항이 전혀 다르다. 이 글에서는 산업별로 어떤 기술이 왜 선택되었는지, 실제 배포 사례를 중심으로 살펴본다.

## 물류/창고: 자산과 동선의 실시간 파악

<figure>
  <img src="/comm-blog/images/Sewio-Case-Study-Header-Image.png" alt="UWB RTLS가 적용된 물류 창고 내부. 포크리프트와 작업자가 실시간으로 추적된다.">
  <figcaption>UWB RTLS가 적용된 물류/제조 환경. (Source: Sewio)</figcaption>
</figure>

물류/창고 환경의 핵심 요구사항은 **수천 개 자산의 실시간 위치 파악**과 **작업자/장비 동선 최적화**다. 선반 단위(30–40cm급)의 정확도가 필요하기 때문에 UWB 기반 RTLS가 주로 쓰인다.

### Sewio + VELUX: 창문 제조 공장의 자재 추적

VELUX(덴마크 창문 제조사)는 Sewio의 UWB RTLS를 도입해 공장 내 자재와 반제품의 위치를 실시간으로 추적한다. 도입 결과 생산성이 10% 향상되었고, 유지보수 성능은 50% 개선되었다. 재공품(WIP, Work in Progress)도 10% 감소했다 [[1]](#ref-1).

수작업으로 "이 부품이 지금 어디 있는지" 찾던 시간이 사라진 것이 가장 큰 효과다. 물류에서 자산 검색 시간 절감은 단순한 편의가 아니라 직접적인 비용 절감이다.

### Ubisense + Airbus: 10개 글로벌 사이트의 공구 추적

Airbus는 Ubisense와 10년 장기 라이선스 계약을 체결하고, 전 세계 10개 생산 시설에 UWB RTLS를 배포했다 [[2]](#ref-2). 항공기 조립에 사용하는 공구의 위치를 실시간으로 추적하여, 바코드 스캔 작업을 제거했다. 공구 하나당 약 6초의 스캔 시간이 절약되는데, 하루 수천 건이 누적되면 상당한 효율 개선이다.

Ubisense의 SmartSpace 소프트웨어는 단순 위치 추적을 넘어서, 교정 기한이 지났거나 허가되지 않은 구역에서 사용되는 공구에 대해 자동으로 Alert을 발생시키거나 작동을 차단(Interlock)하는 기능도 제공한다.

## 제조/스마트팩토리: 안전과 생산 효율

제조 환경은 물류와 비슷하게 UWB가 주력이지만, **작업자 안전**이라는 추가 요구사항이 있다. Geofencing을 통한 충돌 방지, 위험 구역 접근 제어 등이 핵심이다.

### Siemens SIMATIC RTLS + Volkswagen Autoeuropa: 작업자-차량 충돌 방지

Volkswagen의 포르투갈 Autoeuropa 공장(연간 약 20만 대 생산)에서는 Siemens SIMATIC RTLS를 사용한다 [[3]](#ref-3). 이 공장의 교차로에서는 시간당 약 120대의 차량(AGV 포함)이 오가는데, UWB 기반 Geofencing과 신호등 시스템을 연동하여 작업자와 차량의 충돌을 방지한다.

위치 데이터에 AI를 결합한 Location Intelligence로 Geofence 관리를 자동화했고, 초기 구역 성공 이후 공장 전체로 확대를 검토 중이다.

### KINEXON + BMW: 조립 라인의 차량 위치 추적

BMW는 KINEXON OS 플랫폼을 도입하여 조립 라인의 모든 차량에 e-Paper 태그를 부착하고, UWB + RFID + GPS를 결합한 Multi-technology 위치 추적을 수행한다 [[4]](#ref-4). 차량 식별을 위한 수동 스캔의 99%를 제거했고, 특정 조립 구역/공정에서만 공구가 작동하도록 하는 Automated Tool Approval도 구현했다.

### Sewio + Toyota: eKanban으로 Just-in-Time 실현

Toyota는 Sewio UWB RTLS를 도입하여 수작업 Kanban 카드를 eKanban으로 대체했다 [[5]](#ref-5). 창고 정보의 리드타임이 **8시간에서 1초 미만**으로 줄었고, 안전 재고도 8시간분에서 4시간분으로 절반이 되었다. 진정한 의미의 Just-in-Time에 가까워진 사례로, 시스템 투자 비용은 2년 만에 회수되었다.

## 리테일: 고객 동선과 Proximity Marketing

리테일은 제조/물류와 요구사항이 다르다. cm급 정확도보다는 **넓은 면적에 저비용으로 배포**하는 것이 중요하다. BLE Beacon이 주력 기술인 이유다.

### Navigine + Kesko: 100개 이상 슈퍼마켓의 매장 내 내비게이션

핀란드 유통업체 Kesko는 Navigine 플랫폼과 BLE Beacon을 결합하여 100개 이상의 슈퍼마켓에 매장 내 내비게이션을 도입했다 [[6]](#ref-6). 쇼핑카트에 태블릿을 장착하여 고객이 장보기 목록을 입력하면 최적 동선을 안내한다.

결과는 인상적이다. 평균 구매 금액이 6% 증가했고, 카트 사용자의 50%가 Push Notification을 수신했으며, 위치 기반 광고의 전환율은 약 68%에 달했다. 25%의 고객이 추천 동선을 따라 이동한 것도 주목할 만하다.

### Kontakt.io + McDonald's: 위치 기반 프로모션

McDonald's는 Kontakt.io의 BLE Beacon을 활용한 로열티 앱 기반 프로모션을 진행했다 [[7]](#ref-7). 매장 내 고객에게 모바일 전용 아이스커피 쿠폰을 Push하는 방식으로, 매장 내 전환율 20%, 쿠폰 수령 고객의 재방문율 30%를 기록했다.

<figure>
  <img src="/comm-blog/images/ibeacon-assortment.jpg" alt="다양한 BLE Beacon 모듈 분해 사진. 코인셀 배터리와 소형 PCB로 구성된다." style="max-width: 500px; display: block; margin: 0 auto;">
  <figcaption>다양한 형태의 BLE Beacon. 코인셀 배터리와 소형 PCB로 구성되어 단가가 낮다.</figcaption>
</figure>

리테일에서 BLE가 선호되는 이유는 명확하다. Beacon 하나의 가격이 수 달러 수준이고, 기존 매장 인프라를 크게 바꿀 필요가 없으며, 고객 스마트폰이 수신기 역할을 하기 때문에 별도 디바이스가 불필요하다.

## 헬스케어: 장비 추적과 감염 관리

병원 환경은 독특한 요구사항이 있다. **Room-level 정확도**면 충분한 경우가 많지만, 위생 모니터링이나 접촉 추적 같은 특수한 Use Case가 있다. IR(적외선) + RFID Hybrid나 UWB가 주로 쓰인다.

<figure>
  <img src="/comm-blog/images/Use-Case-Feature-Asset-Tracking-Solution-–-AssetsRT.jpg" alt="병원 RTLS 대시보드. 장비 위치가 Floor Plan 위에 실시간으로 표시된다." style="max-width: 520px; display: block; margin: 0 auto;">
  <figcaption>병원 RTLS 대시보드. 장비 목록과 Floor Plan 위 위치가 실시간으로 표시된다. (Source: CenTrak)</figcaption>
</figure>

### CenTrak + Texas Health Resources: 연간 $412,000 절감

Texas Health Resources는 CenTrak의 IR + RFID Hybrid RTLS를 도입하여 첫해에 $412,000의 비용을 절감했다 [[8]](#ref-8). 상처 치료용 VAC 장비 렌탈 비용만 $160,000 이상 줄었고, IV Pump 20대의 추가 구매를 Par-level 최적화로 회피했다. 의공학 장비 수리 건수는 80% 감소했다.

병원에서 장비를 찾는 데 간호사가 교대당 평균 1시간을 소모한다는 조사 결과가 있는데, RTLS 도입 후 이 시간이 4분 미만으로 줄어든 사례도 보고된다 [[9]](#ref-9).

### CenTrak + Denver Health: 손위생 모니터링

Denver Health는 CenTrak의 전자 손위생 모니터링 시스템을 도입하여 손위생 준수율을 75% 향상시켰다 [[10]](#ref-10). 1,700개 이상의 Hand Hygiene Station에 RFID Access Point와 IR 모듈을 설치하여, 의료진이 손위생을 수행했는지 자동으로 기록한다.

병원 내 감염(HAI, Hospital-Acquired Infection)은 환자 안전과 직결되는 문제로, 위치 기반 손위생 모니터링만으로 HAI를 40% 이상 줄일 수 있다는 보고가 있다.

## 스포츠: 선수 추적과 경기 분석

스포츠는 Indoor Localization 기술의 가장 고성능 적용 분야다. **10–15cm 정확도, 10ms 이하 지연시간, 초당 수십 회 위치 갱신**이 요구된다.

### Zebra Technologies + NFL: 전 32개 구장 선수 추적

NFL은 2014년부터 Zebra Technologies의 UWB 기반 Active RFID 시스템을 전 32개 구장에 배포하고 있다 [[11]](#ref-11). 구장당 22–24개의 안테나가 설치되고, 선수는 숄더패드 아래에 2개의 태그를 착용한다(라인맨은 3개). 선수 위치는 12Hz, 공 위치는 25Hz로 추적되며, 정확도는 약 15cm(6인치) 이내다.

수집된 데이터는 약 5초 만에 처리되어 Next Gen Stats로 제공되며, 플레이당 200개 이상의 고유 Metric이 생성된다.

### KINEXON + NBA: 80% 이상의 팀이 사용

NBA에서는 KINEXON의 UWB 기반 Local Positioning System(LPS)을 Dallas Mavericks, Chicago Bulls, Philadelphia 76ers 등 80% 이상의 팀이 사용한다 [[12]](#ref-12). 선수는 15.4g의 센서를 반바지 허리밴드에 착용하고, 코트에는 12개의 UWB Anchor가 설치된다. 정확도는 10cm 이하, 코치에게 전달되는 지연시간은 약 10ms로, 실시간 전술 분석이 가능하다.

### KINEXON + Adidas + FIFA: 월드컵 공인구 내장 센서

2022 FIFA 월드컵에서는 공인구(Adidas Al Rihla) 내부에 KINEXON의 14g UWB + IMU 센서가 내장되었다 [[13]](#ref-13). 12대의 Hawk-Eye 카메라가 선수의 29개 신체 포인트를 초당 50회 추적하는 것과 결합하여, Semi-Automated Offside Technology(SAOT)를 구현했다. 오프사이드 판정의 정확도와 속도를 크게 개선한 사례다.

## 기술 선택 기준: 산업별 요약

| 산업 | 주요 기술 | 정확도 | 핵심 벤더 | 핵심 요구사항 |
|---|---|---|---|---|
| 물류/창고 | UWB | 30–40cm | Sewio(HID), Ubisense, Zebra | 자산 실시간 추적, 동선 최적화 |
| 제조 | UWB (+RFID/GPS) | 30cm | Siemens, KINEXON, Sewio | 작업자 안전, 공정 자동화 |
| 리테일 | BLE Beacon | 1–3m | Kontakt.io, Navigine | 저비용 대면적, 고객 동선 |
| 헬스케어 | IR + RFID Hybrid | Room-level – Sub-meter | CenTrak, Zebra | 장비 추적, 감염 관리 |
| 스포츠 | UWB + RFID | 10–15cm | Zebra, KINEXON | 초저지연, 고빈도 추적 |

결국 기술 선택은 **정확도 요구사항 × 배포 비용 × 환경 제약**의 트레이드오프다.

- **cm급 정확도**가 필요하면 UWB가 사실상 유일한 선택이다. 다만 Anchor 인프라 비용이 높다.
- **넓은 면적에 저비용으로 배포**해야 하면 BLE Beacon이 합리적이다. 정확도는 1–3m로 떨어지지만, Beacon 단가가 낮고 고객 스마트폰을 활용할 수 있다.
- **Room-level이면 충분**하고 특수 기능(손위생 등)이 필요하면 IR + RFID Hybrid가 병원 환경에 최적화되어 있다.

어떤 산업이든 단일 기술로 모든 요구사항을 충족하기 어렵기 때문에, 실제 배포에서는 복수 기술을 조합하는 경우가 많다. KINEXON OS가 UWB + RFID + GPS를 통합하는 것이나, omlox 같은 개방형 표준이 등장하는 것도 이런 맥락이다.

## 상용 솔루션 비교: 벤더별 구현 차이

산업 사례를 보면 반복적으로 등장하는 벤더와 칩셋이 있다. 이 생태계의 구조를 이해하면 기술 선택이 좀 더 명확해진다.

### UWB Chipset: Qorvo (Decawave)

<figure>
  <img src="/comm-blog/images/DW3110_PDP.png" alt="Qorvo DW3110 UWB 칩셋 패키지." style="max-width: 200px; display: block; margin: 0 auto;">
  <figcaption>Qorvo DW3110 UWB Transceiver IC. (Source: Qorvo)</figcaption>
</figure>

UWB Localization 시장에서 Qorvo(2020년 Decawave 인수)는 사실상 표준 칩셋 공급자다. 핵심 제품 두 개를 비교하면:

| 항목 | DW1000 | DW3000 |
|---|---|---|
| 표준 | 802.15.4-2011 | 802.15.4z |
| 채널 | CH1–CH7 | CH5 (6.5GHz), CH9 (8GHz) |
| 보안 | 기본 Frame Filtering | STS (Scrambled Timestamp Sequence) |
| 패키지 | 6×6mm QFN | 2.2×3.2mm |
| Sleep 전류 | ~1µA | ~0.5µA |

DW3000의 핵심 차이는 **802.15.4z 지원**이다. 802.15.4z가 추가한 STS는 거리 조작 공격(Relay Attack)을 방지하는 보안 기능으로, FiRa Consortium과 CCC(Car Connectivity Consortium) 호환의 전제 조건이다. DW1000은 이 생태계에 참여할 수 없다 [[14]](#ref-14).

Sewio, KINEXON 등 Enterprise RTLS 벤더 대부분이 Qorvo 칩셋을 기반으로 하고 있고, Apple과 Samsung에도 UWB 부품을 공급하는 것으로 알려져 있다.

### Consumer: Apple vs Samsung

소비자 시장에서 UWB의 양대 구현은 Apple과 Samsung이다.

<figure>
  <img src="/comm-blog/images/airtag-precision-finding.jpg" alt="Apple Precision Finding. iPhone과 Apple Watch에서 방향 화살표와 거리를 표시한다." style="max-width: 360px; display: block; margin: 0 auto;">
  <figcaption>Apple Precision Finding. UWB로 방향과 거리를 실시간 표시한다. (Source: Apple)</figcaption>
</figure>

**Apple**은 자체 설계한 U1(iPhone 11, 2019)과 U2(iPhone 15, 2023) 칩을 사용한다. U2는 U1 대비 UWB Ranging 전력 효율이 약 3배 개선되었고, Thread 라디오가 통합되어 스마트홈 제어를 직접 지원한다 [[15]](#ref-15). AirTag는 평소 BLE로 Beaconing하다가 Precision Finding 시에만 UWB를 활성화하는 구조로, CR2032 배터리로 약 1년간 동작한다.

Apple의 가장 큰 강점은 **Find My 네트워크**다. 10억 대 이상의 Apple 기기가 릴레이 노드 역할을 하여, 분실된 AirTag의 BLE 신호를 수신하고 E2E 암호화된 위치 리포트를 서버에 업로드한다. 도시 지역에서는 수 분 내에 분실물 위치를 파악할 수 있다.

<figure>
  <img src="/comm-blog/images/galaxy-smarttag.png" alt="Samsung Galaxy SmartTag와 SmartThings Find 앱 화면." style="max-width: 400px; display: block; margin: 0 auto;">
  <figcaption>Samsung Galaxy SmartTag과 SmartThings Find. (Source: Samsung)</figcaption>
</figure>

**Samsung**은 자체 Exynos Connect U100 UWB 칩을 개발했다. FiRa와 CCC Digital Key 3.0을 지원하여, 타사 기기와의 호환성에서 Apple보다 개방적이다. SmartThings Find 네트워크는 약 2억 대 이상의 Galaxy 기기를 활용하지만, Apple 대비 밀도가 낮아 오프라인 추적 신뢰도는 떨어진다 [[16]](#ref-16).

흥미로운 점은 Samsung이 SmartTag2에서 **UWB를 제거**하고 BLE 전용으로 출시했다는 것이다. 분실물 추적기에서는 UWB 정밀도보다 크라우드소싱 네트워크 커버리지와 배터리 수명이 더 중요하다는 판단으로 보인다. 실제로 SmartTag2의 배터리 수명은 약 500일로, UWB를 탑재했던 SmartTag+(약 120일)의 4배 이상이다.

### Automotive: NXP Trimension

자동차 Digital Key 시장에서는 NXP가 지배적이다. Trimension 라인업 중 NCJ29D5는 AEC-Q100 인증(자동차 등급, -40°C–105°C)을 받은 UWB IC로, 차량 주변에 4–8개 배치하여 스마트폰/키포브와 TWR로 위치를 결정한다 [[17]](#ref-17).

BLE만으로도 차량 잠금 해제는 가능하지만, Relay Attack에 취약하다. CCC Digital Key 3.0이 UWB Secure Ranging을 요구하는 이유다. BMW가 2022년 최초로 NXP + Apple U1 기반의 UWB Digital Key를 양산 적용했고, Volkswagen, Hyundai/Kia, Ford도 뒤따르고 있다.

### BLE Direction Finding: Nordic Semiconductor

BLE 5.1의 AoA/AoD Direction Finding은 UWB보다 훨씬 낮은 비용과 전력으로 위치 추정이 가능하다. Nordic의 nRF52833/nRF5340 시리즈가 대표적이다 [[18]](#ref-18).

| 항목 | BLE 5.1 AoA (Nordic) | UWB (Qorvo DW3000) |
|---|---|---|
| 정확도 | 0.5–3m | 10–30cm |
| TX 전류 | ~5mA | ~80mA |
| Tag 단가 | < $2 | $5–10 |
| Multipath 내성 | 낮음 (Narrowband) | 높음 (Wideband Pulse) |
| Secure Ranging | 미지원 | 802.15.4z STS |

정확도는 UWB에 비해 떨어지지만, Room-level이면 충분한 환경(병원 장비 추적, 매장 구역 분석)에서는 BLE가 비용 대비 효과적이다. Tag 하나에 수 달러, 코인셀 배터리로 수 년간 동작한다는 점이 대규모 배포에서 결정적 장점이 된다.

### Enterprise RTLS 플랫폼 비교

앞서 다룬 산업 사례에서 반복 등장한 세 플랫폼을 비교하면:

| 항목 | Sewio | Ubisense | KINEXON |
|---|---|---|---|
| 정확도 | ~30cm (TDoA) | ~15cm (AoA+TDoA) | ~10cm |
| Ranging 방식 | TDoA | AoA + TDoA Hybrid | TDoA / TWR |
| 태그 갱신 주기 | 최대 20Hz | 최대 10Hz | 최대 50Hz |
| 소프트웨어 초점 | Zone 분석, Heatmap | Process Intelligence | 실시간 분석, Sports Analytics |
| 주력 산업 | 제조, 물류 | 항공/자동차 OEM | 프로 스포츠, 제조 |

Ubisense가 정확도에서 강점을 가지는 이유는 AoA + TDoA Hybrid 방식 때문이다. Anchor에 안테나 어레이를 장착하여 각도 정보까지 활용하므로, 적은 수의 Anchor로도 높은 정확도를 달성한다. 다만 2021년에 RTLS 하드웨어 사업부를 매각하고, SmartSpace 소프트웨어 플랫폼에 집중하고 있다. 현재 SmartSpace는 Sewio, Quuppa 등 서드파티 RTLS 하드웨어도 지원한다 [[19]](#ref-19).

KINEXON은 스포츠 분야의 초저지연(10ms 이하) 요구사항을 충족하는 것이 차별점이다. 50Hz 태그 갱신은 NFL/NBA 실시간 중계에 필수적인 스펙이다.

## 정리

Indoor Localization은 기술 자체보다 **적용 환경의 제약 조건**이 기술 선택을 결정한다. UWB는 정밀하지만 비싸고, BLE는 저렴하지만 정확도가 낮으며, IR+RFID Hybrid는 병원이라는 특수 환경에 최적화되어 있다.

벤더 생태계도 마찬가지다. Qorvo가 칩을 만들고, NXP가 자동차에 넣고, Apple/Samsung이 소비자 기기에 통합하고, Sewio/KINEXON이 산업 플랫폼으로 묶는다. 각자의 레이어에서 경쟁하면서도 802.15.4z, FiRa, CCC 같은 표준으로 연결되는 구조다.

## References

<span id="ref-1">1.</span> Sewio, "VELUX Customer Project," [Online]. Available: https://www.sewio.net/customer-projects/velux/

<span id="ref-2">2.</span> Ubisense, "RTLS in Aerospace & Automotive," [Online]. Available: https://ubisense.com/rtls-aerospace-automotive/

<span id="ref-3">3.</span> Siemens, "Automotive RTLS Automation Safety — Volkswagen Autoeuropa," 2022. [Online]. Available: https://www.siemens.com/global/en/company/stories/industry/2022/automotive-rtls-automation-safety-volkswagen-autoeuropa-portugal.html

<span id="ref-4">4.</span> BMW Group / KINEXON, Press Release, 2022. [Online]. Available: https://www.press.bmwgroup.com/global/photo/detail/P90460305/

<span id="ref-5">5.</span> Sewio, "Toyota Motors Customer Project," [Online]. Available: https://www.sewio.net/customer-projects/toyota-motors/

<span id="ref-6">6.</span> Navigine, "Case Study: Kesko," [Online]. Available: https://navigine.com/blog/case-for-kesko/

<span id="ref-7">7.</span> Kontakt.io, "Beacons in Retail: Use Cases," [Online]. Available: https://kontakt.io/blog/beacons-in-retail-use-cases/

<span id="ref-8">8.</span> Healthcare IT News, "Texas Health Saves $412,000 with Real-Time Location System," 2018. [Online]. Available: https://www.healthcareitnews.com/news/texas-health-saves-412000-real-time-location-system

<span id="ref-9">9.</span> HFM Magazine, "RTLS Vendors Improve Health Facility Efficiency," [Online]. Available: https://www.hfmmagazine.com/articles/4605-rtls-vendors-improve-health-facility-efficiency

<span id="ref-10">10.</span> CenTrak, "Hand Hygiene Compliance Monitoring," [Online]. Available: https://centrak.com/solutions/infection-control/hand-hygiene-compliance-monitoring

<span id="ref-11">11.</span> Zebra Technologies, "Zebra & The NFL," [Online]. Available: https://www.zebra.com/us/en/nfl.html

<span id="ref-12">12.</span> KINEXON Sports, "Basketball Tracking," [Online]. Available: https://kinexon-sports.com/sports/basketball/

<span id="ref-13">13.</span> Adidas, "Adidas Reveals the First FIFA World Cup Official Match Ball Featuring Connected Ball Technology," 2022. [Online]. Available: https://news.adidas.com/football/adidas-reveals-the-first-fifa-world-cup--official-match-ball-featuring-connected-ball-technology/

<span id="ref-14">14.</span> Qorvo, "DW3000 Product Brief," [Online]. Available: https://www.qorvo.com/products/p/DW3000

<span id="ref-15">15.</span> Apple, "Apple announces iPhone 15 lineup," Sep. 2023. [Online]. Available: https://www.apple.com/newsroom/2023/09/apple-introduces-iphone-15-and-iphone-15-plus/

<span id="ref-16">16.</span> Samsung, "Exynos Connect U100 UWB," [Online]. Available: https://semiconductor.samsung.com/connectivity/uwb/

<span id="ref-17">17.</span> NXP Semiconductors, "Trimension UWB," [Online]. Available: https://www.nxp.com/products/wireless-connectivity/uwb:UWB

<span id="ref-18">18.</span> Nordic Semiconductor, "nRF52833 — Direction Finding," [Online]. Available: https://www.nordicsemi.com/Products/nRF52833

<span id="ref-19">19.</span> Ubisense, "SmartSpace Platform," [Online]. Available: https://ubisense.com/smartspace/
