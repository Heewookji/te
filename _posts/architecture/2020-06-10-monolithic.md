---
title: monolithic architecture
---

# 개발

그림

개발자 - 로컬 tomcat - DB - github 에서 작업

# 배포

그림

SCP 통해 Clean 배포(stop -> delivery -> start)한 뒤
DNS 따서 tomcat url로 넣어준다.

> 무중단 배포가 불가능한 문제점이 있다.

그림

따라서 톰캣을 두 대로 둔 뒤, Load Balancing을 통해 프로덕션하도록
한다. 이러한 HA구성(High Availability: 고 가용성)은 다운타임이
발생하지않는다. 

그림

서비스 사용자가 더 많아질 경우에는 더 많은  tomcat을 넣고,
배포를 더 빠르게 할 수  있도록 방식을 변경해야한다.

그림

이 후 개발팀이 나뉘어지게 되는 경우
1. 브랜치 머지 컨플릭트
2. QA를 위한 정기배포 일정 조율
등의 여러가지 문제점과 만나게 된다.

그림

아예 도메인을 나누고 프로덕션의 서버도 나누어 관리하는 경우
1. 공통 코드를 위하여 share되는 jar 파일을 가지게 된다.
2. 여전히 정기 배포 일정 조율이 필요(share되는 jar 파일)
3. share되는 jar의 코드를 바꿀 수가 없다.

>콘웨이의 법칙(Conway's Law)
>조직은 조직의 의사소통 구조와 똑같은 구조를 갖는 시스템을 설계한다

## monolithic architecture의 장점
1. 개발이 단순하다. (repo 하나 체크아웃)
2. 배포가 단순하다. (war 하나 배포)
3. Scale-out이 단순하다. (서버 하나 복사)

## monolithic architecture의 단점
1. 무겁다.
2. 기술 스택 변경의 어려움
3. 높은 결합도
4. 코드의 책임 한계


[참고 동영상](https://www.youtube.com/watch?v=D6drzNZWs-Y)
