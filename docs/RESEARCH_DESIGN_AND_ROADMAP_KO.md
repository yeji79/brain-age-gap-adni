# 연구 설계 및 확장 로드맵

**마지막 업데이트: 2026-08-20**

## 1. 연구 목적

이 프로젝트는 인지적으로 정상인(CN) 참가자의 구조 MRI로 정상적인 뇌 노화 패턴을 학습하고, MCI 및 Dementia 집단의 구조적 편차를 Brain Age Gap(BAG)으로 정량화한다.

핵심 질문은 다음과 같다.

1. CN만으로 학습한 구조 MRI 기반 뇌 나이 모델은 실제 나이를 평균 나이 기준선보다 잘 예측하는가?
2. 같은 실제 나이에서 MCI와 Dementia는 CN보다 높은 BAG를 보이는가?
3. MRI와 가까운 시점의 Amyloid PET 양성 상태는 BAG와 연관되는가?
4. 이후 정상 기준에서 벗어난 ROI와 BAG 연관 피질 영역을 해부학적으로 어떻게 요약할 수 있는가?

이 연구는 관찰 코호트의 집단 수준 분석이다. 개인 진단, 개인별 미래 위험, 인과관계, 개인의 뇌 노화 속도를 판정하지 않는다.

## 2. 데이터와 분석 단위

| 역할 | 파일 | 사용 방식 |
|---|---|---|
| 구조 MRI | `ADSP_PHC_T1_FS_22Jan2026.csv` | ComBat 보정 FreeSurfer ROI, MRI 시점 나이·성별·진단·촬영일 |
| 교육연수 | `PTDEMOG_22Jan2026.csv` | 임상군 비교 공변량 보강 |
| 진단 보강 | `DXSUM_22Jan2026.csv` | PHC 진단 결측 시 MRI 날짜와 가까운 임상 진단 연결 |
| Amyloid PET | `ADSP_PHC_PET_Amyloid_Simple_22Jan2026.csv` | PET QC, Amyloid 음성/양성, PET 날짜, Centiloid |
| MRI 변수 사전 | `ADSP_PHC_T1_FS_DATADIC_22Jan2026.csv` | ROI의 해부학적 명칭·측정값 설명 |

분석 단위는 참가자(RID)별 가장 이른 유효 T1 MRI 한 건인 **index MRI**다. 반복 방문이 학습과 평가에 동시에 들어가는 누수를 막기 위한 규칙이다.

## 3. 현재 MVP 설계

```text
PHC 구조 MRI 로드
→ RID별 index MRI 선택
→ 진단·교육연수 결합 및 EDA
→ CN-only train / validation / test 분할
→ train 기반 ROI 결측 처리·표준화
→ Dummy / Ridge / Extra Trees 비교
→ validation 기반 나이 편향 보정
→ held-out CN test 평가
→ 전체 코호트 BAG 계산
→ diagnosis × age 상호작용 분석
→ ROI permutation importance
```

### 누수 방지 원칙

- CN만 뇌 나이 모델 학습과 보정에 사용한다.
- 같은 RID는 train, validation, test 중 한 분할에만 둔다.
- ROI 선택, 중앙값 대치, 표준화, 성별 인코딩은 train 데이터에서만 학습한다.
- 보정식은 validation CN에서만 추정한 뒤 동결한다.
- MCI와 Dementia는 동결된 모델의 BAG 계산 및 임상군 비교에서만 사용한다.

### 모델과 평가

주 모델은 Ridge Regression이다. 상관이 높은 MRI ROI에서 안정적이고 해석이 비교적 쉽기 때문이다. DummyRegressor는 평균 나이 기준선이고, Extra Trees는 비선형 관계를 확인하는 비교 모델이다.

예측 성능은 held-out CN test에서 MAE, RMSE, R², Dummy 대비 MAE 개선으로 평가한다. BAG는 validation 보정식으로 만든 보정 예측 나이에서 실제 나이를 뺀 값이며, 보정 목적은 BAG와 실제 나이의 인위적 관계를 줄이는 것이다.

## 4. 현재 MRI MVP 결과

