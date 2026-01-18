### 3.1 Scenario A - IMSI Catcher Detection 스펙 준수 여부

| 단계 | 동작 | 스펙 준수 | 근거 |
|------|------|:--------:|------|
| 1 | A3 Handover Event | ✅ | TS 36.331 Section 5.5.4 - Measurement Report |
| 2 | RRCConnectionReconfiguration | ✅ | TS 36.331 Section 5.3.5 - Handover |
| 3 | RACH Failure (No RAR) | ✅ | FBS가 응답 안하면 T304 만료 (TS 36.321) |
| 4 | T304 Expiry → HO Failure | ✅ | TS 36.331 Section 5.3.5.6 |
| 5 | RRC Reestablishment Request | ✅ | TS 36.331 Section 5.3.7 |
| 6 | RRC Reestablishment Reject | ✅ | eNB/gNB는 Reject 가능 |
| 7 | Enter RRC_IDLE | ✅ | Reject 시 의무적 IDLE 전환 |
| 8 | TAU Request (TAC mismatch) | ✅ | TS 24.301 Section 5.5.3 |
| 9 | Identity Request (IMSI) | ✅ | TS 24.301 Section 5.4.4 |
| 10 | **Pattern Detection** | 🔵 **특허 신규성** | **워크플로우 기반 탐지** |
| 11 | **Trust Server Report** | 🔵 **특허 신규성** | **협업 방어 메커니즘** |




1) 공격자는 매우 강한 출력(을 내보내어 단말이 A3 Event를 발생 유도.
  * A3 Event (Neighbor Becomes Better than SpCell)
2)  정상 기지국에 MR 보냄.
3) 정상 기지국이 handover command (rrr connection reconfiguration)

4) 단말은 이제 FBS cell은 handover 시도.
5) RACH 시도하나 No response(No RAR)
6) 결국 T304 timer expiry 되어 Handover failure발생.
    // T304 망이 주는 값으로, 보통 500ms(0.5초) 또는 **1000ms(1초)**를 가장 많이 사용.
         기술 규격,설정 가능한 값 (ms)
      LTE (TS 36.331),"ms50, ms100, ms150, ms200, ms500, ms1000, ms2000, spare1"
      5G NR (TS 38.331),"ms50, ms100, ms150, ms200, ms500, ms1000, ms2000, ms10000"


   // (me) 만약 핸드오버 성공 시 AKA인증을 거쳐야하나, FBS는 코어망이 없으므로 AKA인증 통과 불가.
          따라서 H/O 고의로 실패 후 재연결 (rrc reest) 유도하고, 여기에 reject을 때려 단말을 rrc idle로 강제로 전환.
           최종목적) idle 상태가 된 단말은 가장 신호가 센 셀을 다시 찾게됨(cell reselection)
            이때 미리 대기중인 fbs가 tau 를 유도하여 identity request를 보낼 명분으 만듬.
   // (me) idle에서는 단말의 최적의 cell을 직접 찾아서 선택해야함.



   // 일반적인 망동록 상황에서, idle은?
      > data inactivity 에서 idle에서도도 best cell이 있으면 cell resel 수행함..
      > 그렇다면 왜 공격자는 ScenarioA를 사용한 이유는,  즉시성과 확실성 때문.
          > 정상적인 경우는 rrc conn이고,, 데이터 사용안할때 까지 기다려야함.  알수가 없다.
          > 따라서 공격자는 강제로 H/O 시켜서 연결을 끊어버려서, idle 확실하게 즉시 변경하게끔 유도 후.
      > 그다음음 sib1을 읽을 때, tac값을 다른값을 설정되어 있으므로,, tau req를 유도하고,,
      >  identity request는 (스펙상 optional)이고,,, tau req할 때 guti를 사용하더라고, 공격자는 모르는 guti인척하면 imsi를 요청한다.
          > guti를 줘봐도,, fbs 는 코어망이 없으니 guti가 누구인지 알 방법은 없음. 따라서 결국 identity request
      >
      > 



# 나중에 추가할 시나리오

1. 서비스 거부 및 강제 하향 (CSFB/SRVCC 유도 시나리오)
LTE(4G) 전용 단말이 아닌, 음성 통화를 위해 3G/2G로 내려가는 기능을 악용하는 방식입니다.

공격 방식:

FBS가 LTE 기지국인 척 단말을 유인합니다.

단말이 접속하면 FBS는 **"우리 셀은 음성 통화를 지원 안 하니 3G(WCDMA)나 2G(GSM)로 내려가!"**라며 CS Service Notification이나 RRC Release with Redirection을 보냅니다.

이때 단말을 미리 대기 중인 3G/2G 가짜 기지국으로 보냅니다.

왜 하는가?: 3G나 2G는 LTE에 비해 보안 프로토콜이 훨씬 취약합니다. 특히 2G(GSM)는 기지국이 단말만 인증하고, **단말은 기지국을 인증하지 않는 '단방향 인증'**을 사용합니다.

Identity Request 타이밍: 3G/2G로 내려가자마자 수행하는 Location Updating Request 과정에서 아주 쉽게 Identity Request를 던져 IMSI를 탈취합니다.

2. 초기 접속 차단 및 재접속 유도 (Attach Reject 시나리오)
단말이 전원을 켰을 때나 비행기 모드를 해제했을 때 발생하는 Initial Attach를 노리는 방식입니다.

공격 방식:

단말이 처음 망에 붙으려고 Attach Request를 보냅니다.

FBS는 이를 가로챈 뒤 곧바로 **Attach Reject**를 보냅니다.

