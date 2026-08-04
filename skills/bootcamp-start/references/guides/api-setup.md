# 공공데이터포털 API 키 설정 (자동 데이터 검증용)

> 키 1개를 발급해 환경변수로 두면, `tourism-data-validator`와 `topic-card-builder`의 데이터 검증 게이트가 **자동으로** 실시간 데이터를 받아 시그니처를 확인한다.
> Codex·Claude 모두 셸 환경변수를 읽으므로 동일하게 작동한다.

## 0. 핵심: 키는 1개, 신청은 API별 1회
> **공공데이터포털 계정당 일반 인증키(서비스키)는 단 1개.** 이 키 하나로 아래 모든 API를 쓴다.
> 단, **API(데이터셋)마다 "활용신청"을 한 번씩** 해야 한다(대부분 즉시 자동승인·무료). 신청이 승인되면 그 API도 *같은 키*로 동작한다.
> 즉 **관리할 키 = 1개**, 해야 할 일 = **아래 4개 활용신청**. API마다 다른 키를 받는 게 아니다.

## 1. 발급·신청 절차
1. [공공데이터포털](https://www.data.go.kr) 회원가입·로그인.
2. 아래 4개 API 페이지에서 각각 **[활용신청]** 클릭(대부분 자동승인, 일부 1~2일).
3. **마이페이지 > 데이터 활용 > 인증키 발급 현황**에서 내 **일반 인증키(서비스키)** 1개를 확인. → 이 키를 모든 API에 공통으로 사용.
   - 키 형식은 계정에 따라 64자리 16진수(hex) 또는 Base64형(`+`·`/`·`=` 포함) 둘 다 가능. 둘 다 정상이다.

> ⚠ **신규 발급 키는 즉시 작동하지 않는다.** 키 발급/활용신청 승인 직후에는 게이트웨이 반영까지 **최대 1시간(때로 수 시간)** 걸린다. 이때 호출하면 응답이 XML 에러가 아니라 평문 **`Unauthorized`** 로 온다 = "아직 미반영". 시간이 지나면 자동 해결되므로 잠시 후 재시도한다.

| 활용신청할 API | 역할 | data.go.kr |
|---|---|---|
| 한국관광공사_빅데이터_지역별 방문자수_GW | **수요 신호(자동)** — 데이터랩 방문자수 | https://www.data.go.kr/data/15101972/openapi.do |
| 한국관광공사_관광지별 연관 관광지 정보 | 동선·중심-연관(자동) | https://www.data.go.kr/data/15128560/openapi.do |
| 한국관광공사_국문 관광정보 서비스_GW (TourAPI) | **장소 공급(자동)** | https://www.data.go.kr/data/15101578/openapi.do |
| 문화체육관광부 문화예술공연 통합 API | 공연·전시 보강 | https://www.data.go.kr/data/15121487/openapi.do |

> 참가자에게도 동일하게 안내: "키 1개 발급 + 위 활용신청"이면 끝. (SGIS·AI허브는 별도 사이트라 키가 다름 → §3 수동 보강)

> 참고: 수요 신호 API는 "빅데이터 지역별 방문자수_GW(15101972)" 또는 "지역별 관광 자원 수요" 중 신청한 것을 쓴다. 둘은 엔드포인트가 다르므로, 실제 승인된 API 기준으로 호출 경로를 맞춘다(키 활성화 후 확인).

## 2. 키 등록 (환경변수)
표준 변수명: **`DATA_GO_KR_SERVICE_KEY`**

```bash
# 셸/세션에 등록 (Codex·Claude 공통)
export DATA_GO_KR_SERVICE_KEY="발급받은_일반_인증키_Decoding"
```
> 키는 코드·카드·깃에 하드코딩하지 않는다. 환경변수로만 둔다. (디코딩 키 사용 권장)

## 3. 자동/수동 경계
- **키로 자동 검증되는 것**: 지역별 방문자수(수요), 연관 관광지(동선), 관광지·음식점·숙박·행사 개수(공급), 공연·전시.
- **여전히 CSV 다운로드(수동) 보강**: 데이터랩 *체류시간·관광소비*, *야간관광*, *외래객 통계*, 통계청 SGIS 생활인구, AI허브 데이터셋.
- 검증 게이트는 이 경계를 알고 분기한다: 자동 가능한 지표는 받아서 계산, 수동 지표는 `⚠미검증` + 다운로드 액션으로 남긴다.

## 3.5 확인된 호출 레시피 (2026-06-24 실호출 성공)

> 아래 호출은 실제로 검증됨. 키 활성화 후 그대로 재현 가능.

**(a) 지역코드 조회** — `areaCode2` (전국 광역 코드)
```
https://apis.data.go.kr/B551011/KorService2/areaCode2?serviceKey={KEY}&numOfRows=20&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json
```
예: 서울=1, 인천=2, 대전=3, 대구=4, 광주=5 … 강원=32. 시군구 코드는 `areaCode2`에 `&areaCode=32` 추가해 조회.

**(b) 지역별 장소 공급 개수** — `areaBasedList2` (★검증 핵심★)
```
https://apis.data.go.kr/B551011/KorService2/areaBasedList2?serviceKey={KEY}&numOfRows=1&pageNo=1&MobileOS=ETC&MobileApp=ftskill&arrange=A&contentTypeId=39&areaCode=32&sigunguCode=1&_type=json
```
- `numOfRows=1`로 호출하면 응답의 **`totalCount`** 가 그 지역·카테고리의 **개수**(= 공급 신호)다.
- 실호출 결과: 강릉(areaCode=32, sigunguCode=1) 음식점 = **486건**. (정적 카드의 492건과 차이 → 데이터 갱신, 자동 검증으로 최신값 반영)

**contentTypeId 코드 (KorService2)**
| 코드 | 종류 | 코드 | 종류 |
|---|---|---|---|
| 12 | 관광지 | 28 | 레포츠 |
| 14 | 문화시설 | 32 | 숙박 |
| 15 | 축제·공연·행사 | 38 | 쇼핑 |
| 25 | 여행코스 | 39 | 음식점 |

> 한 지역의 공급 프로필 = 위 코드별로 `totalCount`를 합산해 비교. 이게 카드의 "TourAPI 공급 단서"를 자동 채운다.

## 3.6 관광수요지수 3종 (데이터랩 지표를 API로) ★신규

> 데이터랩 '관광수요지수'를 API로 제공. **체류·소비를 이제 수동 다운 없이 받는다.**
> ⚠ 이 3종의 `areaCd`/`signguCd`는 **행정 코드**(서울 11, 구로 11530) — TourAPI 지역코드와 다르다.
> 값은 **상대 지수**(예: 79.08) — 절대 해석 금지, 지역 간 비교·순위로.

| API | Base URL (B551011) | 오퍼레이션 | 핵심 지표코드 |
|---|---|---|---|
| **관광 수요 강도** ★ | `AreaTarDemDsService` | `/areaTarSjrnDsList` (체류)<br>`/areaTarExpDsList` (소비) | 체류 `2102` **숙박 비중**(2101 타권역비중, 2103~05 1·2·3박, 21 전체)<br>소비 `2201` **외지인 소비액**(2202 소비비중, 2203 방문대비소비, 22 전체) |
| 관광 자원 수요 | `AreaTarResDemService` | `/areaTarSvcDemList`<br>`/areaCulResDemList` | 유형별 SNS 언급·소비·내비 검색량 (수요) |
| 관광 다양성 | `AreaTarDivService` | `/areaTouDivList`(연령)<br>`/areaExpDivList`(소비연령)<br>`/areaIntlDivList`(국제·국적) | 연령·국적 다양성 (※자원 다양성 아님) |

**호출 예 (체류·숙박비중)**
```
https://apis.data.go.kr/B551011/AreaTarDemDsService/areaTarSjrnDsList?serviceKey={KEY}&MobileOS=ETC&MobileApp=ftskill&_type=json&baseYm=202509&areaCd=11&signguCd=11530&tarSjrnDsIxCd=2102
```
- 필수: `serviceKey`·`MobileOS`·`MobileApp`·`baseYm`(YYYYMM)·`areaCd`. `signguCd`·지표코드는 옵션.
- **`signguCd`를 빼면 그 시도의 전 시군구**가 나온다 → 17개 시도 루프로 전국 수집 가능.
- 응답: `areaNm`·`signguNm`·`{지표}IxNm`·`{지표}IxVal`.
- ⚠ **지표코드를 안 넣으면 0건**이 나올 수 있다(반드시 지정).

**활용신청**: 세 API 각각 1회(자동승인·무료, 개발계정 1,000회/일).
- 수요 강도 — data.go.kr/data/15151868 · 자원 수요 — 15152138 · 다양성 — 15151365

## 4. 스킬이 키를 쓰는 방식
- `tourism-data-validator`는 실행 시 `DATA_GO_KR_SERVICE_KEY` 존재 여부를 먼저 확인한다.
  - 있으면: 위 자동 데이터셋을 호출해 평균·합계·순위·비교·증감으로 시그니처를 검증.
  - 없으면: 전부 다운로드 액션 목록으로 정리(미검증 플래그).
- `topic-card-builder`의 2단계 데이터 검증은 이 스킬 로직을 그대로 호출한다.

## 3.7 관광지 집중률·방문 추이 예측 (TatsCnctrRateService) — 참가자 조건부
`GET https://apis.data.go.kr/B551011/TatsCnctrRateService/tatsCnctrRatedList`
- 필수: `areaCd`(행정 시도) + `signguCd`(행정 시군구). **`baseYm`/`baseYmd`를 넣으면 INVALID 에러** — 넣지 않는다.
- 응답: `tAtsNm`(관광지) · `baseYmd` · `cnctrRate`. 조회일부터 **30일치 예보**, 가장 붐빌 때를 100으로 본 **상대 지수**(KT 통신데이터 기반).
- 예) 종로구 = 관광지 34곳 × 30일(3,390행). 가회민화박물관 월 57.9 → 토 97.8 → 월 58.8.
- **카드 근거로 쓰지 않는다** — 조회 시점마다 값이 바뀌어 고정 근거가 못 된다.
- 쓰임: (1) 쏠림·계절·분산 주제 팀의 **검증 보조**, (2) 프로토타입의 **혼잡도 기능**.
- ⚠ **CORS 불가**: `apis.data.go.kr`은 CORS 헤더를 주지 않는다 → 브라우저(HTML)에서 직접 fetch하면 실패한다.
  `file://`로 연 HTML은 옆에 둔 로컬 `data.json`을 fetch하는 것도 막힌다(origin `null`) → **별도 JSON 파일도 쓰지 않는다.**
  프로토타입은 **터미널에서 한 번 호출 → 결과를 HTML 스크립트에 `const DATA = [...]` 로 인라인**한다. 파일 하나로 열리고, 키는 `.env`에만 남는다(HTML에 넣으면 업로드 시 노출).
  실서비스라면 정적 데이터가 아니라 **서버가 API를 대신 호출**하는 구조로 간다(CORS·키 문제가 함께 사라진다). 피칭에서 이 한계를 짚으면 좋다.
  → `participant-skills/data-collect-validate` §④, `participant-skills/prototype-build` 심화에 반영됨.

