# NoteMind

Flutter · Supabase · vLLM으로 만드는 RAG 노트 앱 **NoteMind**의 수업 저장소입니다.
학생은 `app-starter/`의 빈 core 함수(TODO)를 채워 문서 업로드와 질문하기 기능을 완성합니다.

## 구성

| 경로 | 내용 |
|---|---|
| [`app-starter/`](app-starter/README.md) | 실습용 Flutter 앱. 화면은 완성되어 있고 core 함수 12개가 비어 있습니다 |
| [`docker/`](docker/README.md) | 로컬 Supabase와 vLLM(NVIDIA) Docker 구성, DB 마이그레이션 |
| [`start.py`](START_GUIDE.ko.md) | 로컬 모델을 골라 vLLM / vLLM-Metal을 띄우고 API를 확인하는 실행기 |

## 처음 한 번: 서버 준비

이미 지난 수업에서 마쳤다면 건너뜁니다. 프로젝트 루트에서 실행합니다.
Windows는 `python3` 대신 `python`을 씁니다.

1. 비밀값 파일(`.env`) 만들기

   ```bash
   python3 docker/init-env.py
   ```

2. Supabase 시작과 DB 마이그레이션

   ```bash
   docker compose --project-directory docker/supabase up -d --wait
   # Mac · Linux
   docker compose --project-directory docker/supabase exec -T db psql -U postgres -d postgres -v ON_ERROR_STOP=1 < docker/supabase/migrations/001_notemind.sql
   ```

   Windows PowerShell은 `<`를 쓸 수 없으므로 다음과 같이 실행합니다.

   ```powershell
   Get-Content docker/supabase/migrations/001_notemind.sql | docker compose --project-directory docker/supabase exec -T db psql -U postgres -d postgres -v ON_ERROR_STOP=1
   ```

3. 추론 서버(채팅 8000 · 임베딩 8001)
   - **Windows (WSL2 + NVIDIA GPU)**: `docker compose --project-directory docker/vllm up -d`
   - **Mac (Apple Silicon)**: [vLLM-Metal 설치](docker/vllm/README.md) 후 터미널 두 개에서 실행합니다.

     ```bash
     python3 docker/vllm/serve-metal.py chat
     python3 docker/vllm/serve-metal.py embedding
     ```

     앱의 기본 모델 ID(`notemind-chat`, `notemind-embedding`)로 서버가 뜹니다.
     `start.py`로 띄우면 모델 ID가 모델 원래 이름이 되므로, 앱의 연결 설정에서 모델 ID를 그 이름으로 바꿔야 합니다.

## 수업 때마다

1. 서버 켜기
   - **Windows**: Docker Desktop에서 `supabase`와 `vllm` 컨테이너를 시작합니다.
   - **Mac**: Docker Desktop에서 `supabase` 컨테이너를 시작하고, 위의 `serve-metal.py` 두 개를 실행합니다.
2. 에뮬레이터를 켠 뒤 포트를 연결합니다. 에뮬레이터를 다시 켤 때마다 필요합니다.

   ```bash
   adb reverse tcp:54321 tcp:54321
   adb reverse tcp:8000 tcp:8000
   adb reverse tcp:8001 tcp:8001
   ```

3. 앱을 실행하고 **연결 설정** 탭에 키를 넣습니다. 자세한 내용은 [`app-starter/README.md`](app-starter/README.md)를 따르세요.
   - Supabase 공개 키: `docker/supabase/.env`의 `ANON_KEY`
   - 로컬 추론 API 키: `docker/vllm/.env`의 `VLLM_API_KEY`

## 종료

```bash
docker compose --project-directory docker/supabase down   # 데이터는 유지됩니다
docker compose --project-directory docker/vllm down       # Windows
```

`down -v`는 DB와 업로드 파일이 담긴 볼륨을 지우므로 일반 종료에 쓰지 않습니다.
Mac의 `serve-metal.py`는 실행한 각 터미널에서 `Ctrl+C`로 종료합니다.

## 라이선스

[MIT](LICENSE). `docker/supabase/`의 Supabase 구성 파일은 원본 라이선스(`docker/supabase/LICENSE.upstream`)를 따릅니다.
