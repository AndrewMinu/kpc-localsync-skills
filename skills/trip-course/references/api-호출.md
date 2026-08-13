# trip-course API 호출 가이드

> `trip-course` ②단계에서 쓰는 실제 호출 명령. 키는 `DATA_GO_KR_SERVICE_KEY`(공공데이터포털 인증키 1개). **키를 채팅·파일에 그대로 남기지 않는다.**
>
> ⚠ 아래 서비스는 **각각 활용신청이 필요하다** — 키가 있어도 신청 안 한 서비스는 `SERVICE_ACCESS_DENIED`가 온다. 부트캠프 참가자는 `$api-select`에서 연다. 그 밖의 사용자는 data.go.kr에서 서비스 이름으로 검색해 [활용신청]을 누른다.

## 1. 지역 코드 확인 — `areaCode2`

```bash
# 시도 목록
curl -s "https://apis.data.go.kr/B551011/KorService2/areaCode2?serviceKey=$DATA_GO_KR_SERVICE_KEY&numOfRows=20&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json"
# 특정 시도의 시군구 목록 (예: 강원=32)
curl -s "https://apis.data.go.kr/B551011/KorService2/areaCode2?serviceKey=$DATA_GO_KR_SERVICE_KEY&numOfRows=30&pageNo=1&MobileOS=ETC&MobileApp=ftskill&areaCode=32&_type=json"
```
서울=1, 인천=2, 대전=3, 대구=4, 광주=5 … 강원=32.

## 2. 장소 후보 수집 — `areaBasedList2`

```bash
curl -s "https://apis.data.go.kr/B551011/KorService2/areaBasedList2?serviceKey=$DATA_GO_KR_SERVICE_KEY&numOfRows=30&pageNo=1&MobileOS=ETC&MobileApp=ftskill&arrange=A&contentTypeId=12&areaCode={시도}&sigunguCode={시군구}&_type=json"
```

contentTypeId — 취향에 맞는 것만 골라 호출한다:

| 코드 | 유형 | 언제 |
|---|---|---|
| 12 | 관광지 | 기본 |
| 14 | 문화시설 | 역사·전시 취향, 비 오는 날 실내 대안 |
| 15 | 행사·축제 | 방문 시기에 열리는 것 (⚠ 등록된 예정 행사만 나옴) |
| 28 | 레포츠 | 액티비티 취향 |
| 32 | 숙박 | 1박 이상일 때 |
| 39 | 음식점 | 식사 동선 |

응답에서 쓸 것: `title`(이름), `addr1`(주소), `mapx`/`mapy`(좌표 — 동선 인접 판단에 사용), `contentid`.

## 3. 혼잡도 예보 — 집중률 API (30일)

```bash
curl -s "https://apis.data.go.kr/B551011/TatsCnctrRateService/tatsCnctrRatedList?serviceKey=$DATA_GO_KR_SERVICE_KEY&numOfRows=1000&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&areaCd={행정시도}&signguCd={행정시군구}"
```
⚠ `baseYm`을 넣으면 에러다. 응답: `tAtsNm`(관광지명)·`baseYmd`(날짜)·`cnctrRate`(100=가장 붐빌 때). **호출일부터 30일치 예보**라 방문일 값을 골라 쓴다.
⚠ 여기의 지역 코드는 **행정표준코드**(TourAPI 코드와 다름). 모르면 지역명으로 행정코드를 검색해 확인한다.

## 4. 운영시간·휴무 확인 — `detailIntro2` (같은 서비스, 추가 신청 불필요)

```bash
curl -s "https://apis.data.go.kr/B551011/KorService2/detailIntro2?serviceKey=$DATA_GO_KR_SERVICE_KEY&MobileOS=ETC&MobileApp=ftskill&contentId={contentid}&contentTypeId={유형코드}&_type=json"
```
유형별 필드가 다르다: 관광지 `usetime`(이용시간)·`restdate`(쉬는날)·`parking`, 음식점 `opentimefood`·`restdatefood`, 문화시설 `usetimeculture`·`restdateculture`. **코스에 확정으로 넣을 후보만** 호출한다(전 후보 호출은 낭비). 방문일이 쉬는날과 겹치면 그 자리에서 뺀다.

