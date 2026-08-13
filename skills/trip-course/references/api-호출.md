# trip-course API 호출 가이드

> `trip-course` ②단계에서 쓰는 실제 호출 명령. 키는 `DATA_GO_KR_SERVICE_KEY`(공공데이터포털 1개 키, `$tourapi-key`로 발급). **키를 채팅·파일에 그대로 남기지 않는다.**

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

## 사용 원칙

- 취향과 무관한 유형까지 다 부르지 않는다 — 필요한 2~4개 유형만.
- 좌표(`mapx`/`mapy`)로 서로 가까운 장소끼리 묶어 동선을 만든다(대략 0.01도 ≈ 1km).
- 혼잡 90 이상은 시간대 변경 또는 대안 제시. 41 같은 낮은 값은 "한산" 근거로 활용.
- 호출이 실패하면(키 미활성 등) 조용히 지어내지 말고 사용자에게 알리고 ⚠실측 아님 모드로 전환한다. 신규 키는 활성까지 최대 1시간 걸린다.
