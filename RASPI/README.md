# 얼굴 인증 기반 음주운전 방지 시스템

## 📌 프로젝트 소개

본 프로젝트는 Raspberry Pi 5와 STM32를 활용한  
얼굴 인증 기반 음주운전 방지 시스템입니다.

단순 음주 측정을 넘어,

- 등록 운전자 얼굴 인증
- 실제 호흡 감지
- 본인 재검증
- 운전자 교체 감지
- 대리 측정 방지

기능을 통합하여 실제 차량 환경에서 발생 가능한 우회 상황을 차단하는 것을 목표로 합니다.

---

# 🧠 시스템 핵심 기능

## ✅ 등록 운전자 인증

- buffalo_sc 기반 얼굴 임베딩 생성
- 등록 운전자 DB 생성
- 등록 / 미등록 운전자 판별
- 512차원 Embedding 기반 비교

---

## ✅ 세션 기반 운전자 추적

초기 얼굴(face_A) 촬영:

- 얼굴 3장 촬영
- 평균 Embedding 생성
- session_embedding.npy 생성
- 세션 운전자 임시 등록

---

## ✅ 본인 재검증 시스템

시동 직전 얼굴(face_B) 재촬영:

- 얼굴 3장 재촬영
- 평균 Embedding 생성
- session_embedding ↔ face_B 비교
- 코사인 유사도 계산
- 운전자 교체 여부 판단

대리 측정 방지:

```text
face_A ≠ face_B

→ MSG_OTHER 전송
→ STM32 IDLE 복귀
```

---

## ✅ 얼굴 가림 / 정면 검증

- buffalo_sc 랜드마크 사용
- 눈 / 코 / 입 모두 확인
- 얼굴 일부 가림 감지
- 얼굴 크기 검사
- 너무 멀거나 화면 밖이면 재시도

예:

```text
NO FACE
눈, 코, 입이 모두 보이도록 정면을 봐주세요
```

---

## ✅ 입-MQ3 위치 유도

카메라 영상 기반:

- 입 위치 추적
- 입 ↔ MQ3 거리 계산
- READY 상태 진입
- 실시간 위치 안내

예:

```text
Move Mouth Dist:68
READY Dist:24
```

---

## ✅ MQ3 + DHT 음주 측정

측정 구조:

```text
1초 baseline 측정
(10개 샘플)

↓

4초 측정
(40개 샘플)
```

계산:

- MQ3 peak 계산
- MQ3 delta 계산
- DHT 습도 변화량 계산

최종 판정:

```text
습도 변화량 부족
→ RETRY

MQ3 delta ≥ threshold
→ FAIL

그 외
→ PASS
```

---

## ✅ 압력센서 기반 운전자 교체 감지

시동 후:

- 압력 센서 지속 체크
- 운전자 자리 이탈 감지
- 운전자 교체 차단

예:

```text
운전자 이탈
↓

재인증 요구
```

---

## ✅ 증거 기록 시스템

자동 저장:

- 음주 감지 이미지
- 얼굴 인증 실패 이미지
- 운전자 교체 시도 기록
- registered / unregistered 분리
- system.log 저장

---

# 🏗️ 시스템 구조

### STM32

```text
버튼 입력
MQ3 측정
DHT 측정
압력센서 측정
LED 제어
Buzzer 제어
Relay 제어
UART 통신
```

### Raspberry Pi

```text
얼굴 인증
등록자 판별
session 생성
본인 재검증
입 위치 추적
MQ3/DHT 분석
증거 저장
로그 저장
UART 처리
```

---

# 🔄 시스템 동작 흐름

```text
1. SYSTEM_START
2. SEAT_ON
3. face_A 촬영
4. 등록 운전자 판별
5. session_embedding 생성
6. 입-MQ3 위치 유도
7. BLOW_START
8. MQ3 / DHT 측정
9. face_B 재촬영
10. session ↔ face_B 비교
11. 최종 PASS / FAIL
12. 시동 후 압력센서 감시
```

---

# 📂 프로젝트 구조

```text
RASPI_BUFFALO_SC/

├── db/
│   ├── images/
│   │   └── registered_driver/
│   │
│   ├── known_embedding.npy
│   └── session_embedding.npy
│
├── logs/
│   ├── temp/
│   ├── registered/
│   ├── unregistered/
│   ├── identity_fail/
│   └── system.log
│
├── alcohol_judge.py
├── face_capture.py
├── mouth_position_checker.py
├── registered_driver.py
├── session_driver.py
├── uart_manager.py
├── file_manager.py
├── logger_manager.py
├── config.py
├── main.py
└── requirements.txt
```

---

# 🖥️ UART 프로토콜

### STM32 → Raspberry Pi

```text
SYSTEM_START
SEAT_ON
SEAT_OFF

BLOW_START
MEASURE_BEGIN

MQ3:1234
HUM:45

MEASURE_END
```

### Raspberry Pi → STM32

```text
PASS
FAIL
RETRY
ERROR
OTHER
```

---

# ⚙️ 사용 기술

## AI / Vision

- buffalo_sc
- InsightFace
- OpenCV
- NumPy

## Hardware

- Raspberry Pi 5
- STM32 Nucleo-F411RE
- MQ-3 Alcohol Sensor
- DHT Sensor
- Pressure Sensor
- Camera
- Relay
- Buzzer

## Communication

- UART Serial Communication