## 5. 동행 조건부 API (같은 키, 활용신청 따로 · 2026-08-13 실호출 확인)

```bash
# 반려동물 — ⚠ Service'2' 다. KorPetTourService(2 없이)는 존재하지 않는 서비스다
curl -s "https://apis.data.go.kr/B551011/KorPetTourService2/areaBasedList2?serviceKey=$DATA_GO_KR_SERVICE_KEY&numOfRows=50&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&areaCode={시도}&sigunguCode={시군구}"

# 무장애 — KorService2와 파라미터가 같다
curl -s "https://apis.data.go.kr/B551011/KorWithService2/areaBasedList2?serviceKey=$DATA_GO_KR_SERVICE_KEY&numOfRows=50&pageNo=1&MobileOS=ETC&MobileApp=ftskill&_type=json&areaCode={시도}&sigunguCode={시군구}"
```

동반 가능/접근 가능 장소만 후보에 남긴다. ⚠ 같은 키로 쓰지만 **활용신청을 따로** 해야 한다.

## 6. 날씨·일몰 (⚠ 별도 활용신청 — TourAPI 신청과 별개다)

엔드포인트는 확인됐다(2026-08-13). 다만 **기관이 달라 활용신청을 따로** 해야 한다.

```bash
# 기상청 단기예보 (기관코드 1360000)
curl -s "https://apis.data.go.kr/1360000/VilageFcstInfoService_2.0/getVilageFcst?serviceKey=$DATA_GO_KR_SERVICE_KEY&numOfRows=100&pageNo=1&dataType=JSON&base_date={YYYYMMDD}&base_time=0500&nx={격자X}&ny={격자Y}"

# 한국천문연구원 출몰시각 (기관코드 B090041 · XML만 반환 · location은 URL 인코딩)
curl -s -G "https://apis.data.go.kr/B090041/openapi/service/RiseSetInfoService/getAreaRiseSetInfo" \
  --data-urlencode "serviceKey=$DATA_GO_KR_SERVICE_KEY" --data-urlencode "locdate={YYYYMMDD}" --data-urlencode "location=서울"
```

- 기상청은 **격자 좌표(nx/ny)** 를 쓴다. 위경도가 아니다 — 지역별 격자값은 포털 문서의 엑셀표에 있다.
  `base_time`은 발표시각(0200·0500·0800…)만 유효하다.
- **`SERVICE_KEY_IS_NOT_REGISTERED_ERROR`가 오면 활용신청이 안 된 것이다** (경로가 틀리면
  `NO_OPENAPI_SERVICE_ERROR`가 온다 — 둘을 구분해서 안내한다).
- 방문일이 예보 범위(3일) 밖이면 호출하지 말고 계절 지식으로 대신하되 **"예보 아님"** 을 표기한다.

## 7. 웹검색 보강 (API 밖의 것)

API는 **존재·좌표·운영시간·혼잡**을 주고, 웹검색은 **평판·최신 영업 여부·행사**를 준다. 확정 장소는 "최근에도 영업 중인가"를 한 번 검색으로 확인하고, 방문일의 지역 행사·팝업도 훑는다. 검색이 안 되는 환경이면 `[놓친 것]`에 미확인 항목을 적는다.

## 사용 원칙

- 취향과 무관한 유형까지 다 부르지 않는다 — 필요한 2~4개 유형만.
- 좌표(`mapx`/`mapy`)로 서로 가까운 장소끼리 묶어 동선을 만든다(대략 0.01도 ≈ 1km).
- 혼잡 90 이상은 시간대 변경 또는 대안 제시. 41 같은 낮은 값은 "한산" 근거로 활용.
- 호출이 실패하면(키 미활성 등) 조용히 지어내지 말고 사용자에게 알리고 ⚠실측 아님 모드로 전환한다. 신규 키는 활성까지 최대 1시간 걸린다.
- 호출 순서는 **넓게 → 좁게**: 목록(areaBasedList2)으로 후보를 넓게 받고, 코스에 들어갈 것만 상세(detailIntro2)·검색으로 좁혀 확인한다.
