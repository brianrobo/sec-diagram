sequenceDiagram
    autonumber
    participant TS as Trust Server
    participant UE as UE (User Equipment)
    participant LegitBS as Legitimate gNB/eNB
    participant CN as Core Network (AMF/MME)

    rect rgb(240, 255, 240)
    Note over UE,LegitBS: C-1. Legitimate Cell Authentication Success

    Note over UE: Cell previously reported as suspicious<br/>CellID = "450-05-eNB7777-Cell2"<br/>Current Score: 35 (LOW_PRIORITY)

    UE->>LegitBS: ATTACH REQUEST / REGISTRATION REQUEST<br/>(GUTI, Last Visited TAI)
    LegitBS->>CN: Forward Attach Request

    CN->>LegitBS: AUTHENTICATION REQUEST<br/>(RAND, AUTN - legitimate values)
    LegitBS->>UE: AUTHENTICATION REQUEST

    UE->>UE: Verify AUTN<br/>Calculate XRES from USIM<br/>Check SQN (Sequence Number)

    Note over UE: ✅ AUTN Verification SUCCESS<br/>This indicates legitimate network

    UE->>LegitBS: AUTHENTICATION RESPONSE<br/>(RES)
    LegitBS->>CN: Forward Auth Response

    CN->>CN: Verify RES = XRES<br/>Authentication SUCCESS

    CN->>LegitBS: SECURITY MODE COMMAND<br/>(Selected: EEA2, EIA2 - Strong Encryption)
    LegitBS->>UE: SECURITY MODE COMMAND

    UE->>UE: Verify strong encryption selected<br/>EEA2/EIA2 confirmed

    UE->>LegitBS: SECURITY MODE COMPLETE<br/>(NAS-MAC verified)
    LegitBS->>CN: Forward Security Mode Complete

    Note over UE: ✅ Full AKA Success with Strong Encryption<br/>Cell is LEGITIMATE

    UE->>UE: Determine Report Score<br/>AKA_SUCCESS → report_score = 0<br/>CellID = "450-05-eNB7777-Cell2"

    UE->>TS: RECOVERY_REPORT<br/>{<br/>"cell_id": "450-05-eNB7777-Cell2",<br/>"event_type": "AKA_SUCCESS",<br/>"report_score": 0,<br/>"evidence": {<br/>  "authentication_success": true,<br/>  "autn_verified": true,<br/>  "encryption_algo": "EEA2",<br/>  "integrity_algo": "EIA2",<br/>  "security_mode_complete": true<br/>},<br/>"timestamp": "2026-01-17T12:45:30Z"<br/>}

    TS->>TS: EMA Calculation (α=0.8)<br/>Current: 24.7, Report: 0<br/>New = (0.8×0) + (0.2×24.7) = 4.9<br/>Status: NORMAL ✅
    TS->>UE: RECOVERY_REPORT_ACK<br/>{<br/>"cell_id": "450-05-eNB7777-Cell2",<br/>"updated_score": 4.9,<br/>"alpha_used": 0.8,<br/>"status": "NORMAL"<br/>}

    UE->>UE: Remove from Low Priority List<br/>Restore normal ranking

    CN->>LegitBS: ATTACH ACCEPT / REGISTRATION ACCEPT
    LegitBS->>UE: ATTACH ACCEPT / REGISTRATION ACCEPT

    Note over UE,CN: ✅ Successfully connected to<br/>RECOVERED LEGITIMATE CELL
    end
