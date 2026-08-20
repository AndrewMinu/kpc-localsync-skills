# 공공데이터포털 API: 키·활용신청·호출 레시피

> 어떤 API를 고를지는 `$api-select`가 팀 주제를 보고 함께 정한다. 이 문서는 **고른 뒤에 어떻게
> 부르는지**를 담는다. 값 해석·주의는 `../data/README.md`.

## 0. 핵심: 키는 1개, 신청은 API별 1회
> **공공데이터포털 계정당 일반 인증키는 단 1개.** 이 키 하나로 아래 API 전부를 쓴다.
> 단, **API(데이터셋)마다 "활용신청"을 한 번씩** 해야 한다(대부분 즉시 자동승인·무료).
> 즉 **관리할 키 = 1개**, 해야 할 일 = **고른 API 수만큼의 활용신청**.

## 1. 발급 절차
1. [공공데이터포털](https://www.data.go.kr) 회원가입·로그인.
2. **마이페이지 > 데이터 활용 > 인증키 발급 현황**에서 **일반 인증키(디코딩)** 1개를 확인.
   - 키 형식은 계정에 따라 64자리 16진수(hex) 또는 Base64형(`+`·`/`·`=` 포함) 둘 다 가능. 둘 다 정상이다.
3. 쓸 API마다 그 페이지에서 **[활용신청]**.

> ⚠ **신규 키·신규 활용신청은 즉시 작동하지 않는다.** 게이트웨이 반영까지 **최대 1시간(때로 수 시간)**.
> 이때 호출하면 XML 에러가 아니라 평문 **`Unauthorized`** 가 온다 = "아직 미반영". 잠시 후 재시도.

## 1.5 활용신청 대상: 한국관광공사 API 14종 (단일 목록)

**이 표가 유일한 목록이다.** 다른 문서는 여기를 가리킨다.

> ⚠ **관광공사의 "법정동코드 변경 대상 14건" 공지와 이 표의 14종은 다른 목록이다.** 공지 쪽 14건은
> 국문·다국어 관광정보 9종 + 무장애·의료관광·웰니스·관광공모전·반려동물이다. **겹치는 건 5종**.
> `KorService2`(국문)·`KorWithService2`(무장애)·`MdclTursmService`(의료관광)·`WellnessTursmService`(웰니스)·`KorPetTourService2`(반려동물).
> **나머지 9종은 공지 대상이 아니고, 2026-08-19 실측에서도 옛 행정코드로 정상 작동했다**. 수요강도·집중률·연관관광지·중심관광지를 `29110`(광주 동구)·`46110`(목포시)·`28260`(인천 서구)로 불러도 값이 나온다.
> 즉 **`region_admin.csv`는 고치지 않는다.** 자세히는 `../data/README.md`의 **⚠ 2026-07-01 행정구역 개편** 절.

| API 이름 (data.go.kr 검색어) | 엔드포인트 | 절 | data.go.kr |
|---|---|---|---|
| 국문 관광정보 서비스_GW | `KorService2/areaBasedList2` | §3.5 | `/data/15101578` |
| 빅데이터_지역별 방문자수_GW | `DataLabService/locgoRegnVisitrDDList` | §3.55 | `/data/15101972` |
| 기초지자체 중심 관광지 | `LocgoHubTarService1/areaBasedList1` | §3.56 | `/data/15128559` |
| 두루누비 정보 서비스_GW | `Durunubi/courseList` | §3.57 | 이름으로 검색 |
| 지역별 관광 수요 강도 | `AreaTarDemDsService/areaTarSjrnDsList`·`areaTarExpDsList` | §3.6 | `/data/15151868` |
| 지역별 관광 자원 수요 | `AreaTarResDemService/areaTarSvcDemList` | §3.6 | `/data/15152138` |
| 지역별 관광 다양성 | `AreaTarDivService/areaTouDivList` | §3.6 | `/data/15151365` |
| 관광지 집중률 방문자 추이 예측 | `TatsCnctrRateService/tatsCnctrRatedList` | §3.7 | 이름으로 검색 |
| 관광지별 연관 관광지 정보 | `TarRlteTarService1/areaBasedList1` | §3.8 | `/data/15128560` |
| 무장애 여행 정보 | `KorWithService2/areaBasedList2` | §3.9 | 이름으로 검색 |
| 반려동물 동반여행 서비스 | `KorPetTourService2/areaBasedList2` | §3.9 | 이름으로 검색 |
| 웰니스관광정보 | `WellnessTursmService/areaBasedList` | §3.9 | 이름으로 검색 |
| 고캠핑 정보 조회서비스_GW | `GoCamping/basedList` | §3.9 | 이름으로 검색 |
| 의료관광정보 | `MdclTursmService/areaBasedList` | §3.9 | 이름으로 검색 |

> ✅ **14종 전부 실호출 확인**(2026-08-13 · **2026-08-19 재확인**, 15개 호출 전부 `HTTP 200` / `resultCode:0000`).
> §3.5~§3.9에 14종 각각의 **완전한 최소 호출**이 있다. 키만 넣으면 그대로 실행된다.
>
> ⚠ 이름이 헷갈리는 셋, 반려동물은 `KorPetTourService`가 아니라 **`KorPetTourService2`**,
> 두루누비는 **`Service`가 안 붙고**(`B551011/Durunubi/...`), 중심 관광지는 **`LocgoHubTarService1`**이다.

(선택 보강) 문화체육관광부 문화예술공연 통합 API, `/data/15121487`. 공연·전시 후보 보강용.

## 2. 키를 넘기는 법
**기본은 채팅에 그대로 붙여넣기.** 환경변수 설정을 몰라도 된다. 스킬이 이번 세션 동안 쓴다.

별도 앱을 만들어 제출할 팀만 프로젝트 `.env`에 둔다:
```bash
DATA_GO_KR_SERVICE_KEY="발급받은_일반_인증키_디코딩"   # .gitignore에 .env 추가
```
> ⚠ Codex는 **다른 터미널에서 `export` 한 값을 못 볼 수 있다** → `.env`를 쓴다.
> 키는 코드·카드·`context.md`·깃·HTML에 하드코딩하지 않는다.

## 3. 이 키로 안 되는 것
통계청 **SGIS**(생활인구)와 **NIA AI허브**는 별도 사이트라 키가 다르다. 회원가입 후 직접 다운로드해야
하고 시간이 걸리니 **1일차 출발용으로 쓰지 않는다.**

## 3.5 확인된 호출 레시피

> 아래 호출은 실제로 검증됨(2026-06-24 최초 · 2026-08-13 재확인). 키 활성화 후 그대로 재현 가능.

**(a) 지역코드 조회**: `areaCode2` (전국 광역 코드)
```
https://apis.data.go.kr/B551011/KorService2/areaCode2?serviceKey={KEY}&numOfRows=20&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json
```
예: 서울=1, 인천=2, 대전=3, 대구=4, 광주=5 … 강원=32. 시군구 코드는 `areaCode2`에 `&areaCode=32` 추가해 조회.
> **이 코드는 2026-07-01 행정구역 개편의 영향을 받지 않는다**. 2026-08-19 실측에서도 광주=5·전남=38이 그대로 나왔다. 개편 대상은 법정동/행정코드다(`../data/README.md`의 ⚠ 절).

**(a-2) 법정동 코드 조회**: `ldongCode2` (2026-07 개편 확인용)
```
# 시도 목록 (16개, 12 신설, 광주 29·전남 46 폐지)
https://apis.data.go.kr/B551011/KorService2/ldongCode2?serviceKey={KEY}&numOfRows=100&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json
# 시군구 목록, 인천 신설구 확인
https://apis.data.go.kr/B551011/KorService2/ldongCode2?serviceKey={KEY}&numOfRows=100&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&lDongRegnCd=28&lDongListYn=Y
```
- **웰니스·의료관광에 넣을 코드는 여기서 확인한다.** `lDongSignguCd`는 3자리다(`28`+`275` = 인천 서해구).
- 개편이 궁금하면 이걸 팀에게 직접 돌려보게 하라, 1초면 나오고, 데이터가 살아 있다는 감각을 준다.

**(b) 지역별 장소 공급 개수**: `areaBasedList2` (★검증 핵심★)
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

## 3.55 지역별 방문자수 (DataLabService): ★ 일별 데이터

```
https://apis.data.go.kr/B551011/DataLabService/locgoRegnVisitrDDList?serviceKey={KEY}&numOfRows=1000&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&startYmd=20250901&endYmd=20250907
```

> ⚠ **가장 흔한 오해**: `touDivCd`·`signguCd`는 **요청 파라미터가 아니라 응답 필드**다.
> 요청에 넣으면 `INVALID_REQUEST_PARAMETER_ERROR`가 난다. **전국을 받아서 코드로 걸러낸다.**

- **필수: `startYmd`** (`YYYYMMDD`). 없으면 `NO_MANDATORY_REQUEST_PARAMETERS_ERROR1(startYmd)`.
  `endYmd`를 같이 준다. **일 단위**다(월 단위 아님).
- **응답 필드**: `signguCode`(행정 시군구) · `signguNm` · `daywkDivCd`/`daywkDivNm`(요일) ·
  `touDivCd`/`touDivNm` · `touNum`(명, 소수점 포함) · `baseYmd`
- **`touDivCd`**: `1` 현지인(제외) · `2` 외지인 · `3` 외국인 → **방문 = 2 + 3**
- **분량**: 하루 = **792행**(264 시군구 × 3 구분). `numOfRows=1000`이면 하루가 한 번에 들어온다.
  일주일이면 5,544행이니 `numOfRows=1000` × 6페이지 또는 하루씩 7번.

```python
# 한 주, 한 지역의 방문 = 외지인 + 외국인
rows = [r for r in items if r['signguCode'] == '11110' and r['touDivCd'] in ('2', '3')]
visit = sum(float(r['touNum']) for r in rows)
```

> **연간을 받으려 하지 마라.** 365일 × 792행 = 29만 행이다. 연간 값은 기준데이터
> (`../data/context_visitors.csv`, `asof 2025-06~2026-05`)에 이미 있다.
> 라이브는 **특정 기간의 요일·계절 패턴**을 볼 때 쓴다. 그게 이 API의 쓸모다.

## 3.56 기초지자체 중심 관광지 (LocgoHubTarService1)

```
https://apis.data.go.kr/B551011/LocgoHubTarService1/areaBasedList1?serviceKey={KEY}&numOfRows=100&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&baseYm=202509&areaCd=11&signguCd=11110
```
- 필수: `baseYm`(YYYYMM) + `areaCd` + `signguCd`: **행정코드**다(`../data/region_admin.csv`).
- 티맵 내비 기반. **그 시군구에서 다른 관광지와 가장 많이 연계 방문되는 관광지 100위.**
- 응답: `hubTatsNm`(관광지명) · `hubRank`(순위) · `hubCtgryLclsNm`/`hubCtgryMclsNm`(분류) ·
  `mapX`/`mapY`(좌표) · `signguNm`
- 쓸모: **프로토타입의 출발점 목록**. 좌표가 있어 지도에 바로 찍힌다. `TarRlteTarService1`(§3.8)과
  짝지으면 "중심 관광지 → 거기서 어디로 새는가"가 한 화면에 나온다.

## 3.57 두루누비 걷기여행길 (Durunubi)

```
# 코스 (실제로 쓸 것): 전국 144개
https://apis.data.go.kr/B551011/Durunubi/courseList?serviceKey={KEY}&numOfRows=200&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json
# 테마 4개 (남파랑길·해파랑길 등)
https://apis.data.go.kr/B551011/Durunubi/routeList?serviceKey={KEY}&numOfRows=10&pageNo=1&...
```
- ⚠ **서비스 경로에 `Service`가 안 붙는다**. `B551011/Durunubi/...`.
- ⚠ **지역 파라미터가 없다.** 전국 144개를 받아 응답의 **`sigun`**(예: `"부산 영도구"`)으로 거른다.
  코드가 아니라 **한글 지역명**이라 `regions.csv`의 `sigungu`와 문자열로 맞춰야 한다.
- 응답: `crsKorNm`(코스명) · `crsDstnc`(km) · `crsTotlRqrmHour`(분) · `crsLevel`(난이도) ·
  `crsCycle`(순환/비순환) · `crsSummary`·`crsTourInfo`·`travelerinfo`(교통편) · `gpxpath`(GPX)
- 쓸모: 뚜벅이·접근성(B3)·웰니스(E3) 주제. **소요시간·난이도가 숫자로 있어 코스 추천에 바로 쓴다.**

## 3.6 관광수요지수 3종 (데이터랩 지표를 API로) ★신규

> 데이터랩 '관광수요지수'를 API로 제공. **체류·소비를 이제 수동 다운 없이 받는다.**
> ⚠ 이 3종의 `areaCd`/`signguCd`는 **행정 코드**(서울 11, 구로 11530), TourAPI 지역코드와 다르다.
> 값은 **상대 지수**(예: 79.08), 절대 해석 금지, 지역 간 비교·순위로.

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
- ⚠ **지표코드를 안 넣으면 0건**이 나올 수 있다(반드시 지정). 단 **활성확인은 응답코드만 보므로** 0건이어도 `resultCode:0000`이면 통과다.

**호출 예 (소비)**: 같은 서비스의 다른 오퍼레이션
```
https://apis.data.go.kr/B551011/AreaTarDemDsService/areaTarExpDsList?serviceKey={KEY}&MobileOS=ETC&MobileApp=ftskill&_type=json&baseYm=202509&areaCd=11&signguCd=11530&tarExpDsIxCd=2201
```

**호출 예 (관광 자원 수요)**
```
https://apis.data.go.kr/B551011/AreaTarResDemService/areaTarSvcDemList?serviceKey={KEY}&numOfRows=10&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&baseYm=202509&areaCd=11&signguCd=11530
```

**호출 예 (관광 다양성)**
```
https://apis.data.go.kr/B551011/AreaTarDivService/areaTouDivList?serviceKey={KEY}&numOfRows=10&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&baseYm=202509&areaCd=11&signguCd=11530
```

**활용신청**: 세 API 각각 1회 (§1.5 목록 참조 · 자동승인·무료, 개발계정 1,000회/일).

## 3.7 관광지 집중률·방문 추이 예측 (TatsCnctrRateService): 참가자 조건부
`GET https://apis.data.go.kr/B551011/TatsCnctrRateService/tatsCnctrRatedList`
- 필수: `areaCd`(행정 시도) + `signguCd`(행정 시군구). **`baseYm`/`baseYmd`를 넣으면 INVALID 에러**: 넣지 않는다.
- 응답: `tAtsNm`(관광지) · `baseYmd` · `cnctrRate`. 조회일부터 **30일치 예보**, 가장 붐빌 때를 100으로 본 **상대 지수**(KT 통신데이터 기반).
- 예) 종로구 = 관광지 34곳 × 30일(1,020행). 가회민화박물관 월 57.9 → 토 97.8 → 월 58.8.
- **카드 근거로 쓰지 않는다**. 조회 시점마다 값이 바뀌어 고정 근거가 못 된다.
- 쓰임: (1) 쏠림·계절·분산 주제 팀의 **검증 보조**, (2) 프로토타입의 **혼잡도 기능**.
- ⚠ **CORS 불가**: `apis.data.go.kr`은 CORS 헤더를 주지 않는다 → 브라우저(HTML)에서 직접 fetch하면 실패한다.
  `file://`로 연 HTML은 옆에 둔 로컬 `data.json`을 fetch하는 것도 막힌다(origin `null`) → **별도 JSON 파일도 쓰지 않는다.**
  프로토타입은 **터미널에서 한 번 호출 → 결과를 HTML 스크립트에 `const DATA = [...]` 로 인라인**한다. 파일 하나로 열리고, 키는 `.env`에만 남는다(HTML에 넣으면 업로드 시 노출).
  실서비스라면 정적 데이터가 아니라 **서버가 API를 대신 호출**하는 구조로 간다(CORS·키 문제가 함께 사라진다). 피칭에서 이 한계를 짚으면 좋다.

