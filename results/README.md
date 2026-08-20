# 결과 파일 저장 규칙

이 폴더에는 원본 ADNI 자료나 RID/PTID/촬영일 같은 참가자 식별자가 없는 **집계 결과**만 저장한다. 참가자 수준 처리 파일은 `data/processed/`에만 두며 Git에 올리지 않는다.

## 폴더 이름

모든 실행 결과는 아래 형식을 사용한다.

```text
results/
├── tables/<analysis_id>_<run_date>_<version>/
└── figures/<analysis_id>_<run_date>_<version>/
```

| 구성요소 | 규칙 | 예시 |
|---|---|---|
| `analysis_id` | 실행 노트북과 같은 순서·목적 | `01_mri_bag_mvp`, `02_amyloid_pet_bag` |
| `run_date` | 분석 정의를 확정하거나 변경한 날짜 | `20260820` |
| `version` | 같은 분석 정의의 버전 | `v1`, `v2` |

예시:

```text
results/tables/01_mri_bag_mvp_20260820_v1/model_performance_heldout_cn_test.csv
results/figures/01_mri_bag_mvp_20260820_v1/bag_diagnosis_age_interaction.png
```

## 파일 이름

분석 ID는 폴더에 이미 있으므로, 파일명은 결과의 내용과 분석 대상을 설명한다.

| 종류 | 권장 파일명 |
|---|---|
| 모델 비교 | `model_selection_cv.csv` |
| 독립 CN test 성능 | `model_performance_heldout_cn_test.csv` |
| 편향 보정 계수 | `bag_bias_correction_parameters.csv` |
| 진단군 상호작용 ANCOVA | `ancova_diagnosis_by_age_interaction_hc3.csv` |
| 나이별 CN 대비 | `age_specific_bag_contrasts_vs_cn_hc3.csv` |
| PET 정렬 흐름 | `amyloid_pet_alignment_flow.csv` |
| PET 상호작용 ANCOVA | `amyloid_pet_bag_ancova_hc3.csv` |
| 정적 그림 | 분석 내용을 설명하는 `snake_case.png` |

## 버전 관리 원칙

- **같은 코드·같은 데이터·같은 분석 정의를 재실행:** 같은 폴더에 덮어쓴다.
- **분석 규칙 변경:** 예를 들어 날짜 창, ROI 집합, 공변량, 기준군 정의를 바꾼 경우 새 `v2` 폴더를 만든다.
- **결과 해석 변경만 있는 경우:** 결과 파일을 복제하지 않고 문서의 업데이트 날짜와 설명을 갱신한다.
- **공개 전:** 결과 CSV의 열 이름에 `RID`, `PTID`, `date`, `scan_date`, `visit`가 포함되지 않는지 확인한다.
