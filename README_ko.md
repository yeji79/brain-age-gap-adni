# 알츠하이머병 스펙트럼의 구조 MRI 기반 Brain Age Gap

> 구조 MRI로 정상적인 뇌 노화 패턴을 학습하고, Brain Age Gap(BAG)을 통해 알츠하이머병 임상 스펙트럼의 구조적 편차를 평가하는 재현 가능한 머신러닝 프로젝트입니다.

[English README](README.md)

공개용 문서: [프로젝트 개요](docs/PROJECT_OVERVIEW_KO.md), [연구 설계 및 로드맵](docs/RESEARCH_DESIGN_AND_ROADMAP_KO.md)

## 프로젝트 한눈에 보기

이 프로젝트는 ADNI의 ADSP PHC ComBat 보정 FreeSurfer 구조 MRI ROI 데이터를 사용합니다. 인지적으로 정상인 참가자(CN)만으로 실제 나이를 예측하는 뇌 나이 모델을 학습한 뒤, 고정된 모델을 CN·MCI·Dementia 집단에 적용해 나이 편향이 보정된 Brain Age Gap(BAG)을 계산합니다.

**주 연구 질문:** 실제 나이, 성별, 교육연수, 두개강 용적을 고려한 뒤에도 MCI와 Dementia 집단은 CN보다 높은 BAG를 보이는가?

**부차 연구 질문:** PET가 있는 하위표본에서, 임상 진단과 인구통계학적 공변량을 고려한 뒤에도 BAG는 Amyloid 양성과 연관되는가?

## 입문자를 위한 노트북 설계

이 프로젝트의 노트북은 결과만 빠르게 내는 코드를 모아 둔 파일이 아닙니다. 바이오데이터와 머신러닝을 처음 접하는 사람이 분석의 이유와 순서를 따라갈 수 있도록 각 단계 앞에 Markdown 설명을 넣었습니다.

- **무엇을 하는가:** 데이터 결합, 전처리, 모델 학습, 검증, 통계 분석의 작업을 명시합니다.
- **왜 필요한가:** RID 누수 방지, CN-only 정상 기준, 나이 편향 보정처럼 결과의 신뢰도를 지키는 이유를 설명합니다.
- **어떻게 읽는가:** MAE, BAG, ANCOVA, 상호작용, ROI 중요도의 생물학적 의미와 해석 한계를 함께 설명합니다.
- **어떻게 확장하는가:** MRI MVP → Amyloid PET → normative ROI → 뇌 표면 시각화 순서와 각 단계의 완료 기준을 남깁니다.

따라서 노트북은 학습 자료이면서도, 각 분석 결정을 재현 가능하게 기록하는 연구 작업 문서입니다.

## 현재 진행 상태

**마지막 업데이트: 2026-08-20**

| 단계 | 현재 상태 | 다음 확인 사항 |
|---|---|---|
| 구조 MRI MVP | 실행 및 결과 해석 완료 | held-out CN test를 기준군으로 제한한 민감도 분석 |
| Amyloid PET 확장 | 실행 노트북 작성 완료 | 로컬 실행 후 MRI–PET 정렬 표본·회귀 결과 확인 |
| Normative ROI | 설계 완료 | PET 결과 확정 뒤 구현 |
| 뇌 표면 시각화 | 설계 완료 | ROI·민감도 분석 정의 확정 후 구현 |

실제 결과가 추가되면 날짜, 분석 표본 수, 모형, 핵심 효과크기를 함께 갱신합니다.

## Brain Age Gap이란?

```text
BAG = 나이 편향이 보정된 예측 뇌 나이 − 실제 나이
```

양의 BAG는 해당 참가자의 구조 MRI 패턴이 같은 실제 나이의 CN 기준보다 더 고령의 정상 뇌 구조와 유사하다는 뜻입니다. BAG는 정상 기준 모델에서 벗어난 정도를 나타내는 연구 지표이며, **치매 진단, 인과적 바이오마커, 미래 위험 점수, 개인 노화 속도의 직접 측정값이 아닙니다.**

## 연구 설계

```text
PHC 구조 MRI + 인구통계 + 임상 진단
                    ↓
참가자(RID)별 가장 이른 index MRI 한 건 선택
                    ↓
CN만 RID 기준 train / validation / 독립 test로 분할
                    ↓
Dummy 기준선 + Ridge 주 모델 + 비선형 비교 모델
                    ↓
validation CN에서만 나이 편향 보정식 적합
                    ↓
고정된 모델을 CN / MCI / Dementia에 적용
                    ↓
진단군 BAG 비교(ANCOVA)
                    ↓
선택 확장: Amyloid PET, normative ROI Z-score, 민감도 분석
```

