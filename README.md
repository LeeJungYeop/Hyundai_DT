# Hyundai_DT(Vision) 

이 프로젝트는 **비전 데이터**를 대상으로  
COCO 형식의 어노테이션을 **YOLO 형식으로 변환**하고  
다양한 YOLO 버전(YOLOv8/9/10 등)을 **동일 환경에서 손쉽게 실험**할 수 있도록 만든  
자동화 파이프라인입니다.

---

## Overview
이 저장소는 다음 순서대로 작업을 자동화합니다.

- **COCO Split** : 이미지 폴더(train/test) 기준으로 COCO json을 `*_train.json`, `*_test.json`으로 분리  
- **COCO → txt 변환** : 분리된 json을 YOLO txt 라벨로 변환  
- **YOLO 학습** : ultralytics 패키지로 원하는 YOLO 버전을 학습  
- **검증/시각화** : 모델 예측 vs GT 이미지 비교 결과 자동 저장

---

## 환경 세팅 (Docker 기준)

### 프로젝트 클론
```bash
git clone <repo_url> hyundai_dt
cd hyundai_dt
```

### Docker 이미지 빌드 및 컨테이너 실행
아래 명령을 **한 번에 복사·실행**하면 Docker 이미지 생성부터 컨테이너 실행까지 완료됩니다.
```bash
# 1. 이미지 빌드
docker build -t hyundai_dt .

# 2. 컨테이너 실행 (GPU 사용 시 --gpus all 옵션 추가)
docker run -it --name hyundai_dt_container --gpus all -v ${PWD}:/workspace hyundai_dt /bin/bash
```

> 컨테이너 내부의 기본 작업 디렉터리는 `/workspace`입니다.

---

## 데이터 준비

아래와 같은 디렉터리 구조를 맞춰주세요. 유출 위험이 있기 때문에 데이터는 직접 넣어주세요.

```text
hyundai_dt/
├─ data/
│  ├─ images/
│  │   ├─ A_site/
│  │   │   ├─ train/*.png
│  │   │   └─ test/*.png
│  │   └─ B_site/...
│  └─ labels/
│      ├─ A_site.json
│      ├─ B_site.json
│      └─ ...
├─ configs/
│  └─ data.yaml
└─ src/
   ├─ main.py
   ├─ coco_split.py
   ├─ json2txt.py
   ├─ baseline.py
   ├─ only_val.py
   └─ coco_verification.py
```

---

## 주요 실행 파일
| 파일 | 설명 |
|------|------|
| `src/coco_split.py` | COCO json → train/test json 분리 |
| `src/json2txt.py`   | COCO → YOLO txt 변환 |
| `src/baseline.py`   | 단일 YOLO 모델 학습 샘플 |
| `src/only_val.py`   | 검증 이미지 예측/라벨 시각화 |
| **`src/main.py`**   | 전체 파이프라인 (Split → 변환 → 학습 → 검증) |

---

## 빠른 시작 (One-Line 실행)

아래 명령 한 줄로 **데이터 분리 → 변환 → 학습 → 검증**까지 자동 실행됩니다.

```bash
python src/main.py   --model yolov8n.pt   --imgsz 800   --epochs 100   --batch 16
```

### 주요 옵션
| 옵션 | 설명 | 기본값 |
|------|------|------|
| `--model` | YOLO 모델/가중치 (`yolov8n.pt`, `yolov8s.pt`, `yolov9s.pt`, `yolov10m.pt` 등) | `yolov8n.pt` |
| `--imgsz` | 입력 이미지 크기 | 800 |
| `--epochs` | 학습 epoch 수 | 100 |
| `--batch` | 배치 크기 | 16 |

모든 결과는 `runs/exp_<모델명>/` 폴더에 저장됩니다.

---

## 다양한 YOLO 버전 실험
예를 들어 YOLOv8 small 모델을 학습하려면:
```bash
python src/main.py --model yolov8s.pt --epochs 150
```
YOLOv9 또는 YOLOv10도 동일하게 실행 가능합니다.  
(ultralytics 패키지에서 지원하는 가중치 사용)

---

## 결과 확인
- `runs/exp_<모델명>/results.png` : 학습 곡선(mAP, Precision/Recall)
- `runs/exp_<모델명>/weights/best.pt` : 최적 가중치
- `runs/person_val/` : 검증 데이터의 **예측/GT 시각화 이미지**

---
