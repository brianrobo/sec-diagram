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
