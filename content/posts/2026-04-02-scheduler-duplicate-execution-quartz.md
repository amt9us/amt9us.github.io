---
title: "이중화 환경에서 Scheduler 중복 실행 문제 해결 (Quartz)"
date: 2026-04-02T20:42:30+09:00
description: "Spring Boot 이중화 환경에서 @Scheduled 스케줄러가 중복 실행되는 문제와 Quartz Scheduler를 사용해 해결한 경험을 정리합니다."
categories:
  - Backend
tags:
  - Spring Boot
  - Scheduler
  - Quartz
draft: false
---

서비스를 운영하다 보면 **예약 메시지 발송, 통계 집계, 데이터 Sync** 등 특정 시간에 자동으로 실행되어야 하는 작업이 필요합니다.

Spring Boot에서는 보통 `@Scheduled`를 사용해 간단하게 스케줄러를 구현할 수 있습니다.

```java
@Scheduled(cron = "0 0/1 * * * *")
public void sendMessages() {
  // 예약 메시지 발송
}
```

하지만 서비스가 이중화(멀티 인스턴스) 환경이라면 예상하지 못한 문제가 발생할 수 있습니다.

## 1. 이중화 환경에서 발생하는 스케쥴러 중복 실행 문제

운영 환경에서는 보통 다음과 같은 구조를 사용합니다.

```text
       [Load Balancer]
              |
       +------+------+
       |             |
   [Server A]    [Server B]
```

이때 `@Scheduled`는 각 서버에서 독립적으로 실행됩니다.

즉 다음과 같은 상황이 발생합니다.

```
Server A → 스케줄 실행
Server B → 스케줄 실행
```

결과적으로 같은 작업이 2번 실행됩니다.

이 때문에 이중화 환경에서는 단순 스케줄러 사용이 위험할 수 있습니다.

## 2. Quartz Scheduler를 사용하는 이유

이 문제를 해결하기 위해 Quartz Scheduler를 사용하게 되었습니다.  
Quartz는 단순한 스케줄러가 아니라 분산 환경을 고려한 Job Scheduler 입니다.

### 주요 특징
- DB 기반 Job 관리
- 클러스터 환경 지원
- Job 상태 관리
- 동시 실행 제어

특히 Job 상태를 DB에 저장하고 관리한다는 점이 중요한데, 여러 서버가 동일한 스케줄을 공유하더라도 동시에 실행되지 않도록 제어할 수 있습니다.

또한 예약 작업이 DB 데이터를 기반으로 동작하는 경우가 많기 때문에 스케줄 상태를 DB에서 관리하는 구조가 운영 측면에서도 안정적이라고 판단했습니다.

## 3. Quartz가 중복 실행을 막는 방식

Quartz는 스케줄 정보를 DB에 저장하고 관리합니다.

구조는 다음과 같습니다.

```
Spring Boot Server A
        ▼
      Quartz
        ▼
     Quartz DB
        ▲
Spring Boot Server B
``` 

### Job 실행 과정

1. Server A가 Job 실행 시도
2. Quartz DB에서 Lock 획득
3. Server A 실행
4. Server B 실행 시도
5. DB Lock 실패
6. 실행하지 않음

> 즉 여러 서버가 있더라도 Job은 한 번만 실행됩니다.

## 4. Quartz 사용 시 발생할 수 있는 또 다른 충돌

Quartz를 사용해도 환경 구성이 잘못되면 충돌이 발생할 수 있습니다.

대표적인 사례가 다음입니다.

- 로컬 / 개발 / 운영이 같은 Quartz DB를 바라보는 경우

이 세 환경이 같은 Quartz DB를 사용하면 아래와 같은 문제가 발생할 수 있습니다.

```
Local 서버 → 스케줄 실행
Dev 서버 → 스케줄 실행
Prod 서버 → 스케줄 실행
```

심지어 개발 서버가 운영 스케줄을 실행하는 상황도 생길 수 있습니다.  
운영 환경에서는 이런 문제도 실제 장애로 이어질 수 있기 때문에 환경 구성을 명확하게 분리하는 것이 중요합니다.

##  5. 방지하기 위한 방법

### 1. 환경별 Quartz DB 분리

가장 안전한 방법입니다.

```
local quartz DB
dev quartz DB
prod quartz DB
```

각 환경이 서로 영향을 주지 않습니다.

### 2. Quartz Cluster 설정

Quartz는 클러스터 모드를 지원합니다.

```md
# 대표 설정
org.quartz.jobStore.isClustered = true

```
이 옵션을 활성화하면

- 여러 서버가 동일한 Quartz DB 사용
- Job 실행은 하나의 서버만 수행

### 3. Scheduler 실행 환경 제한

운영 서버에서만 스케줄러가 실행되도록 제한하기도 합니다.

예를 들어 `Spring Profile`을 사용하면 특정 환경에서만 스케줄러가 동작하도록 설정할 수 있습니다.

```java
@Profile("prod")
@Component
public class MessageScheduler {
}
```
이렇게 설정하면 prod 프로파일이 활성화된 환경에서만 스케줄러가 등록됩니다.

```
local → 스케줄 비활성
dev → 스케줄 비활성
prod → 스케줄 실행
```

이를 통해 로컬이나 개발 환경에서 의도치 않게 스케줄러가 실행되는 상황을 방지할 수 있습니다.

---

이중화 환경에서 스케줄러를 사용할 때는 반드시 중복 실행 문제를 고려해야 합니다.

현재는 스케줄 작업이 대부분 DB 기반 데이터 처리이기 때문에 Quartz를 사용해 스케줄을 관리하고 있지만,  
앞으로 또 다른 이슈나 요구사항이 생긴다면 각 방식들의 특징과 어떤 상황에서 적합한지도 따로 정리해볼 예정입니다.
