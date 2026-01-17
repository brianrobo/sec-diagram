sequenceDiagram
    autonumber
    participant TS as Trust Server
    participant UE as UE (User Equipment)
    participant FBS as Suspicious gNB/eNB
    participant LegitBS as Legitimate gNB/eNB
    participant CN as Core Network (AMF/MME)

    rect rgb(250, 250, 250)
    Note over UE,TS: E-1. Extensible Framework for New Threats

    Note over UE: 새로운 공격 패턴 추가 시:<br/>1. 패턴 정의 (워크플로우)<br/>2. 점수 증감 값 설정<br/>3. Evidence 필드 정의<br/>4. Trust Server에 보고

    Note over TS: Trust Server는 새로운 threat_type을<br/>동적으로 처리 가능

    Note over UE,TS: 확장 가능한 아키텍처로<br/>미래의 위협에 대응
    end



    Scenario E는 확장 가능한 프레임워크를 나타내는 개념적 시나리오로, 실제 공격 플로우보다는 새로운 위협 유형을 추가할 수 있는 시스템의 확장성을 강조합니다. 미래에 발견될 새로운 공격 패턴도 동일한 구조(패턴 정의 → 점수 설정 → Evidence 정의 → Trust Server 보고)로 처리할 수 있음을 보여줍니다.
