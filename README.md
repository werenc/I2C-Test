# MPU6500 I2C Data Fetch Time Test

Lolin S3 PRO에서 I2C를 이용해 MPU6500의 가속도, 온도, 자이로 데이터를 읽고,
하나의 데이터 요청에 걸리는 시간을 측정한 실험

## 1. 실험 목적
- Lolin S3 PRO과 MPU6500 간 I2C 통신 확인
- MPU6500의 센서 데이터 레지스터에서 14 bytes 연속 읽기
- 하나의 I2C 데이터 요청에 걸리는 시간 측정
- 향후 센서 샘플링 주기와 제어 루프 주기 설계에 필요한 기초 자료 확보

## 2. 실험 환경
| 항목 | 설정 |
|---|---|
| MCU board | Lolin S3 PRO |
| Sensor | MPU6500 |
| Communication | I2C |
| SDA pin | 8 |
| SCL pin | 9 |
| I2C clock | 400 kHz |
| MPU6500 address | `0x68` |
| Start register | `0x3B` |
| Read size | 14 bytes |
| Serial baud rate | 115200 |

## 3. 읽은 데이터 범위

MPU6500의 `0x3B` 레지스터부터 14 bytes를 연속으로 읽었다.

해당 데이터에는 다음 값이 포함된다.

| 데이터 | 크기 |
|---|---:|
| Accelerometer X, Y, Z | 6 bytes |
| Temperature | 2 bytes |
| Gyroscope X, Y, Z | 6 bytes |
| Total | 14 bytes |

## 4. 실험 과정

### 4.1 I2C 초기화

```cpp
Wire.begin(8, 9);
Wire.setClock(400000);
```

### 4.2 측정 시작
```cpp
uint32_t start = micros();
```

### 4.3 레지스터 지정
```cpp
Wire.beginTransmission(0x68);
Wire.write(0x3B);
Wire.endTransmission(false);
```
endTransmission(false)를 사용해 STOP condition을 발생시키지 않고, 이어서 repeated START 방식으로 읽기 요청을 수행했다.

### 4.4 14bytes 데이터 요청
```cpp
Wire.requestFrom(0x68, 14);
```
센서 데이터가 모두 도착할 때까지 대기한 뒤, 14바이트를 순서대로 읽었다.
```cpp
while (Wire.available() < 14);
for (int i = 0; i < 14; i++) wire.read();
```
### 4.5 소요 시간 측정
모든 데이터를 읽은 후 micros()를 호출했다.
```cpp
uint32_t end = micros();
```
전체 요청 시간은 다음과 같이 계산하였다.
```text
end-start
```

### 4.6 결과
1회의 요청에 650us가 소요됨을 확인할 수 있었다.