## 3.8 관광지별 연관 관광지 (TarRlteTarService1) — 카드 근거
`GET .../TarRlteTarService1/areaBasedList1` · 필수: `baseYm`(202509) + `areaCd` + `signguCd`(**시군구 필수**).
- 티맵 내비 기반 관광지→연관 관광지 랭킹. 파생: **동선 유출비율**(연관지가 타 시군구인 비율), 연관 관광지 수, 최다 유출처.
- 전국 252 시군구 수집 완료 → `templates/context_rlte.csv` (평균 유출 40.8%). 도시 구는 구조상 높으니 **도시·시군을 나눠 비교**한다.

## 3.9 니치 공급 3종 — 카드 근거
- 무장애: `KorWithService2/areaBasedList2` (TourAPI 코드) — 9,948건
- 웰니스: `WellnessTursmService/areaBasedList` (**`langDivCd=KOR` 필수**, lDong 코드) — 174건, 137개 시군구 0건
- 캠핑: `GoCamping/basedList` (도/시군구 **이름**) — 3,044건
→ `templates/context_niche.csv`
- ⚠ 2026년 **광주+전남 → '전남광주통합특별시'(코드 12)** 통합. 시도코드로만 집계하면 두 지역이 0으로 보인다. 이름 기준으로 되돌려 붙일 것.
- 의료관광(`MdclTursmService`, `langDivCd` 필수): 331건 중 서울 214(65%) — 지역 편중이 심해 **카드로 쓰지 않는다**.
