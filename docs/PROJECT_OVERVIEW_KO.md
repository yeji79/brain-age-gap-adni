# 구조 MRI 기반 Brain Age Gap 분석: 알츠하이머병 스펙트럼

이 저장소는 ADNI의 ADSP PHC ComBat 보정 구조 MRI FreeSurfer ROI를 사용해 정상 인지군(CN)의 뇌 나이를 예측하고, Brain Age Gap(BAG)을 알츠하이머병 스펙트럼에서 탐색하는 재현 가능한 분석 프로젝트입니다.

**마지막 업데이트: 2026-08-20**

## 연구 질문

CN의 구조 MRI로 학습한 정상 노화 기준에서, MCI와 Dementia 집단은 같은 실제 나이의 CN보다 더 높은 BAG를 보이는가? 또한 MRI 촬영 시점과 가까운 Amyloid PET의 양성 상태는 BAG와 어떤 연관을 보이는가?

`BAG = 나이 편향을 보정한 예측 뇌 나이 − 실제 나이`입니다. 양의 BAG는 해당 MRI가 같은 실제 나이의 CN 기준보다 더 고령의 정상 뇌 구조와 유사함을 뜻하는 연구용 구조적 편차 지표입니다.

## 데이터와 분석 단위

- **MRI:** `ADSP_PHC_T1_FS_22Jan2026.csv`의 ComBat 보정 FreeSurfer ROI
- **공변량 보강:** `PTDEMOG`, `DXSUM`
- **PET 부차 분석:** `ADSP_PHC_PET_Amyloid_Simple_22Jan2026.csv`
- **분석 단위:** 참가자(RID)별 가장 이른 유효 T1 MRI 한 건(index MRI)

원본 MRI 영상이 아닌 ROI 표형 데이터를 사용합니다. ADNI 원본 및 참가자 수준 처리 데이터는 데이터 사용 계약에 따라 저장소에 포함하지 않습니다.

## 분석 설계

1. CN만으로 train/validation/test를 RID 기준으로 분할합니다.
2. train 데이터에서만 MRI ROI 결측 처리·표준화 규칙을 학습합니다.
3. Dummy, Ridge, Extra Trees를 비교하고, 상관된 ROI에 안정적인 Ridge를 주 모델로 사용합니다.
4. validation CN에서 나이 편향 보정식을 적합하고, 이를 동결해 모든 집단에 적용합니다.
5. held-out CN test에서 예측 성능과 보정 후 BAG–나이 관계를 확인합니다.
6. 진단군 비교에서는 `diagnosis × AGE` 상호작용을 점검하고, 필요할 때 65·75·85세의 CN 대비 조정 BAG 차이를 보고합니다.
7. PET 확장에서는 QC를 통과한 Amyloid PET를 index MRI와 ±180일 안에서 가장 가까운 시점으로 연결합니다.

## 현재 MRI MVP 결과

현재 실행 기록에서 Ridge의 held-out CN test raw MAE는 **3.943년**이었고, Dummy 기준선보다 **1.450년** 낮았습니다. Validation 기반 보정 뒤 BAG와 실제 나이의 상관은 -0.737에서 -0.102로 감소했습니다.

전체 MVP 코호트의 상호작용 모형에서 진단군과 나이의 상호작용이 유의했습니다. 따라서 단일 평균 차이 대신 기준 나이별 대비를 해석했습니다. 75세에서 MCI와 Dementia의 조정 BAG는 CN보다 각각 **4.63년**과 **11.68년** 높았습니다.

이 결과는 ADNI 내부의 집단 수준 관찰 결과입니다. 독립 CN test를 기준군으로 제한하는 민감도 분석과 Amyloid PET 분석은 진행 중이며, 결과는 해당 분석이 실제로 완료된 뒤에만 갱신합니다.

## 현재 진행 상태

| 단계 | 상태 | 다음 산출물 |
|---|---|---|
| 구조 MRI MVP | 실행 및 해석 완료 | held-out CN 기준 민감도 분석 |
| Amyloid PET | 실행 노트북 작성 완료 | PET 정렬 흐름표·상호작용 결과 |
| Normative ROI | 설계 완료 | CN train 기반 Z-score 요약 |
| 피질 뇌 표면 | 설계 완료 | FDR 보정 parcel 지도 |

## 실행 방법

VS Code에서 이 프로젝트의 최상위 폴더를 열고 Python/Jupyter 커널을 선택합니다.

1. `notebooks/01_mri_brain_age_bag_mvp.ipynb`를 위에서부터 실행합니다.
2. `notebooks/02_mri_amyloid_pet_bag.ipynb`를 실행해 PET 부차 분석을 수행합니다.

노트북은 `data/adni_all`을 기준으로 프로젝트 루트를 자동 탐색합니다.

## 해석과 한계

- BAG는 개인의 치매 진단, 미래 위험, 개인별 뇌 노화 속도를 확정하는 점수가 아닙니다.
- ADNI는 관찰 코호트이므로 BAG와 진단 또는 Amyloid 상태의 연관성은 인과관계를 뜻하지 않습니다.
- ROI 중요도와 이후의 뇌 표면 시각화는 예측 모델과 연관된 구조 정보를 보여 줄 뿐, 특정 뇌 영역의 독립적 원인성을 증명하지 않습니다.
- 외부 코호트 검증 전에는 다른 병원이나 인구집단으로의 일반화를 주장할 수 없습니다.

## 다음 확장

Amyloid PET 결과를 확정한 뒤, CN train 기반 normative ROI Z-score, 민감도 분석, 그리고 FDR 보정을 포함한 Desikan-Killiany 피질 뇌 표면 시각화를 순서대로 수행할 계획입니다.

자세한 설치·데이터 보호 원칙은 [한국어 README](../README_ko.md)와 [English README](../README.md)를 참고하세요.
