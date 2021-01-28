# 2. Kotlin, Spring, Gradle

Kotlin을 스터디하면서 사용해보고 싶었기 때문에 해당 프로젝트를 Kotlin으로 구현하기로 했으며 Java와 호환이 잘되는 Kotlin의 장점을 이용해 Java의 대표적인 프레임워크인 Spring을 함께 사용해서 구현을 하기로 했다.

또한 Gradle의 기반 언어로 Groovy 대신 Kotlin을 사용해보기로 했다. 구글링을 해봤을 때 Groovy에 비해 레퍼런스가 좀 적은 느낌도 있지만 그래도 이왕 Kotlin 스터디이기에 한번 해보기로 했다.

## 2.1. IntelliJ CE와 Spring

우선 IntelliJ Ultimate 버전과 Community 버전에서 벽을 느꼈다. STS(Spring Tool Suite)와 같이 Spring 프로젝트를 알아서 생성해주는 Ultimate 버전과 Community의 차이에서 살짝 박탈감을 느끼긴 했으나 불편하고 어려운 개발은 좀 더 기억에 잘 남기 때문에 *오히려 좋아*라는 마인드로 임했다.

왜냐하면 Spring도 Spring Boot를 써서 개발환경을 구성하면 Kotlin으로 만드는 Spring Boot 프로젝트는 구글링하기도 편하고 [Spring Initializer](https://start.spring.io/)에서 단순히 클릭만으로 쉽고 편하게 만들어도 되지만 굳이 검색도 안되는 Spring, Kotlin, Gradle 조합을 고집하고 있었기 때문에 위와 같은 상황도 즐길 수(?) 있을 것 같다.