이때 Reject 사유(Cause Value)로 #9 (MS identity cannot be derived by the network) 또는 #10 (Implicitly detached) 등을 사용합니다.

핵심 메커니즘: 이 사유를 받은 단말은 "아, 내 임시 ID(GUTI)가 서버에서 삭제됐구나"라고 판단합니다.

Identity Request 타이밍: 단말은 즉시 다음 Attach Request를 보낼 때 GUTI 대신 IMSI를 직접 담아서(평문) 보내거나, FBS가 다시 Identity Request를 보낼 명분을 완벽하게 만들어줍니다.


3. "유도(Steering)"의 또 다른 진화: Silent Paging
단말이 가만히 IDLE 상태로 있을 때, 강제로 말을 걸어 깨우는 방식입니다.

공격 방식: FBS가 단말의 존재를 확인하면, 가짜 Paging 메시지를 방송합니다.

동작: 단말은 누군가 나에게 전화를 걸거나 데이터를 보내는 줄 알고 Service Request를 보내며 RRC_CONNECTED 상태로 올라옵니다.

결과: 단말이 깨어나는 순간, FBS는 다시 위에서 언급한 절차들(TAU 유도 또는 Identity Request 직접 송신)을 수행합니다.

4. 질문자님의 특허 아이디어(Scoring)에 적용
이 추가 시나리오들을 알고리즘에 넣으면 다음과 같은 **'복합 지표'**를 만들 수 있습니다.

Scenario B (RAT Downfall): LTE에서 갑자기 이유 없이 2G/3G로 Redirection을 지시받은 후, 그 직후에 Identity Request가 발생하면? (+25점)

Scenario C (Attach Abuse): Attach Reject 수신 후 재시도 과정에서 Identity Request가 발생하면? (+25점)

💡 특허 preview 미팅용 전략적 멘트
"공격자는 단순히 TAU만 노리는 것이 아니라, 단말의 프로토콜 취약점을 이용해 2G로 강제 하강시키거나(CSFB), 임시 ID가 무효하다고 속이는(Attach Reject) 등 다양한 경로로 Identity Request를 유도합니다. 저희 시스템은 이러한 다양한 'Identity Request 유발 경로'를 시나리오 DB화하여, 어떤 방식으로 접근하든 결국 **최종적인 정보 탈탈취 행위(Identity Request without AKA)**를 포착해 낼 수 있습니다."





공격 전략 (Strategy),기술적 수단 (Tactics),구체적 동작 (Procedure),상세 목적 (Objective)
1. 강제 유입형 (Forced Ingress),Handover Manipulation,정상 Cell에서 A3 Event 유도 후 HO Failure 발생 → Reestablishment Reject,단말을 즉시 Disconnected 상태로 만들어 FBS를 Best Cell로 선택하게 함
2. 위치 갱신형 (Tracking Area Update),TAC Mismatch,SIB1에서 가짜 TAC 방송 → 단말의 TAU Request 유발,위치 등록 절차를 명분으로 Identity Request를 던질 타이밍 확보
3. 보안 무력화형 (Security Downgrade),RAT Redirection,LTE 접속 단말에게 CSFB(Voice) 유도 후 2G/3G로 강제 이동,보안이 취약한 하위 RAT에서 평문(Plaintext) Identity Request 수행
4. 신원 무효화형 (Identity Invalidation),Attach Reject,"초기 접속 시 ""임시 ID(GUTI) 무효"" 사유(#9)로 거절",단말이 스스로 IMSI를 포함한 Attach Request를 보내도록 강제
5. 강제 기상형 (Silent Paging),False Paging,IDLE 단말에게 가짜 Paging 전송 → Service Request 유발,잠자는 단말을 깨워 통신 세션을 열고 NAS 절차(Identity Request) 시작




| 공격 전략 (Strategy) | 기술적 수단 (Tactics) | 주요 동작 (Procedure) | 기술적 큰 목적 (High-level Objective) |
| :--- | :--- | :--- | :--- |
| **1. 강제 유입형** | **Handover Manipulation** | A3 Event 유도 → HO Failure (T304 만료) → Reestablishment Reject | **[단말 확보]** 정상 망과의 연결을 강제로 끊고 단말을 IDLE 상태로 만들어 FBS를 최우선 셀로 선택하게 함 |
| **2. 위치 갱신형** | **TAC Mismatch** | SIB1에서 인접 셀과 다른 TAC 방송 → 단말의 **TAU Request** 유발 | **[절차 유도]** 위치가 바뀌었다는 명분을 주어 단말이 스스로 신원 보고(NAS 절차)를 시작하도록 유도 |
| **3. 보안 무력화형** | **RAT Redirection** | LTE 단말에게 CSFB 지시 → 2G/3G FBS로 강제 이동 (Downgrade) | **[보안 약화]** 상위 RAT(4G/5G)의 강력한 보안을 우회하여, 평문 Identity Request가 쉬운 하위 망으로 유인 |
| **4. 신원 무효화형** | **Attach Reject** | 초기 접속 시 "ID 무효" 사유(#9)로 Reject → 재접속 유도 | **[ID 초기화]** 단말이 가진 임시 ID(GUTI)를 강제로 폐기시키고, 진짜 ID(IMSI)를 제출할 수밖에 없는 상황을 조성 |
| **5. 강제 기상형** | **Silent Paging** | IDLE 상태 단말에게 가짜 Paging 전송 → **Service Request** 유발 | **[세션 강제]** 가만히 있는 단말을 깨워 통신 세션을 열게 만든 뒤, 즉각적으로 Identity Request를 던질 기회 확보 |



   
