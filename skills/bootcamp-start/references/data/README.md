# 기준데이터 — 전국 234개 시군구

주제·지역 대화에서 **실제 숫자로 보여줄 때** 쓴다. 모든 파일은 `region_id`(예: 36-17=통영시)로 조인된다. `regions.csv`가 지역 이름↔코드 사전이다.

| 파일 | 담은 것 |
|---|---|
| supply_tourapi.csv | 공급 8종: 관광지·문화·축제·코스·레포츠·숙박·쇼핑·음식 |
| pet_tourapi.csv | 반려동물 동반 가능 장소 |
| context_niche.csv | 무장애·웰니스·캠핑 (0 = 진짜 0, 그게 곧 공백) |
| context_visitors.csv | 방문(천명)·외국인비중(%)·월별편차(%) |
| context_stay_spend.csv | 숙박비중·외지인소비 ⚠상대지수 |
| context_demand_index.csv | 서비스수요·문화자원수요·연령/소비/국제 다양성 ⚠상대지수 |
| context_rlte.csv | 동선 유출비율(%)·연관관광지수·최다유출처 |
| context_population.csv / popdecline / 6chsanup | 인구·인구감소지역·6차산업 |

⚠ **지수는 %도 금액도 아니다** — 지역 간 비교로만. 공급은 등록 건수지 실제 총량이 아니다.
