---
layout: post
title: 라즈베리파이 제로의 낮은 전력 소모
lang: kr
tags:
- raspberry pi
published: true
---

정확한 전력 소모량을 측정할 장비는 없지만,

다음의 세 주변기기로 테스트 해보았다:

- WiFi 동글,
- HAT,
- 카메라 모듈

나는 싸구려 전원 어댑터로도 이 세 주변기기가 모두 잘 작동하는지 확인하고 싶었다.

----

## A. 테스트한 주변기기

### 1. WiFi 동글

AirLink101 사의 USB WiFi 동글. [이놈](http://airlink101.com/products/awll5099.php)이다.

라즈베리 파이 1B 및 2B에서 2A 짜리 전원 어댑터를 끼우면 보통 잘 동작하던 녀석이다.

### 2. HAT

Pimoroni의 [Scroll pHat](https://shop.pimoroni.com/collections/raspberry-pi/products/scroll-phat).

### 3. 카메라 모듈

v1.3[(구형)](https://www.raspberrypi.org/products/camera-module/)을 사용했다.

라즈베리 파이 1B 및 2B에서는 외부 전원이 포함된 USB 허브에서도

WiFi가 연결된 상태에서는 종종 죽곤 했던걸 보니 꽤 전력을 많이 먹는 듯 했다.

### 4. 파워 어댑터

어댑터의 뒤에는 이렇게 적혀있다:

```
INPUT:  AC 100-240V
        50-60Hz 0.15A
OUTPUT: DC 5.0v 700mA
```

즉 0.7A 출력인데, [공식 파워 어댑터](https://www.raspberrypi.org/products/universal-power-supply/)의 2.0A보단 훨씬 작은 값이다.

이 어댑터로 라즈베리 파이 1B 모델에서 WiFi 동글을 쓰려면 추가로 외부 전원 USB 허브가 필요했고,

반면 1B+ 및 2B 모델에서는 이 어댑터만으로도 잘 구동이 됐다.

## B. 라즈베리 파이 제로에서 테스트한 방법

![설정](https://cloud.githubusercontent.com/assets/185988/16071555/feb944ae-3316-11e6-836a-e574fec0d7df.jpg)

WiFi에 연결된 상태에서, `raspistill`을 반복적으로 실행해 사진을 찍었고

Scroll pHat의 LED를 껐다 켰다를 반복했다.

## C. 테스트 결과

이렇게 테스트 해도 라즈베리 파이 제로는 응답 없는 상태에 빠지지 않고 계속 살아있었다.

테스트 중에 여러가지 명령어를 동시에 실행해 보기도 했으나, 달라지는건 없었다.

## D. 결론

이게 정확한 테스트는 아닌지라 보장할 수 있는건 없지만

최소한, 라즈베리 파이 제로가 WiFi 동글과 HAT, 카메라 모듈이 작동하는 상태에서도

보통 수준의 부하는 0.7A 파워로 버텨내는 듯 싶다.

라즈베리 파이 제로는 형님 모델들에 비해 적은 전력을 소모하는 것으로 광고가 되곤 했는데,

어느 정도는 그 증명이 되는 셈이다.

----

## 마무리

라즈베리 파이 제로엔 LAN이나 USB 포트 등이 없어 아쉽지만

작은 사이즈와 낮은 전력 소모는 그 단점을 상쇄하고도 남는다!

