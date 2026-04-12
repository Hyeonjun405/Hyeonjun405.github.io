---
title: (Sparta_WIL_05) 캐싱전략
date: 2026-04-12 10:00:00 +09:00
categories: [Sparta, SpartaWIL11]
tags: [ Java, Spring Framework ]
---


## 1. Log
### 1. Keyword
- JPA / Optional
  - JPA에서 엔티티 설정하는 부분 복습하고
  - Optional로 어떻게 사용하는지 파악하고 
  - 영속성에 대해서 공부하고
  - QueryDSL에 대해서 접근하면 될 듯
- 계속 기반을 건너뛰고 용도만 접근하려고 하니 어려움.  
  
## 2. 문제의 기록
- 엔드포인트에 대한 고민
  - URL은 어떤 기준으로 잡는게 좋을까 ?
  - 리스폰에서 애매한데 뭐는 뭘로 잡고 뭐는 뭘로하고?  
- Optional과 JPA 활용에 대한 고민
  - 서비스단에서 JPA로 조회하는 부분은 조회만 할 것.
  - JPA로 조회하고 튀어나온 Optional에서 조회된걸로 뭔가를 하는건 애매한듯
  - 딱딱 역할에 맞게만 정의할 것
- JPA에서 외래키가 잡혀있을때,
  - 가운데 엔티티에서 위아래 영속성을 끊어버리면
  - 그 엔티티는 붕떠서 사라짐!

