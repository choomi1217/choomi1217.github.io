---
title:  "[개발자 취업] Java의 가비지 컬렉션(Garbage Collection) 동작 메커니즘에 대한 이해를 설명해주세요"

categories:
  - get-a-job
tags:
  - [get-a-job]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-10-25
last_modified_at: 2023-10-25
---

**제 경험이 조금씩 첨가된 주관적인 답변이므로 참고만 하세요! 답이 아닙니다! 여러분의 의견을 존중합니다!**

### Java의 가비지 컬렉션(Garbage Collection) 동작 메커니즘에 대한 이해를 설명해주세요

JVM의 Heap영역에서 사용하지 않는 객체를 삭제하는 일을 gc가 합니다.
GC는 Heap영역에서 GC Root(Stack영역, 메소드영역, Heap의 영역에 있는 다른 객체도 포함 등)가 닿을수 있는(참조된) 객체와 닿을 수 없는(참조가 없는) 객체를 탐색합니다.
이런 닿을 수 없는 객체가 수거 대상이고 Mark And Sweep 방식으로 수거합니다.
참조되는 객체는 Marking을하고 닿을 수 없는 객체는 Sweep을 합니다. 조각난 Heap의 메모리를 Compact로 모아줄 때도 있습니다.
Heap 자체가 Young과 Old Generation, metaspace로 나뉘어 있습니다.
이중 young generation은 eden과 survivor0,1로 구성되었는데 에덴이 꽉차면 minor gc가 닿은 수 없는 객체를 survivor0으로 이동시키는데
0과 1은 하나는 꼭 비어있어야만 합니다. 살아남은 애들은 age가 올라갑니다. 또 에덴이 차고 minor gc가 다시 수거하고 살아남은 객체는 1로 이동합니다. 이때 또 객체의 age가 올라갑니다.
age 임계점이 있는데 이 임계점에 도달하면 old generation으로 이동합니다. old generation이 꽉 차면 major gc가 발생합니다.
이제 gc 발생 시마다 stop the world가 발생하는데 이게 gc 종류마다 시간이 좀 다릅니다.
지금까지 gc 종류로 크게 serial, parallel, g1, z 가 나왔는데 이중에 parallel이 8의 gc기 때문에 parallel은 young generation 에 일어나는 minor gc에 한해서 병렬수행합니다.
그러므로 stop the world 시간이 직렬 gc 보단 적습니다. 그러고 g1gc는 heap 영역을 young,old로 나누지 않고 region이란 영역으로 나누고 이중에 garbage 마크가 있는 영역만 수거해갑니다.

### 널널한 개발자의 강의를 보고 난 후의 가비지 컬렉터와 작동원리... 과연 그것을 물은 것일까? 영상을 보고..
JVM Heap Dump 분석 해봤어? 장애 대응 해봤어? 이걸 물은걸 수도 있다.
JVM Memory Leak에 대한 이야기를 봐야함.
서비스 지연 -> 장애 -> 대응해야함. 결국 시니어는 장애 상황에서 대응 할 줄 알아야 한다.
