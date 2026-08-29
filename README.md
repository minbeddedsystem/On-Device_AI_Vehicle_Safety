# 🚗 On-Device AI 기반 차량 내 잔류 탑승자 위험 감지 및 능동 안전 시스템

NVIDIA **Jetson Orin Nano** 위에서 3대의 Webcam 입력을 실시간으로 처리하여, 운전자 졸음, 차량 내 잔류 아동/반려동물, 차량 주변 비인가 접근을 하나의 On-Device AI 파이프라인으로 감지하는 팀 프로젝트입니다. 모든 추론과 위험 판단은 외부 서버 없이 Jetson 내부에서 수행하며, 최종 상태와 이벤트 정보만 Web UI/Telegram으로 전달합니다.

> 온디바이스 시스템반도체설계 2기 팀 프로젝트 (2026.08.05 ~ 2026.08.19, 팀원: 이나경, 이찬미, 최민영)

## 📌 Overview

| 항목 | 내용 |
|---|---|
| 핵심 플랫폼 | NVIDIA Jetson Orin Nano Developer Kit (Ampere GPU, 최대 67 INT8 TOPS) |
| 입력 | USB Webcam × 3 (운전자 / 뒷좌석 / 차량 주변) |
| OS / SDK | JetPack 6.2.2 (Jetson Linux), CUDA 12.6 |
| AI Framework | PyTorch, TensorRT FP16, TorchScript |
| 모델 학습 환경 | Google Colab (Tesla T4 GPU) |
| 결과 전달 | Web UI (FastAPI / Flask), Telegram Bot Alert |
| 서브시스템 | System1 졸음 감지 · System2 잔류 탑승자 감지 · Security System 보안카메라 |

## ✨ 프로젝트 목표

기존의 단순 인원 감지 방식에서 확장하여 운전자 상태, 차량 내 잔류 탑승자, 차량 주변 접근 상황을 각각 분석하고, 위험 상황 발생 시 사용자에게 경고를 제공하는 통합 시스템을 목표로 하였습니다. 모델 선정 시에도 정확도만 우선하지 않고, Jetson 환경에서의 실시간 추론 가능성, 모델 경량성, 전력 효율을 함께 평가 기준으로 삼았습니다.

## 🏗️ 전체 시스템 구조

```
                     ┌────────────────────────────┐
   Webcam(운전자) --> │                            │
                     │                            │
   Webcam(뒷좌석) --> │   Jetson Orin Nano         │ --> Web UI (FastAPI/Flask)
                     │   (On-Device AI Inference) │
   Webcam(차량주변) --> │                            │ --> Telegram Bot Alert
                     └────────────────────────────┘
```

| 서브시스템 | 감지 대상 | 적용 모델 | 최종 출력 |
|---|---|---|---|
| **System1** (졸음 감지) | 눈 감김 · 하품 · 고개 자세 | MobileViT-XXS 기반 Multi-task | NORMAL / WARNING / DANGER |
| **System2** (잔류 탑승자 감지) | 7세 이하 아동 · 반려동물 잔류 | Fine-tuned YOLO11n + MiVOLO V2 | CHILD / ADULT / ANIMAL, Stage 0~3 |
| **Security System** (보안카메라) | 등록/미등록 사용자, 위조 얼굴 | YuNet + MiniFASNetV2 + SFace | OWNER / GUEST / UNKNOWN / SPOOF |

각 서브시스템의 상세 구조, 실행 방법, 성능 비교 결과는 폴더별 README를 참고하세요.

- [`system1/`](./system1) — 운전자 졸음 모니터링 (Multi-task Eye · Yawn · Head Pose)
- [`system2/`](./system2) — 잔류 아동/반려동물 감지 (YOLO11 + MiVOLO V2)
- [`security_system/`](./security_system) — 차량 보안카메라 (얼굴 인증 · Anti-Spoofing)

## 🔄 AS-IS / TO-BE

| 구분 | AS-IS | TO-BE |
|---|---|---|
| 운전자 모니터링 | 단일 행동/제한된 조건 중심 감지 | 눈 감김·하품·고개 자세를 종합한 실시간 Risk Score 판단 |
| 잔류 탑승자 감지 | 레이다 기반 단순 인원 잔류 여부 판단 | 아동·성인·반려동물 구분 + 잔류 지속시간 기반 단계별 위험도 |
| 차량 보안 | 영상 기록 또는 단순 접근 감지 | 등록 여부 + Spoofing 판별 + 지속시간 기반 이상 접근 판단 |
| 모델 적용 | 정확도 중심 모델 선정 | 정확도·Recall·Latency·FPS·전력 효율을 종합한 Edge 최적 모델 선정 |
| 추론 방식 | 외부 서버/개별 시스템 중심 처리 | Jetson Orin Nano On-Device AI 단일 처리 |
| 경고 방식 | 단순 감지 결과 또는 개별 알림 | 위험 상태·지속시간 기반 Web UI + Alert Server 단계별 경고 |

