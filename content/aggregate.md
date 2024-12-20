### 애그리거트 단위에 대한 고민

도메인 주도 설계(DDD)에서 애그리거트는 "변경 단위"이자 "트랜잭션 단위"라는 개념을 강의와 책을 통해 이해했습니다. 또한, 애그리거트를 구성하는 도메인 엔티티와 값 객체(Value Object)는 **생성 시점과 소멸 시점**, 즉 **생명주기**에 따라 구분된다는 원칙에도 동의했습니다.

하지만 이번 경험을 통해 새로운 관점을 얻게 되었습니다.

---

### **상황: `User` 관련 애그리거트 설계**

프로젝트에서 사용자 정보를 관리하기 위해 아래와 같은 세 가지 객체를 설계했습니다.

1. **User**: 기본 사용자 정보를 담고 있는 객체
2. **UserDetail**: 추가 정보를 담고 있는 객체
3. **UserProfile**: 선택 정보를 담고 있는 객체

이 세 객체는 **입력 및 저장 시점이 서로 다르다**는 점에서 **독립적인 애그리거트**로 설계했습니다. 예를 들어:

- 사용자가 회원가입을 완료하면 **User**가 생성됩니다.
- 추가 정보를 입력할 때 **UserDetail**이 생성됩니다.
- 프로필 설정을 완료하면 **UserProfile**이 저장됩니다.

이처럼 객체 간 **생명주기와 입력 시점이 다르다**는 점에 주목해 각각을 분리된 엔티티로 관리한 것은 합리적인 선택처럼 보였습니다.

### **문제 발견 1: 삭제 시 종속성**

설계 과정에서 미처 고려하지 못했던 점은 **소멸 시의 종속성**입니다.

- **User**가 삭제될 경우, **UserDetail**과 **UserProfile**은 단독으로 존재할 수 없습니다.
- 즉, **User**가 없는 상태에서 **UserDetail**과 **UserProfile**은 그 의미를 잃게 됩니다.

이러한 점에서 **UserDetail**과 **UserProfile**은 본질적으로 **User**에 강하게 종속되어 있으며, 사실상 **User 애그리거트의 일부**로 보는 것이 더 적합하다는 결론에 이르렀습니다.

### **문제 발견 2: 조회와 비즈니스 로직의 복잡성 증가**

두 번째 문제는 **애그리거트 간 분리를 기반으로 한 데이터 조회 및 비즈니스 로직의 복잡성 증가**였습니다.

user는 userDetailId와 userProfileId를 간접참고하고 있었습니다.

- 비즈니스 요구사항 상, `User`, `UserDetail`, `UserProfile` 정보를 동시에 조회해야 하는 경우가 많았습니다.
- 또한, 세 애그리거트가 함께 사용되는 비즈니스 로직에서 **데이터 일관성을 유지하기 위한 작업**이 필요해졌고, 이는 시스템 복잡성을 증가시켰습니다.

결과적으로, 이러한 분리 설계는 **유지보수성과 성능**에 부정적인 영향을 미쳤습니다.

### **교훈과 개선 방향**

1. **애그리거트의 경계를 종속성과 비즈니스 로직 중심으로 재정의**
    - 단순히 **생명주기나 생성 시점**만을 기준으로 경계를 나누는 것은 불완전할 수 있습니다.
    - **소멸 시 종속성**과 **데이터 일관성을 요구하는 비즈니스 로직**을 고려해야 합니다.
    - 이 관점에서 `User`, `UserDetail`, `UserProfile`은 하나의 애그리거트로 묶는 것이 더 적합합니다.
2. **조회 및 사용 시나리오를 설계 초기 단계에서 반영**
    - 프로젝트 초기에 데이터 조회 및 사용 시나리오를 명확히 정의하고, 이를 애그리거트 설계에 반영해야 합니다.
    - 예를 들어, 사용자 정보를 한 번에 가져오는 조회 로직이 빈번하다면, 이를 하나의 애그리거트로 묶는 것이 더 효율적입니다.
3. **생성 시점이 다른 객체를 한 애그리거트에서 관리하는 방법 탐색**
    - 생명주기가 다르더라도, 도메인 규칙에 따라 논리적으로 연관된 객체를 한 애그리거트에 포함시킬 수 있습니다.
    - 예를 들어, `User` 애그리거트에서 `UserDetail`과 `UserProfile`을 지연 로딩하거나, 부분적으로 생성된 상태를 허용할 수 있습니다.

이 경험은 애그리거트를 설계할 때 단순히 **변경 단위**나 **생명주기** 뿐만 아니라, **소멸 시 종속성** 과 **비즈니스 요구사항** 까지도 반드시 고려해야 한다는 중요한 교훈을 주었습니다.
앞으로는 이러한 점을 반영해 더 실용적이고 일관성 있는 도메인 모델을 설계할 수 있을 것입니다.