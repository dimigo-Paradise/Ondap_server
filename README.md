# Ondap_server

## 폴더 구조

```text
Ondap_server/
├─ src/ondap_server/
│  ├─ core/                       # 설정·로깅·공통 예외
│  ├─ tasks/                      # Celery 작업 진입점·재시도·시간 제한
│  ├─ pipelines/
│  │  ├─ story/                   # 어르신 녹음: 변환 → STT → 마스킹 → 요약 → 결과 보고
│  │  └─ reply/                   # 보호자 답장: 형식·길이 검사 → 재생용 변환 → 결과 보고
│  ├─ clients/
│  │  └─ ondap_api/               # Ondap_api 내부 API 호출 (contracts/jobs 형식)
│  └─ integrations/               # 외부 서비스 어댑터
│     ├─ storage/                 # S3 원본 읽기·변환본 쓰기
│     ├─ audio/                   # FFmpeg 변환·형식·길이 검사
│     ├─ stt/                     # STT API
│     ├─ masking/                 # 요약 API로 보내기 전 이름·연락처·주소 등 가림
│     └─ llm/                     # 요약 API
├─ tests/
│  ├─ unit/                       # 파이프라인 단계별 테스트
│  ├─ integration/                # 실제 STT·요약 API 연동 (수동 실행)
│  └─ fixtures/                   # 시험용 짧은 녹음 (실제 어르신 녹음 금지)
├─ scripts/                       # 개발 실행·작업 수동 발행 도구
├─ docs/                          # 처리 흐름·외부 API 선택 기록
└─ .github/workflows/             # 검사·빌드·배포
```

## 필요해질 때 추가할 것

| 위치 | 내용 | 추가 시점 |
|---|---|---|
| `pipelines/` 추가 | 대화 주제 추천 등 새 처리 | 기능 확정 시 |
| `native/` | C++ 최적화 | CPU 병목이 측정으로 확인된 뒤 |

## 온답 저장소 구성

| 저장소 | 역할 |
|---|---|
| [Ondap_app_senior](https://github.com/dimigo-Paradise/Ondap_app_senior) | 어르신 앱 (Android·iOS). 이야기 녹음, 가족 답장 듣기. **MVP의 어르신 쪽 접점** |
| [Ondap_app_carer](https://github.com/dimigo-Paradise/Ondap_app_carer) | 보호자 앱 (Android·iOS). 요약 확인, 답장 녹음·승인 |
| [Ondap_api](https://github.com/dimigo-Paradise/Ondap_api) | HTTP API 서버 (Python·FastAPI). **DB를 소유하는 유일한 저장소** |
| [Ondap_server](https://github.com/dimigo-Paradise/Ondap_server) | 백그라운드 작업 (Python·Celery). STT·마스킹·요약·오디오 변환. DB 직접 접근 없음 |
| [Ondap_embed](https://github.com/dimigo-Paradise/Ondap_embed) | 스피커 기기 펌웨어 (ESP-IDF). MVP 이후 |

```text
senior app ─┐
            ├─ HTTPS ─▶ Ondap_api ─▶ PostgreSQL · S3 · FCM/APNs
carer app  ─┘              │  ▲
                           ▼  │   작업 발행(Redis) ↓ / 결과 보고(내부 API) ↑
                       Ondap_server ─▶ STT · 요약 API
```