## 👥 Team & Role

| 팀원 | 담당 업무 |
|---|---|
| 최민영 | **System1(졸음 감지) 개발 및 성능 분석 중심**. Eye/Yawn/Head Pose 데이터셋 구축, ResNet18·MobileNetV2·MobileViT-XXS 학습 및 성능(정확도·모델크기·Jetson Latency) 비교, Grad-CAM 기반 판단 근거 분석, YOLO11n/s/m 전력 소비(Energy/Frame) 측정 및 분석, System1 최종보고서·발표자료 작성, 최종 발표 진행 |
| 이나경 | 각 시스템 초기 구현 및 Security System 중심 개발. System1 3종 모델 데이터셋 학습, System1 시연영상 제작, System2 MiVOLO GPU 실행 구조 구축 및 YOLO11n/s/m+MiVOLO Pipeline E2E Latency·FPS 분석, Security System 구현 및 시연영상 제작, System1·Security 보고서/발표자료 작성 |
| 이찬미 (팀장) | **System2(잔류 탑승자 감지) 개발 및 성능 분석 중심**. YOLO11n/s/m 정확도 평가 환경 구축, Precision·Recall·F1-score·mAP50·mAP50-95 및 Young/Animal Recall 분석, System2 최종 모델 선정 및 YOLO11n Fine-tuning, System2 시연영상 제작, System2 보고서/발표자료 작성 |
| 팀 전체 | 프로젝트 주제·구현 범위 결정, 시스템 아키텍처·데이터 흐름 설계, 모델 종합 평가 및 최종 모델 선정, 시스템별 결과 검증, 전체 시연/발표자료/보고서 최종 검토 |

## 🗓️ 진행 일지

| 날짜 | 단계 | 주요 작업 |
|---|---|---|
| 8/5 | 기획 | 프로젝트 주제·기능 범위 선정, 3개 서브시스템 구조 설계, 알림 방식 초기 검토 |
| 8/6~8/7 | 아키텍처·데이터 설계 | 서브시스템별 모델 후보 검토, 학습/평가 데이터셋 수집, 알림 방식(Telegram/Discord/WebSocket) 비교 |
| 8/6~8/10 | 시스템·모델 구현 | System1 Multi-task 모델 학습, System2 YOLO11/MiVOLO GPU 환경 구축, Security 모델 연동, Web UI 구성 |
| 8/7~8/11 | 성능 평가·모델 비교 | 모델별 Accuracy/Recall/연산량/추론시간/전력/모델크기 측정, YOLO11n/s/m 비교 분석 |
| 8/11~8/14 | Fine-tuning 및 시스템 통합 | System1 경고 판정 기준 보완, System2 YOLO11n Fine-tuning 및 재평가, Security 사용자 판별/알림 통합 |
| 8/12~8/18 | 결과 분석·자료 정리 | 모델 비교 결과·트러블슈팅 분석, Grad-CAM 시각화, 시연 영상 및 발표자료 제작 |
| 8/18 | 최종 검증·문서화 | Jetson 통합 실행 확인, 소스코드·모델·실행방법 정리, 완료보고서 점검 |
| 8/19 | 발표 | 최종 시스템 시연, 프로젝트 결과 발표 및 질의응답 |

## ⚙️ Design Environment

- Hardware: NVIDIA Jetson Orin Nano Developer Kit, USB Webcam × 3
- OS / SDK: JetPack 6.2.2 (Jetson Linux), CUDA 12.6
- Language: Python
- AI Framework: PyTorch, TensorRT FP16, TorchScript
- Model Training: Google Colab (Tesla T4 GPU)
- Object Detection: YOLO11n / YOLO11s / YOLO11m (Ultralytics)
- Age Estimation: MiVOLO V2
- User Interface: Web UI (FastAPI / Flask), Telegram Bot Alert
- Version Management: Git / GitHub

## 📦 참고 사항

- 용량이 큰 모델 가중치(ResNet18 Multi-task `.pth`/`.onnx` 43MB, YOLO11s/m 계열, MiVOLO V2 `.pt`/`.onnx.data` 100MB 이상, TensorRT `.engine` 파일 등)와 실제 등록된 얼굴 임베디드 DB(`face_database.sqlite3`), 보안 이벤트 캡처 이미지는 저장소 용량 제한 및 개인정보 보호를 위해 포함하지 않았습니다. `.engine` 파일은 Jetson 보드·TensorRT 버전에 종속적이라 재사용이 불가능하므로 각 시스템 폴더의 실행방법 문서를 참고해 직접 변환해서 사용하세요.
- 각 서브시스템 실행 방법은 폴더별 `실행방법_*.md` 문서에 상세히 정리되어 있습니다.