## 3.8 관광지별 연관 관광지 (TarRlteTarService1): 카드 근거

```
https://apis.data.go.kr/B551011/TarRlteTarService1/areaBasedList1?serviceKey={KEY}&numOfRows=10&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&baseYm=202509&areaCd=11&signguCd=11110
```
- 필수: `baseYm`(202509) + `areaCd` + `signguCd`(**시군구 필수**). 둘 다 **행정코드**다.
- 티맵 내비 기반 관광지→연관 관광지 랭킹. 파생: **동선 유출비율**(연관지가 타 시군구인 비율), 연관 관광지 수, 최다 유출처.
- 전국 252개 행정 시군구 수집 → 234개 TourAPI 지역으로 집계 → `../data/context_rlte.csv` (평균 유출 40.8%). 도시 구는 구조상 높으니 **도시·시군을 나눠 비교**한다.

## 3.9 대상별 공급 5종 (2026-08-13 실호출 확인)
| 대상 | 엔드포인트 | 지역 지정 | 비고 |
|---|---|---|---|
| 무장애 | `KorWithService2/areaBasedList2` | `areaCode`·`sigunguCode` (TourAPI) | 9,948건 |
| 반려동물 | **`KorPetTourService2`**`/areaBasedList2` | `areaCode`·`sigunguCode` (TourAPI) | 서울 54건. 응답은 KorService2와 같은 스키마 |
| 웰니스 | `WellnessTursmService/areaBasedList` | lDong 코드 | **`langDivCd=KOR` 필수** · 174건, 137개 시군구 0건 |
| 캠핑 | `GoCamping/basedList` | 도/시군구 **이름** | 3,090건 |
| 의료관광 | `MdclTursmService/areaBasedList` | `lDongRegnCd`·`lDongSignguCd` | **`langDivCd` 필수** · KOR 326건, 서울 65% 편중 |
→ `../data/context_niche.csv`

