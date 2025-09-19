# 🚗 Hyundai_DT – COCO → YOLOv8 End-to-End Pipeline

이 프로젝트는 **현대건설 DT 비전 데이터**를 대상으로  
COCO 형식의 어노테이션을 **YOLO 형식으로 변환**하고  
다양한 YOLO 버전(YOLOv8/9/10 등)을 **동일 환경에서 손쉽게 실험**할 수 있도록 만든  
완전 자동화 파이프라인입니다.

---

## 📚 Tutorial Overview
이 저장소는 다음 순서대로 작업을 자동화합니다.

- **COCO Split** : 이미지 폴더(train/test) 기준으로 COCO json을 `*_train.json`, `*_test.json`으로 분리  
- **COCO → YOLO 변환** : 분리된 json을 YOLO txt 라벨로 변환  
- **YOLO 학습** : ultralytics 패키지로 원하는 YOLO 버전을 학습  
- **검증/시각화** : 모델 예측 vs GT 이미지 비교 결과 자동 저장

---

## 🔧 환경 세팅

### ▸ 프로젝트 클론
```bash
git clone <repo_url> Hyundai_DT
cd Hyundai_DT
```

### ▸ Conda 환경 생성 및 패키지 설치
아래 명령을 **한 번에 복사·실행**하면 Conda 환경 생성부터 패키지 설치까지 완료됩니다.
```bash
conda create -n vision_env python=3.10 -y && conda activate vision_env && pip install -r requirements.txt
```
> 💡 Docker를 사용할 경우 `requirements.txt`를 그대로 COPY 후 빌드하면 됩니다.

---

## 📂 데이터 준비

아래와 같은 디렉터리 구조를 맞춰주세요.

```
Hyundai_DT/
├─ data/
│  ├─ images/
│  │   ├─ A_site/
│  │   │   ├─ train/*.png
│  │   │   └─ test/*.png
│  │   └─ B_site/...
│  └─ labels/
│      ├─ A_site.json
│      └─ B_site.json
├─ configs/
│  └─ data.yaml
└─ src/...
```

* **images/** : 각 사이트별 `train/`, `test/` 폴더에 원본 이미지 배치  
* **labels/** : 사이트별 COCO json 원본

---

## ⚙️ 주요 실행 파일
| 파일 | 설명 |
|------|------|
| `src/coco_split.py` | COCO json → train/test json 분리 |
| `src/json2txt.py`   | COCO → YOLO txt 변환 |
| `src/baseline.py`   | 단일 YOLO 모델 학습 샘플 |
| `src/only_val.py`   | 검증 이미지 예측/라벨 시각화 |
| **`src/main.py`**   | 🔥 전체 파이프라인 (Split → 변환 → 학습 → 검증) |

---

## 🚀 빠른 시작 (One-Line 실행)

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

## 🔬 다양한 YOLO 버전 실험
예를 들어 YOLOv8 small 모델을 학습하려면:
```bash
python src/main.py --model yolov8s.pt --epochs 150
```
YOLOv9 또는 YOLOv10도 동일하게 실행 가능합니다.  
(ultralytics 패키지에서 지원하는 가중치 사용)

---

## 📊 결과 확인
- `runs/exp_<모델명>/results.png` : 학습 곡선(mAP, Precision/Recall)
- `runs/exp_<모델명>/weights/best.pt` : 최적 가중치
- `runs/person_val/` : 검증 데이터의 **예측/GT 시각화 이미지**

---

## 🛠️ 개발 팁
| 주제 | 팁 |
|------|----|
| **환경 재현** | `requirements.txt` 버전 고정 |
| **하이퍼파라미터 탐색** | Optuna 등 외부 튜너 연동 가능 |
| **배치 실험** | bash 스크립트로 여러 모델 자동 실행 |
| **클라우드/GPU** | Docker 이미지로 손쉬운 배포 |

---

## 📜 라이선스
연구 및 교육 목적으로 자유롭게 사용 가능.  
상업적 사용 시 데이터/가중치 라이선스 확인 필요.

---

## 👥 팀 공통 체크리스트
- [ ] `requirements.txt` 설치 완료  
- [ ] `data/` 폴더 구조 및 COCO json 배치 완료  
- [ ] `configs/data.yaml` 클래스 수와 이름 확인  
- [ ] `python src/main.py` 테스트 실행

---

✨ 이제 팀원 누구나 **동일한 환경**에서  
단 한 줄 명령으로 YOLO 파이프라인을 실행하고  
여러 버전을 손쉽게 실험할 수 있습니다!
