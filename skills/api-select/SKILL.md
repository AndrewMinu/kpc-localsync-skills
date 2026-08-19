---
name: api-select
description: 부트캠프 2단계로, 팀 주제에 맞는 관광 API를 골라 활용신청하고 서비스별로 작동을 확인한다.
---

# 2단계 · 우리 데이터 고르기 (api-select)

## 목적
팀이 만든 주제카드를 읽고, **한국관광공사 API 14종 중 우리에게 필요한 것**을 골라 활용신청하고,
서비스마다 실제로 응답이 오는지 확인한다. 4단계에서 데이터를 받을 때 막히지 않게 미리 뚫어두는 단계다.

**주도는 Blue**, 반대편에서 찌르는 사람은 **Green**이다.
*"○○님이 Blue니까 — 우리 문제 확인하려면 뭐가 필요할 것 같아요?"*

## 말투
가볍고 친절하게. 활용신청은 처음 해보는 사람이 많으니 겁주지 말고 "생각보다 금방 돼요 🙂" 톤으로.

## 시작 전
- `context.md`의 **`주제`·`경로`** 를 읽는다(없으면 `topic-select` 먼저).
- **인증키는 이미 있어야 한다** — `bootcamp-start`에서 회원가입·발급을 마치고 왔다.
  없으면 [공공데이터포털](https://www.data.go.kr) 회원가입 → **마이페이지 > 인증키 발급 현황**에서
  **일반 인증키(디코딩)** 1개를 받아오게 한다. **키 하나로 아래 14종 전부 쓴다.**
- 호출 레시피·파라미터는 `../bootcamp-start/references/guides/api-setup.md`.

---

## ① 우리 문제엔 뭐가 필요한가 — 팀이 고른다

**표를 기계적으로 적용하지 않는다.** 팀이 자기 언어로 쓴 주제카드의 **문제·가설·확인할 데이터**를
읽고, **근거와 함께 2~4개를 제안**한 뒤 팀이 고르게 한다.

> *"우리 가설이 '외국인은 오는데 지방 정보가 없다'였잖아요. 그럼 장소가 실제로 얼마나 있는지(공급)랑
> 외국인이 어느 나라에서 오는지(다양성)를 보면 되겠는데 — 이 셋이면 될까요, 빠진 게 있을까요?"*

| API (data.go.kr 검색어) | 서비스 | 무엇을 주나 |
|---|---|---|
| 국문 관광정보 서비스_GW `15101578` | `KorService2` | **거의 모든 팀이 쓴다.** 장소 개수·이름·좌표·운영시간. 프로토타입에 박을 실제 장소가 여기서 나온다 |
| 빅데이터_지역별 방문자수_GW `15101972` | `DataLabService` | 얼마나 오나. 월별 추이로 계절 편차 |
| 지역별 관광 수요 강도 `15151868` | `AreaTarDemDsService` | 자고 가나(체류)·돈 쓰나(소비) |
| 지역별 관광 자원 수요 `15152138` | `AreaTarResDemService` | 관광서비스·문화자원을 찾는 수요. **공급과 비교하면 갭이 보인다** |
| 지역별 관광 다양성 `15151365` | `AreaTarDivService` | 연령·소비·국적이 고른가. 낮으면 쏠렸다는 뜻 |
| 관광지별 연관 관광지 정보 `15128560` | `TarRlteTarService1` | 동선. 보고 나서 어디로 넘어가나 |
| 관광지 집중률 방문자 추이 예측 정보 | `TatsCnctrRateService` | **30일 혼잡 예보.** 검증에도 쓰고 프로토타입 실시간 기능으로도 쓴다 |
| 무장애 여행 정보 | `KorWithService2` | 휠체어·유아차 접근 가능 장소 |
| 반려동물 동반여행 서비스 | `KorPetTourService2` | 반려동물 동반 가능 장소 |
| 웰니스관광정보 | `WellnessTursmService` | 치유·명상·스파. 전국 174건뿐이라 **0이 곧 공백** |
| 고캠핑 정보 조회서비스_GW | `GoCamping` | 야영장·캠핑장 |
| 의료관광정보 | `MdclTursmService` | 의료관광 기관. 서울 65% 편중이라 지역 비교엔 부적합 |
| 두루누비 정보 서비스_GW | `Durunubi` | 걷기여행길 코스 144개. 거리·소요시간·난이도·교통편이 숫자로 있다 |
| 기초지자체 중심 관광지 | `LocgoHubTarService1` | 시군구별 **연계 방문 많은 관광지 100위** + 좌표. 프로토타입 출발점 |

**고를 때 짚어줄 것**
- **공급만 받고 끝내지 않는다.** 장소 개수는 "추천할 게 있나"만 말해준다. 문제를 증명하려면
  **방문 → 체류 → 소비** 사슬 중 어디가 끊겼는지 봐야 한다.
- **혼잡(집중률)은 두 겹으로 쓸모 있다** — 검증 보조 + 프로토타입 실시간 기능. 쏠림·계절·분산 주제가
  아니어도 데모를 살리고 싶으면 신청해둘 만하다.
- 대상이 뚜렷한 주제(무장애·반려동물·웰니스·캠핑·의료)는 그 전용 API가 곧 근거다.
- 애매하면 물어라: *"우리 문제가 '언제·어디가 붐비냐'와 상관있나요?"*

> ⚠ **필요한 것만 고른다.** 활용신청은 건마다 활용목적을 적는 폼이라 14개를 다 하면 20분이 녹는다.

---

## ② 활용신청 (참가자 본인)

고른 것마다 data.go.kr에서 **[활용신청]** 을 누른다. 대부분 즉시 자동승인·무료다.
- 검색창에 위 표의 이름을 그대로 넣으면 나온다. 번호가 있는 건 `data.go.kr/data/{번호}/openapi.do`.
- 개발계정 기준 **1,000회/일**. 부트캠프 규모에선 넉넉하다.

> ⚠ **신청 직후엔 안 된다.** 승인돼도 게이트웨이 반영까지 **최대 1시간(때로 수 시간)**. 이때 호출하면 XML 에러가
> 아니라 평문 `Unauthorized`가 온다 = 아직 미반영. 시간이 지나면 자동으로 풀린다.

## ③ 기다리는 동안 (게이트를 대화로 채운다)

반영을 기다리는 사이 **다음 단계(지역 선택)로 넘어간다.** 지역 토론이 30분~1시간 걸리니 그동안
반영이 끝난다. 넘어가기 전에 팀과 이런 걸 나눠도 좋다:

- *"우리 주제, 팀원 모두 진짜 납득했어요? 갸우뚱한 사람 없어요?"*
- *"이 지역에서 왜 그 문제가 생겼을지, 우리 나름의 가설을 더 세워볼까요?"*
- *"우리 사용자가 누굴지 미리 한 명만 그려볼까요? 다음 단계에서 크게 쓰여요."*
- 시간 되면 `$deep-dive`로 그 지역·사용자 이야기를 미리 들어봐도 좋다.

## ④ 활성확인 — **고른 서비스마다** 한 번씩

한 건만 찔러보고 "됐다"고 넘기지 않는다. **신청한 서비스 각각**에 최소 호출을 날린다.
값은 출력하지 않고 응답 코드만 본다.

```
# 공급 (KorService2)
https://apis.data.go.kr/B551011/KorService2/areaBasedList2?serviceKey={KEY}&numOfRows=1&pageNo=1&MobileOS=ETC&MobileApp=ftskill&arrange=A&contentTypeId=12&areaCode=1&sigunguCode=1&_type=json

# 수요강도 (AreaTarDemDsService)
https://apis.data.go.kr/B551011/AreaTarDemDsService/areaTarSjrnDsList?serviceKey={KEY}&MobileOS=ETC&MobileApp=ftskill&_type=json&baseYm=202509&areaCd=11&signguCd=11530&tarSjrnDsIxCd=2102

# 집중률 (TatsCnctrRateService) — baseYm 넣으면 에러난다
https://apis.data.go.kr/B551011/TatsCnctrRateService/tatsCnctrRatedList?serviceKey={KEY}&numOfRows=1&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&areaCd=11&signguCd=11110
```

```
# 방문자수 (DataLabService) — startYmd 필수. touDivCd·signguCd는 요청에 넣으면 에러다
https://apis.data.go.kr/B551011/DataLabService/locgoRegnVisitrDDList?serviceKey={KEY}&numOfRows=1&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&startYmd=20250901&endYmd=20250901

# 반려동물 — Service'2' 다. KorPetTourService는 없는 서비스다
https://apis.data.go.kr/B551011/KorPetTourService2/areaBasedList2?serviceKey={KEY}&numOfRows=1&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&areaCode=1

# 두루누비 — 경로에 Service가 안 붙는다
https://apis.data.go.kr/B551011/Durunubi/courseList?serviceKey={KEY}&numOfRows=1&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json
```

나머지 서비스의 최소 호출은 `api-setup.md`의 해당 절(§3.5~§3.9)을 그대로 쓴다. **14종 전부
2026-08-13에 실호출로 확인됐다** — 실패하면 경로 문제가 아니라 **활용신청이 빠진 것**이다.

**판정과 안내**
| 응답 | 뜻 | 안내 |
|---|---|---|
| `resultCode:0000` | 활성 | 통과 |
| 평문 `Unauthorized` | 키 또는 신청이 아직 미반영 | 잠시 후 재시도 |
| `SERVICE_ACCESS_DENIED` / `Forbidden` | **그 서비스만 활용신청이 빠졌다** | **어느 API인지 이름을 대고** 신청하게 한다 |
| `INVALID_REQUEST_PARAMETER` | 파라미터 문제 (키는 정상) | `api-setup.md`의 필수 파라미터 확인 |
| `NO_OPENAPI_SERVICE_ERROR` | **엔드포인트 경로가 틀렸다** (활용신청 문제 아님) | §1.5 표의 경로를 그대로 복사 |

> 실패했을 때 "안 되네요"로 끝내지 않는다. **어느 활용신청이 빠졌는지 이름을 말해준다.**
> 4단계에서 이걸 발견하면 1시간을 다시 기다려야 한다.

---

## 보안 (짚고 가기)
- 이 키는 **공공데이터포털 무료 인증키**로 민감도가 낮다(요금 없음·요청 제한 있음·언제든 재발급/폐기).
  비밀번호·결제정보가 아니다.
- 그래도 **깃 커밋·카드·`context.md`·HTML에는 저장하지 않는다.** 화면에 값을 다시 출력하지도 않는다.
- 채팅에 붙여넣는 건 실무상 무난하다. 세션이 바뀌면 다시 붙여넣게 하면 된다.
- (선택·앱 제출자용) 별도 앱을 만들 팀은 프로젝트 `.env`(`DATA_GO_KR_SERVICE_KEY=...`, `.gitignore`에
  추가)에 둔다. ⚠ 다른 터미널에서 `export` 한 값은 Codex 셸에 안 보일 수 있으니 `.env`를 쓴다.

## 자동으로 채우지 않기
**어떤 API가 필요한지는 팀이 정한다.** AI는 주제카드를 근거로 제안만 하고, 팀이 고르지 않은 것을
"일단 다 신청해두죠"로 밀지 않는다. 왜 그게 필요한지 팀이 한 줄로 말할 수 있어야 고른 것이다.

## context.md 갱신
- `활용신청`: 고른 API 이름 목록 + 각각 `활성확인` 여부. **키 값은 절대 기록하지 않는다.**
- `단계: 2`, 로그 추가.

## 끝맺음
- *"필요한 데이터 창구는 다 열어놨어요. 반영에 좀 걸릴 수 있으니 그동안 지역부터 정하죠 🙂"*
- **다음은 `$region-select`** — 이 주제를 어느 지역에서 풀지 팀이 정한다.
- 미반영(`Unauthorized`)이 남아 있어도 **멈추지 않는다.** 지역 대화를 하고 4단계 들어가기 전에
  다시 확인하면 된다.