현재 실행 기록에서 Ridge의 held-out CN test raw MAE는 3.943년이고 Dummy 기준선보다 1.450년 낮았다. Validation 기반 보정 뒤 BAG와 실제 나이의 상관은 -0.737에서 -0.102로 감소했다.

진단군 분석에서는 `diagnosis × AGE` 상호작용이 유의했다. 따라서 모든 연령대에 하나의 진단군 평균 차이를 적용하지 않고, 기준 나이별 CN 대비 조정 BAG 차이를 제시한다. 75세에서 MCI와 Dementia의 조정 BAG는 CN보다 각각 4.63년과 11.68년 높았다.

이 수치는 현재 MVP 실행 기록의 집단 수준 결과다. 최종 보고에서는 CN 기준군을 held-out CN test로 제한한 민감도 분석과 동일 방향인지 함께 확인한다.

## 5. 확장 단계와 완료 기준

| 순서 | 분석 | 주요 산출물 | 완료 기준 |
|---:|---|---|---|
| 1 | MRI MVP 확정 | 모델 비교표, CN test 평가, 진단군 대비표 | 재실행 가능한 표·그림·해석 문장 연결 |
| 2 | Amyloid PET 부차 분석 | PET 정렬 흐름표, Amyloid 상태별 BAG 표·그림 | QC 통과 PET를 MRI와 ±180일 내 가장 가까운 시점으로 연결 |
| 3 | 견고성 분석 | 날짜 창·ROI 집합·eTIV·모델 민감도 표 | 주 분석과 비교해 결론의 방향을 투명하게 제시 |
| 4 | Normative ROI 분석 | CN 기대치 대비 ROI Z-score 요약 | CN train에서만 정상 기대치 모형 적합 |
| 5 | 피질 뇌 표면 시각화 | FDR 보정 parcel 표, PNG, 회전형 HTML | 앞 단계의 정의와 주 결과가 고정된 뒤 수행 |

### Amyloid PET 부차 분석

PET image QC와 timing QC가 모두 통과한 기록 중, index MRI와 절대 날짜 차이가 180일 이하인 가장 가까운 PET 한 건을 참가자별로 연결한다. 주 모형은 다음과 같다.

```text
BAG ~ amyloid_status × AGE + diagnosis + sex + education + eTIV
```

Amyloid 상태와 진단은 서로 관련될 수 있으므로, 결과는 진단·나이·성별·교육·eTIV 조건을 고려한 연관성으로 해석한다. ±90일·±180일·±365일 날짜 창 민감도 분석을 함께 제시한다.

### Normative ROI와 뇌 표면 시각화

CN train에서 각 ROI의 정상 기대치 모형을 만들고, held-out CN 및 임상군에서 기대치 대비 Z-score를 계산한다. 피질 두께 68개 Desikan-Killiany parcel에서는 다음과 같은 관계를 평가한다.

```text
cortical aging burden ~ standardized BAG + age + sex + eTIV
```

각 parcel의 HC3 robust 회귀 p값은 Benjamini–Hochberg FDR로 보정한다. fsaverage5 표면의 정적 PNG와 회전형 HTML은 집단 수준의 ROI 연관성 지도이며, 개인 MRI 판독·voxel 수준 병변·인과 지도가 아니다.

## 6. 실행 순서

VS Code에서 프로젝트 최상위 폴더를 열고 Python/Jupyter 커널을 선택한다.

1. `notebooks/01_mri_brain_age_bag_mvp.ipynb`
2. `notebooks/02_mri_amyloid_pet_bag.ipynb`

노트북은 `data/adni_all` 경로를 기준으로 프로젝트 루트를 자동 탐색한다. ADNI 원본 및 참가자 수준 처리 데이터는 저장소에 포함하지 않는다.

## 7. 해석 원칙

- 예측에 기여한 ROI는 원인 또는 독립 바이오마커가 아니다.
- Amyloid PET와 BAG의 연관성은 관찰적 관계이며 인과관계를 증명하지 않는다.
- 외부 코호트 검증 전에는 ADNI 밖의 일반화 성능을 주장하지 않는다.
- 결과가 유의하지 않더라도, 사전 정의된 분석과 한계를 투명하게 제시한다.
