# Deep Learning Term Project

딥러닝 기말 프로젝트의 코드와 실험 요약을 관리하는 저장소입니다.
현재 Task 1을 정리했으며 Task 2 코드는 아직 포함하지 않았습니다.

## 바로 실행

[Colab에서 task1.ipynb 열기](https://colab.research.google.com/github/Xun7122/deep-learning-term-project/blob/main/task1.ipynb)

위 링크는 이 파일들이 main에 커밋된 이후 사용할 수 있습니다.

1. Colab에서 `task1.ipynb`를 엽니다. 기본 CPU 런타임으로 실행할 수 있습니다.
2. Google Drive에 원본 `heloc_release.csv` 또는 `heloc_release.xlsx`를 보관합니다.
3. 코드 맨 위 `DATA_FILE`을 실제 Drive 경로로 바꿉니다.
4. **코드 셀 하나를 실행**하고 Drive 연결을 승인합니다.

```python
USE_DRIVE = True
DATA_FILE = '/content/drive/MyDrive/heloc_release.csv'
```

실제 파일이 하위 폴더에 있으면 경로에 그 폴더를 포함하세요.
Colab의 왼쪽 파일 창에서 Drive 파일을 우클릭해 경로를 복사할 수 있습니다.
ZIP이나 별도 Python 파일 업로드는 필요하지 않습니다.

## 실행 흐름

원본 데이터 → 층화 Train/Valid/Local Test 분리 → 전처리 학습 →
10개 설정 × 학습 seed 2개 비교 → Validation 기준 단일 DNN 선택 →
Local Test 1회 평가 → 선택한 설정으로 전체 제공 데이터 재학습 → 결과 저장.

- 분리 seed: `20261125`.
- 실제 분리: Train 4,737 / Validation 1,579 / Local Test 1,580명.
- 전처리 통계와 KNN 이웃은 후보 학습 시 Train에서만 학습합니다.
- -7/-8/-9를 코드별 표시로 보존하며, 표준화 수치를 0 또는 KNN으로 대체합니다.
- KNN의 거리 계산에 정답과 표시 변수를 사용하지 않습니다.
- 후보: KNN 이웃 수·가중 방식, 작은 DNN 구조, Dropout, BatchNorm, 규제.
- 공통 학습: AdamW, BCEWithLogitsLoss, 학습률 감소, gradient clipping, 조기 종료.
- 후보 선택은 학습 seed별 Validation AUROC의 평균으로 합니다.
- 최종 가중치는 선택된 설정 중 Validation AUROC가 가장 높은 **단일 DNN**입니다.
- 앙상블하지 않습니다. 이 노트북은 5-fold 교차검증을 실행하지 않습니다.

기본값은 최대 150 epoch, 조기 종료 patience 20, 배치 128입니다.
`REFIT_ALL_DATA = True`이므로 Test 평가 후 고정된 epoch와 학습률 일정으로
전체 제공 데이터 재학습까지 진행합니다. 생략하려면 False로 바꾸세요.
이 단계에서 Test 결과로 설정을 조정하지 않습니다.

## 기록된 실험: 2026-11-25

이 기록은 실행 결과 ZIP에서 가져온 집계값입니다.
노트북의 학습 파이프라인은 해당 실험과 같은 코드이며, 입력·저장 부분을
Drive 경로 및 CSV/XLSX 지원 방식으로 정리했습니다.
정리한 노트북 전체를 여기서 다시 학습 실행하지는 않았습니다.
노트북 출력은 지워두었으며 세부 설정별 점수는 `experiment_log.csv`에 있습니다.

| 항목 | 선택 결과 |
|---|---|
| 전처리 | KNN 30, 동일 가중 + 특수상태 표시 |
| 은닉층 | 32개 뉴런 한 층 |
| Dropout | 0.2 |
| AdamW weight decay | 0.001 |
| BatchNorm | 사용하지 않음 |
| 선택 seed / epoch | 42 / 45 |
| 선택 설정의 Validation AUROC 평균 | 0.826475 (학습 seed 42·43) |
| 선택된 단일 모델의 Train AUROC | 0.817490 |
| 선택된 단일 모델의 Validation AUROC | 0.826719 |
| 선택된 단일 모델의 Local Test AUROC | 0.790403 |

**위 Local Test 점수는 전체 데이터 재학습 전 모델의 점수입니다.**
전체 7,896명으로 재학습한 제출 후보에는 별도의 새 Local Test 점수가 없습니다.
Validation 평균의 표준편차는 같은 고객 분리에서 학습 seed만 바꾼 결과이며,
데이터 분할에 따른 성능 변동이나 교차검증 편차를 뜻하지 않습니다.

이전 실험에서 이미 이 고객 데이터 전체가 사용되었습니다. 새 Local Test는
앞으로 설정 선택에서 분리하기 위한 데이터이지 역사적으로 완전히 미사용인 Test가 아닙니다.
교수님 비공개 Test가 최종 평가입니다. Test 점수를 보고 반복 튜닝하지 않습니다.
이전 NumPy 구현의 AUROC 0.7957과는 분리·구현이 달라 직접 비교하지 않습니다.
이번 Validation 1·2위 차이는 약 0.00006이므로 선택 구조의 확실한 우월성을 주장하지 않습니다.

## 결과 보관

결과는 Drive의 원본 파일 옆 `task1_results/<실행시각>/`와 결과 ZIP에 저장됩니다.

- `task1.pt`, `preprocessor.joblib`: 학습/검증으로 선택하고 Local Test에서 평가한 모델.
- `submission_model/task1.pt`, `submission_model/preprocessor.joblib`: 전체 재학습 제출 후보.
- 모델과 그 모델에 대응하는 전처리 파일을 반드시 **같은 폴더의 한 쌍**으로 보관하세요.
- `decision.json`, `test_report.json`, 학습 이력과 실제 라이브러리 버전도 Drive에 보관합니다.
- KNN 전처리에는 학습 이웃 정보가 포함됩니다. 공개 저장소에 올리지 않습니다.

## 환경

노트북은 Colab에 설치된 패키지를 사용하고 없는 패키지만 설치합니다.
실제 실행 버전은 매 실행의 `requirements_actual.txt`에 기록됩니다.
저장소 `requirements.txt`에는 2026-11-25 실험에서 기록된 버전을 보관했습니다.
그래프용 matplotlib의 당시 정확한 버전은 기록되지 않아 고정하지 않았습니다.
다른 환경에서는 해당 CPU PyTorch wheel의 제공 여부와 패키지 호환성을 확인해야 합니다.
기록된 버전 파일이 모든 환경에서 동일 실행 결과를 보장하지는 않습니다.

## Git에 포함하는 것

- `task1.ipynb`: 출력이 제거된 한 셀 실행 노트북.
- `README.md`: 실행법, 모델과 평가 결과의 의미.
- `requirements.txt`: 실제 실험의 라이브러리 버전.
- `experiment_log.csv`: 고객별 정보가 없는 집계 실험 요약.
- `.gitignore`: 원본 데이터·가중치·전처리·상세 결과 제외 규칙.

원본 CSV/XLSX, 고객별 예측, 분할 고객 ID, 모델 가중치와 결과 ZIP은 포함하지 않습니다.
이 저장소에는 데이터 다운로드 링크도 포함하지 않습니다.
