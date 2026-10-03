# 환율 계산기 (FastAPI)

전 세계 통화 환율을 실시간으로 계산해주는 웹 사이트입니다. FastAPI로 만들었고,
open.er-api.com의 무료 환율 API를 사용합니다.

## 기능
- 금액, 보내는 통화, 받는 통화를 선택하면 즉시 환율이 계산됩니다.
- 160개 이상의 통화를 지원합니다.
- 접속한 브라우저의 언어를 감지해 한국어, English, 日本語, 中文, Español 중 하나로 자동 표시됩니다.
- 서버에서 1시간 동안 환율을 캐시해 API 요청 제한을 피합니다.
- 환율 API 접속이 실패하면 예비 환율로 대신 계산하고, 화면에 안내 문구를 보여줍니다.

## 로컬에서 실행하기
```bash
pip install -r requirements.txt
uvicorn main:app --reload
```
브라우저에서 http://127.0.0.1:8000 으로 접속합니다.

자동 생성된 API 문서는 http://127.0.0.1:8000/docs 에서 확인할 수 있습니다.

## API
- `GET /api/convert?amount=100&from=USD&to=KRW` — 환율 변환
- `GET /api/currencies` — 지원하는 통화 목록

## 배포하기 (Render 기준)
1. 이 저장소를 GitHub에 올립니다.
2. [render.com](https://render.com)에 GitHub 계정으로 가입합니다.
3. New + → Web Service → 이 저장소 선택.
4. Build Command: `pip install -r requirements.txt`
5. Start Command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
6. Instance Type: Free 선택 후 Create Web Service.
7. 배포가 끝나면 `https://앱이름.onrender.com` 주소로 전 세계 어디서든 접속할 수 있습니다.

## 출처 표기
환율 데이터는 [ExchangeRate-API](https://www.exchangerate-api.com)에서 제공하며,
이용 약관에 따라 화면 하단에 출처 링크를 표시하고 있습니다. 이 부분은 지우지 마세요.
