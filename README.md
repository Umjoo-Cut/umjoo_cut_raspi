

# Hardware & System Architecture

본 시스템은 STM32F411RE를 중심으로 센서 데이터 수집, 시스템 상태 제어, Raspberry Pi 통신 및 차량 시동 제어 기능을 수행한다.

---

# PINMAP

## STM
<img width="1014" height="775" alt="image" src="https://github.com/user-attachments/assets/87fc4add-7ff4-4eb5-b279-baf40f17c223" />
<img width="376" height="330" alt="image" src="https://github.com/user-attachments/assets/988f76b1-a890-485d-a979-8dcab23a426d" />

# RASPI

<img width="767" height="478" alt="image" src="https://github.com/user-attachments/assets/3ab9fa4b-673a-4e3d-82d0-ec6a8905c159" />


# Main Controller

## STM32F411RE

### 역할
- 시스템 전체 동작을 제어하는 메인 MCU

### 기능
- 센서 데이터 수집 및 처리
- 상태 머신(State Machine) 실행
- Raspberry Pi와 UART 통신 수행
- LED 및 부저 제어
- 시스템 상태 관리

---

# System State Definition

시스템은 상태 머신(State Machine) 기반으로 동작하며 각 상태에 따라 제어 로직을 수행한다.

| 상태 | 설명 |
|--------|------|
| STATE_IDLE | 시스템 초기 대기 상태 |
| STATE_WAIT_SEAT | 사용자 착석 여부 확인 |
| STATE_SEAT_LOST_WAIT | 측정 중 자리 이탈 감지 |
| STATE_WAIT_BLOW | 사용자 숨 입력 대기 |
| STATE_RETRY_WAIT | 재측정 요청 후 대기 |
| STATE_MEASURING | 센서 측정 진행 |
| STATE_WAIT_RESULT | Raspberry Pi 결과 대기 |
| STATE_PASS | 측정 성공 및 시동 허용 |
| STATE_FAIL | 측정 실패 및 시동 차단 |

---

# System Flow

```text
STATE_IDLE
      ↓
STATE_WAIT_SEAT
      ↓
STATE_WAIT_BLOW
      ↓
STATE_MEASURING
      ↓
STATE_WAIT_RESULT
      ↓
STATE_PASS / STATE_FAIL
```

### 예외 상황

- 사용자 자리 이탈

```text
STATE_MEASURING
      ↓
STATE_SEAT_LOST_WAIT
```

- 측정 실패 또는 재측정 요청

```text
STATE_FAIL
      ↓
STATE_RETRY_WAIT
```

---

# MQ-3 Alcohol Sensor

### 역할
- 사용자의 호흡을 통해 알코올 농도를 측정

### 기능

- ADC를 이용한 센서 값 측정
- 음주 수치 데이터 100ms에 한번 측정 총 5초 동안 50번 제공
- 측정 결과 처리

---

# DHT11 Temperature & Humidity Sensor

### 역할
- 측정기에 숨을 불었는지 입김에 의한 습도변화로 감지

### 기능

- 습도 수치 데이터 5초 동안 총 5번 제공
- 측정 결과 처리 

---

# Status LED

### Yellow LED

**역할**
- 측정 진행 및 대기 상태 표시

### Green LED

**역할**
- 측정 성공 상태 표시

### Red LED

**역할**
- 실패 및 경고 상태 표시

### Engine LED

**역할**
- 시동 허용 여부 표시

---

# Buzzer

### 역할

- 사용자에게 소리 알림 제공

### 기능

- 시스템 상태 알림
- 실패 상태 경고음 출력
- 재측정 요청 알림

---

# UART Communication

STM32와 Raspberry Pi는 UART 통신을 이용하여 시스템 상태 및 처리 결과를 문자열 형태로 송수신한다.

---

## STM32 → Raspberry Pi

STM32는 시스템 상태 및 센서 진행 정보를 Raspberry Pi로 전송한다.

### 전송 메시지

| Message | 설명 |
|----------|------|
| SYSTEM_START | 시스템 시작 |
| SEAT_ON | 사용자 착석 감지 |
| SEAT_OFF | 사용자 자리 이탈 감지 |
| BLOW_START | 숨 입력 시작 |
| MEASURE_BEGIN | 측정 시작 |
| MEASURE_END | 측정 종료 |
| DHT_ERROR | DHT11 센서 오류 발생 |

### 기능

- 시스템 상태 전달
- 사용자 착석 여부 전달
- 측정 시작 및 종료 상태 전달
- 센서 오류 상태 전달

---

## Raspberry Pi → STM32

Raspberry Pi는 처리 결과 및 제어 명령을 STM32로 전달한다.

### 수신 메시지

| Message | 설명 |
|----------|------|
| PASS | 인증 성공 및 시동 허용 |
| FAIL | 인증 실패 및 시동 차단 |
| ERROR | 시스템 처리 오류 |
| RETRY | 재측정 요청 |
| OTHER | 기타 예외 상황 |

### 기능

- 처리 결과 전달
- 재측정 요청 처리
- 오류 및 예외 상태 전달

---

## Communication Information

| 항목 | 내용 |
|------|------|
| 통신 방식 | UART (USART) |
| 데이터 형식 | String |
| 통신 대상 | Raspberry Pi ↔ STM32 |
