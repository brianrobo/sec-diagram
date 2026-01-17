sequenceDiagram
    autonumber
    participant TS as Trust Server
    participant UE as UE (User Equipment)
    participant FBS as Suspicious gNB/eNB
    participant LegitBS as Legitimate gNB/eNB
    participant CN as Core Network (AMF/MME)

    rect rgb(245, 245, 255)
    Note over UE,FBS: D-1. Bidding Down Attack Pattern (확장 예시)

    Note over UE: UE in 5G SA Mode<br/>Supports NR + LTE + UMTS

    FBS->>UE: RRC Release with Redirection<br/>(Target: 4G LTE, Cause: Load Balancing)

    Note over UE: ⚠️ Suspicious Redirection<br/>5G signal is strong, but forced to 4G

    UE->>FBS: Re-attach in LTE mode
    FBS->>UE: Security Mode Command<br/>(LTE with weaker encryption)

    Note over UE: ⚠️ Bidding Down Detected!<br/>Pattern: 5G → 4G without valid reason

    UE->>UE: Calculate Untrust Score Increment<br/>BIDDING_DOWN_SCORE = +20

    UE->>TS: THREAT_REPORT<br/>{<br/>"cell_id": "450-05-gNB6666-Cell3",<br/>"threat_type": "BIDDING_DOWN",<br/>"report_score": 30,<br/>"evidence": {<br/>  "forced_rat_change": "5G_to_4G",<br/>  "nr_signal_strength": "STRONG",<br/>  "redirection_cause": "Load_Balancing",<br/>  "suspicious": true<br/>}<br/>}

    TS->>TS: EMA Calculation (α=0.4)<br/>Current: 10.5, Report: 30<br/>New = (0.4×30) + (0.6×10.5) = 18.3
    TS->>UE: THREAT_REPORT_ACK<br/>{"updated_score": 18.3, "status": "LOW_PRIORITY"}

    Note over UE,TS: 새로운 공격 패턴을<br/>동일한 프레임워크로 처리
    end