### 잘못된 결과를 막기 위한 원칙

- **RID 단위 분할:** 같은 `RID`가 train, validation, test 중 둘 이상에 들어가지 않습니다.
- **CN-only 정상 기준:** MCI와 Dementia는 모델 학습, 하이퍼파라미터 선택, 나이 편향 보정에 사용하지 않습니다.
- **Train 기반 전처리:** ROI 결측률 필터, 중앙값 대치, 표준화, 인코딩은 train 내부에서만 학습합니다.
- **보정식 동결:** 나이 편향 보정식은 validation CN에서만 만들고, 이후 test와 임상군에는 같은 보정식을 적용합니다.
- **독립 test 보존:** CN test는 최종 성능을 보고할 때만 사용합니다.
- **신중한 해석:** ROI 중요도는 나이 예측 기여도이며 질병 원인이나 독립 위험인자를 뜻하지 않습니다.

## 데이터

이 저장소에는 ADNI 원본 데이터나 참가자 수준 처리 데이터가 포함되지 않습니다. 데이터 접근과 사용은 [ADNI Data Use Agreement](https://adni.loni.usc.edu/)를 따라야 합니다.

로컬 ADNI 데이터는 아래와 같이 `data/adni_all/` 아래에 둡니다.

```text
data/adni_all/
├── ADSP_PHC/
│   ├── ADSP_PHC_T1_FS_22Jan2026.csv
│   ├── ADSP_PHC_T1_FS_DATADIC_22Jan2026.csv
│   └── ADSP_PHC_PET_Amyloid_Simple_22Jan2026.csv  # PET 부차 분석
├── Assessments/
│   └── DXSUM_22Jan2026.csv
├── Subject_Characteristics/
│   └── PTDEMOG_22Jan2026.csv
```

| 데이터 | 프로젝트에서의 역할 |
|---|---|
| `ADSP_PHC_T1_FS` | 주 MRI 입력. ComBat 보정 피질 두께·피질 부피·피질하 ROI, MRI 시점 나이·성별·진단·촬영일 |
| `ADSP_PHC_T1_FS_DATADIC` | ROI의 해부학적 이름과 변수 메타데이터 |
| `PTDEMOG` | 교육연수 공변량 |
| `DXSUM` | PHC 진단이 없는 경우 MRI 날짜와 가까운 임상 진단을 보강 |
| `ADSP_PHC_PET_Amyloid_Simple` | PET QC, Amyloid 음성/양성 상태, Centiloid, PET 촬영일 |

MVP에서는 각 `RID`에서 가장 이른 T1 MRI 한 건(`index MRI`)을 선택합니다. MRI 시점 나이는 `PHC_Age_T1`을 사용하며, 진단은 PHC 값을 우선 사용하고 결측일 때만 ±180일 이내 가장 가까운 DXSUM 진단으로 채웁니다.

## 모델과 평가

주 모델은 **Ridge regression**입니다. 구조 MRI ROI는 서로 강하게 연관되어 있는데, Ridge는 이런 상관된 변수를 안정적으로 다루며 표 형식 신경영상 데이터에서 해석과 재현이 쉽습니다. `DummyRegressor`는 MRI 없이 평균 나이만 예측하는 기준선이고, `ExtraTreesRegressor`는 비선형 비교 모델입니다.

CN 참가자는 `RID` 기준으로 다음과 같이 나뉩니다.

- train 70% — 5-fold `GroupKFold` 튜닝과 파이프라인 학습
- validation 15% — 나이 편향 보정만 수행
- 독립 test 15% — 최종 성능 평가만 수행

독립 CN test에서 보고할 지표는 다음과 같습니다.

- MAE, RMSE(년 단위)
- R²
- Dummy 기준선 대비 MAE 개선량
- 보정된 BAG와 실제 나이의 상관

주 임상 분석 모형은 다음과 같습니다.

```text
BAG ~ diagnosis × AGE + sex + education + eTIV
```

현재 MVP는 ComBat ROI를 제공된 값 그대로 사용합니다. 추가적인 단순 `volume / ICV` 비율 변환은 하지 않으며, `EstimatedTotalIntraCranialVol_combat`은 ANCOVA 공변량으로 고려합니다.

## 폴더 구조

```text
brain-age-gap-adni/
├── data/
│   ├── adni_all/              # 로컬 ADNI 원본 — Git 제외
│   └── processed/             # 참가자 수준 처리 파일 — Git 제외
├── docs/
│   ├── PROJECT_OVERVIEW_KO.md
│   └── RESEARCH_DESIGN_AND_ROADMAP_KO.md
├── notebooks/
│   ├── 01_mri_brain_age_bag_mvp.ipynb
│   └── 02_mri_amyloid_pet_bag.ipynb
├── results/
│   ├── figures/               # 보고용 집단 수준 그림
│   └── tables/                # 보고용 집단 수준 결과표
├── src/                       # 재사용 함수 (프로젝트 진행에 따라 추가)
├── LICENSE
├── README.md
└── README_ko.md
```

## 로컬 VS Code에서 실행하기

1. ADNI에서 필요한 파일을 내려받아 위 폴더 구조에 맞게 배치합니다.
2. VS Code에서 `brain-age-gap-adni` 폴더를 열고 Python 커널을 선택합니다.
3. [`01_mri_brain_age_bag_mvp.ipynb`](notebooks/01_mri_brain_age_bag_mvp.ipynb)를 위에서부터 실행합니다.
4. 이어서 PET 확장 분석인 [`02_mri_amyloid_pet_bag.ipynb`](notebooks/02_mri_amyloid_pet_bag.ipynb)를 실행합니다.

두 노트북은 `data/adni_all` 폴더를 기준으로 프로젝트 루트를 자동 탐색하므로, 개인 컴퓨터의 절대 경로를 직접 입력할 필요가 없습니다.

노트북은 `pyarrow` 의존성을 피하기 위해 Parquet 대신 CSV를 사용합니다.

## 결과 파일 저장 규칙

실행 결과는 분석 ID와 버전이 포함된 하위 폴더에 저장합니다. 예를 들어 MRI MVP의 첫 확정 실행은 다음처럼 저장합니다.

```text
results/
├── tables/01_mri_bag_mvp_20260820_v1/
└── figures/01_mri_bag_mvp_20260820_v1/
```

분석 정의가 달라지면 `v2`처럼 버전을 올리고, 단순 재실행은 같은 버전 폴더를 사용합니다. 자세한 규칙은 [results/README.md](results/README.md)를 참고하세요.

## 진행 로드맵

| 단계 | 목표 | 상태 |
|---|---|---|
| MVP | PHC 데이터 감사, CN 뇌 나이 모델, 편향 보정, 독립 test, 진단군 ANCOVA | 진행 중 |
| 확장 1 | Amyloid PET 날짜 정렬 분석 | 계획됨 |
| 확장 2 | CN 기준 normative ROI Z-score profile | 계획됨 |
| 확장 3 | ROI 집합, 날짜 창, eTIV 처리, 모델 선택 민감도 분석 | 계획됨 |
| 확장 4 | 분석 정의 확정 후 FDR 보정 피질 뇌 표면 시각화 | 계획됨 |
| 포트폴리오 | `src/` 모듈화, 완성 그림·표, 재현성 문서 | 계획됨 |

## 해석과 한계

- 본 연구는 횡단면 관찰 분석이므로 인과관계나 개인별 노화 속도를 증명할 수 없습니다.
- `Dementia`는 임상 진단 라벨이며, 병리학적으로 확정된 알츠하이머병과 동일하게 다루지 않습니다.
- Amyloid PET 분석은 부차 분석이며 PET 보유 하위표본 선택 편향이 있을 수 있습니다.
- ComBat 조화는 ADSP PHC가 제공한 것입니다. 다기관 기술적 차이를 줄이는 장점이 있지만, 조화 절차 자체는 연구의 제한점으로 투명하게 서술합니다.
- ROI 수준 결과를 voxel 수준 위치 정보처럼 해석할 수 없습니다.
- ADNI 내부 test 성능이 다른 병원이나 인구집단에서도 동일하리라는 보장은 없습니다.

## ADNI 데이터 인정문

본 프로젝트에 사용된 데이터는 Alzheimer's Disease Neuroimaging Initiative(ADNI) 데이터베이스에서 얻었습니다. ADNI 연구진은 ADNI의 설계·수행 및/또는 데이터 제공에 기여했지만, 이 프로젝트의 분석이나 작성에는 참여하지 않았습니다. 전체 인정문과 인용 안내는 [ADNI](https://adni.loni.usc.edu/)에서 확인할 수 있습니다.

## 라이선스

분석 코드는 이 저장소의 [MIT License](LICENSE)를 따릅니다. ADNI 데이터는 별도의 ADNI Data Use Agreement를 따르며, 이 저장소를 통해 재배포해서는 안 됩니다.
