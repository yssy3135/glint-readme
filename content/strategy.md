## 1. 문제 분석
회원 가입 시 사용자가 입력하는 다양한 프로필 정보나 미팅에 참가 시 선택된 조건에 대해 검증을 해야 합니다. 이 검증 로직을 한 군데에서 관리하기보다는 전략 패턴을 사용하여 각 조건을 개별적으로 처리하고, 조건의 추가 및 삭제가 용이하도록 설계하려고 합니다.

## 2. 전략 패턴 설계
전략 패턴(Strategy Pattern) 은 알고리즘을 클래스에서 분리하여 캡슐화하고, 클라이언트에서 쉽게 교체할 수 있게 하는 패턴입니다. 이를 통해 검증 로직을 전략으로 분리하고, 각 검증 항목을 독립적인 클래스로 처리할 수 있습니다.
## 3. 설계 흐름
#### (1) ConditionValidator 인터페이스 정의

```
public interface ConditionValidator {
boolean validate(User user);  // 검증 조건에 맞는지 확인하는 메소드
}
```

#### (2) 각 검증 로직 구현
다음으로, 구체적인 검증 조건을 구현한 클래스들을 작성합니다. 예를 들어, AgeValidator, ProfileCompleteValidator 등 사용자가 입력한 정보에 대해 각각의 조건을 검증하는 클래스를 구현합니다.

```
public class AgeValidator implements ConditionValidator {
private static final int MIN_AGE = 18;

    @Override
    public boolean validate(User user) {
        return user.getAge() >= MIN_AGE;
    }
}

public class ProfileCompleteValidator implements ConditionValidator {

    @Override
    public boolean validate(User user) {
        return user.isProfileComplete();
    }
}
```

#### (3) UserMeetingValidationHandler 클래스
UserMeetingValidationHandler는 여러 조건에 따라 검증을 수행하는 클래스입니다. 여기서 조건에 맞는 검증기(ConditionValidator)들을 Map에 저장하고, 조건을 검증합니다.


```
import java.util.Map;

public class UserMeetingValidationHandler {
private Map<String, ConditionValidator> validators;

    public UserMeetingValidationHandler(Map<String, ConditionValidator> validators) {
        this.validators = validators;
    }

    public boolean validate(User user) {
        for (Map.Entry<String, ConditionValidator> entry : validators.entrySet()) {
            if (!entry.getValue().validate(user)) {
                System.out.println(entry.getKey() + " 조건 불합격");
                return false;  // 하나라도 실패하면 검증 실패
            }
        }
        return true;  // 모든 조건을 통과한 경우
    }
}
```
#### (4) User 객체
검증을 수행할 대상이 되는 User 클래스는 사용자의 정보를 포함합니다. 이 클래스는 검증 로직에서 필요한 데이터를 제공합니다.
```
public class User {
private int age;
private boolean profileComplete;

    public int getAge() {
        return age;
    }

    public boolean isProfileComplete() {
        return profileComplete;
    }
}
```
#### (5) 검증 실행
마지막으로, 검증 로직을 실행하는 부분입니다. UserMeetingValidationHandler에 각 조건별로 검증기(ConditionValidator)들을 넣고, validate 메소드를 호출해 검증을 실행합니다.

```
import java.util.HashMap;
import java.util.Map;

public class Main {
public static void main(String[] args) {
// 조건별 검증기 생성
Map<String, ConditionValidator> validators = new HashMap<>();
validators.put("Age", new AgeValidator());
validators.put("Profile Completion", new ProfileCompleteValidator());

        // 검증 처리 클래스 생성
        UserMeetingValidationHandler validationHandler = new UserMeetingValidationHandler(validators);

        // 검증할 사용자 정보
        User user = new User(20, true);

        // 검증 실행
        boolean isValid = validationHandler.validate(user);
        System.out.println("검증 결과: " + (isValid ? "성공" : "실패"));
    }
}
```
## 4. 확장성 & 유지보수
조건 추가: 새로운 검증 조건을 추가하고 싶을 때, ConditionValidator 인터페이스를 구현한 새로운 클래스를 작성하여 validators 맵에 추가하면 됩니다.
조건 수정: 기존 조건을 수정하고 싶을 때는 해당 조건을 구현한 클래스를 수정하면 됩니다.
조건 삭제: 더 이상 필요한 조건이 없다면, 해당 검증 클래스를 삭제하고 validators 맵에서 제거하면 됩니다.

## 5. 장점
유연성: 새로운 검증 조건을 추가하는 데 있어 기존 코드를 수정하지 않고 새로운 전략만 추가하면 됩니다.
가독성: 여러 조건을 if 또는 switch로 처리하는 것보다 각 검증 로직을 독립된 클래스에서 처리하므로 코드가 간결하고 명확합니다.
유지보수 용이: 검증 로직이 클래스별로 분리되어 있기 때문에 수정이 용이하고, 특정 조건만을 독립적으로 수정할 수 있습니다.