**최소 호출 5종** (키만 넣으면 그대로 실행된다)
```
# 무장애, KorService2와 같은 스키마
https://apis.data.go.kr/B551011/KorWithService2/areaBasedList2?serviceKey={KEY}&numOfRows=1&pageNo=1&MobileOS=ETC&MobileApp=ftskill&arrange=A&areaCode=1&sigunguCode=1&_type=json

# 반려동물, Service'2'다
https://apis.data.go.kr/B551011/KorPetTourService2/areaBasedList2?serviceKey={KEY}&numOfRows=1&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&areaCode=1

# 웰니스, langDivCd 필수, 지역은 lDong(법정동) 코드
https://apis.data.go.kr/B551011/WellnessTursmService/areaBasedList?serviceKey={KEY}&numOfRows=10&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&langDivCd=KOR&lDongRegnCd=11&lDongSignguCd=11110

# 캠핑, 지역 파라미터 없이 전국. 지역은 응답의 do/sigungu 이름으로 거른다
https://apis.data.go.kr/B551011/GoCamping/basedList?serviceKey={KEY}&numOfRows=10&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json

# 의료관광, langDivCd 필수, 지역은 lDong 코드
https://apis.data.go.kr/B551011/MdclTursmService/areaBasedList?serviceKey={KEY}&numOfRows=10&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&langDivCd=KOR&lDongRegnCd=11&lDongSignguCd=11110
```
> 웰니스·의료관광은 전국 건수가 적어 **특정 시군구에서 0건이 정상**이다. 활성확인은 건수가 아니라 `resultCode:0000`으로 판정한다.
>
> 🚨 **웰니스·의료관광에 `../data/region_admin.csv`의 코드를 그대로 넣지 마라.** 그 CSV는 옛 행정코드고,
> 이 둘만 새 법정동 코드로 넘어갔다. `28260`(인천 서구)·`29110`(광주)·`46710`(담양)은 **0건**이 온다.
> 새 코드는 `ldongCode2`(§3.5 a-2)로 조회하거나 `../data/README.md`의 표를 본다.
> `lDongSignguCd`는 **3자리**다(`28`+`275` = 서해구). 나머지 API는 `region_admin.csv`가 여전히 맞다.
>
> 🚨 **고캠핑은 지역 이름이 새 것으로 왔다.** `regions.csv`의 `전라남도`로 매칭하면 **0건**이다.
> 응답은 `전남광주통합특별시`로 온다. 인천도 중구·동구·서구가 없고 **제물포구·영종구·검단구·서해구**로 온다.
> (두루누비는 반대로 아직 `전남 강진군` 같은 옛 이름이다. 둘을 같이 쓰면 기준을 따로 잡아야 한다.)

- ⚠ **2026-07-01 개편이 실제로 적용된 건 이 두 서비스뿐이다**(2026-08-19 실측). 웰니스·의료관광은 lDong(법정동) 코드로 부르는데 **옛 코드는 죽었다**. `lDongRegnCd=29`(광주)·`46`(전남)·`lDongSignguCd=260`(인천 서구)은 전부 **0건**. 새 코드로 불러야 한다: 통합특별시 `12`(담양군 `12`-`710`), 인천 서해구 `28`-`275`·검단구 `28`-`290`. 전체 목록은 `../data/README.md`의 **⚠ 2026-07-01 행정구역 개편** 절, 또는 `KorService2/ldongCode2`로 직접 조회.
> 의료관광은 서울 편중(65%)이 심해 **기준데이터·카드에는 안 쓴다.** 카드 E6를 고른 팀만 라이브로 받아 검증한다.
