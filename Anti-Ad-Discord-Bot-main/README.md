# Anti-Ad Discord Bot (안티-광고 디스코드 봇)

이 프로젝트는 디스코드 서버 내의 스팸 및 광고 이미지를 AI 및 다양한 이미지 해싱/비교 알고리즘을 통해 탐지하고 차단하는 봇입니다.

## 1. 프로젝트 분석
- **기능**: 이미지 스팸 탐지 (5가지 알고리즘 결합: Perceptual Hash, Histogram, SIFT, ORB, AKAZE), 웹 관리 포털 제공, 사용자 처벌(뮤트, 밴 등), 이의 제기 시스템.
- **주요 스택**: Python 3.8+, `discord.py`, `Flask` (웹 포털), `OpenCV`, `scikit-image`.
- **데이터베이스**: 로컬 JSON 파일 기반 (`data.json`)을 사용하여 별도의 데이터베이스 서버 구축 없이도 가볍게 동작합니다.

## 2. 실행 환경 및 시스템 요구사항 (Galaxy Tab A7 - Chroot Ubuntu 24.04)
이 프로젝트는 Galaxy TAB A7 기기의 Chroot Ubuntu 24.04 환경에서 정상 구동되도록 최적화 및 디버깅 되었습니다.
- **주의사항**: 커널 호환성 문제로 Docker가 작동하지 않으므로, **가상환경(venv)을 통한 직접 실행** 방식을 사용해야 합니다.
- **OpenCV 호환성**: Chroot 등 GUI 환경(X11 등)이 없는 환경에서는 `opencv-python` 대신 `opencv-python-headless`를 사용하여 `libGL.so.1` 의존성 에러를 방지했습니다.

## 3. 주요 기능 및 변경사항
- ✅ **광고/스팸 이미지 탐지**: 봇에 업로드된 훈련용 데이터 이미지와 유사한 이미지를 업로드할 경우 즉각 탐지
- ✅ **자동 처벌 시스템**: 설정된 규칙에 따라 유저 자동 처벌. 기존의 Mute 기능뿐만 아니라 **Ban(차단) 기능이 추가**되었습니다. `.env` 파일의 `PUNISHMENT_TYPE` 설정을 통해 Mute와 Ban 중 선택하여 작동시킬 수 있습니다.
- ✅ **웹 어드민 포털**: 웹을 통해 통계, 탐지 기록, 유저 관리 등을 손쉽게 모니터링 가능
- ✅ **이의 제기(Appeal) 시스템**: 처벌받은 유저를 위한 이의 제기 및 관리자 승인/거절 기능

## 4. 설치 및 실행 가이드 (Chroot Ubuntu 환경)

### 1단계: 필수 패키지 설치
Chroot 환경에는 기본적인 라이브러리가 누락된 경우가 많습니다. 터미널을 열고 아래 패키지들을 먼저 설치해주세요.
```bash
sudo apt-get update
sudo apt-get install -y python3 python3-pip python3-venv git
```

### 2단계: 가상환경 세팅 및 모듈 설치
프로젝트 폴더 내에서 아래 명령어를 실행하여 가상환경을 만들고 필요한 라이브러리를 설치합니다.
```bash
# 가상환경 생성 및 활성화
python3 -m venv venv
source venv/bin/activate

# 요구사항 설치
pip install -r requirements.txt
```

### 3단계: 환경 설정 파일(.env) 작성
`config` 폴더에 `.env` 파일을 생성하고 봇 설정을 기입해야 합니다.
```bash
cp config/.env.example config/.env
```
`.env` 파일을 열어 다음 핵심 항목들을 본인 서버에 맞게 작성합니다.
- `DISCORD_TOKEN`: 디스코드 개발자 포털에서 발급받은 봇 토큰
- `GUILD_ID`: 봇이 동작할 서버(길드)의 ID
- `PUNISHMENT_TYPE`: 적발 시 처벌 옵션 (`mute` 또는 `ban`)

### 4단계: 훈련 데이터(스팸 이미지) 추가
스팸으로 탐지하고 싶은 이미지 파일들을 `Training-Data/` 폴더 안에 넣어줍니다. (JPG, PNG, WEBP 등 이미지 포맷 지원)

### 5단계: 봇 실행
봇과 웹 포털을 백그라운드에서 동시에 실행하기 위해 제공된 쉘 스크립트를 사용합니다.
```bash
chmod +x START.sh
./START.sh --background
```
실행 후 터미널을 종료해도 봇은 백그라운드에서 계속 동작합니다.
웹 포털은 기기 내부 또는 동일 네트워크 기기에서 `http://localhost:5000` 주소로 접속 가능합니다.

## 5. 트러블슈팅
1. **포트 충돌 에러 (Port 5000 is in use)**: 다른 프로세스가 5000번 포트를 쓰고 있을 수 있습니다. `kill $(lsof -t -i :5000)` 명령어로 해당 포트의 프로세스를 종료 후 다시 실행하세요.
2. **봇이 디스코드에서 오프라인일 때**: 토큰 설정이 잘못되었거나 백그라운드 프로세스가 죽었을 수 있습니다. `logs/bot.log` 파일을 열어 어떤 에러가 발생했는지 확인하세요.
3. **이미지 탐지율 조정**: 오탐지가 많거나 광고를 잘 잡지 못할 경우, `.env` 파일 내 `SIMILARITY_THRESHOLD` 값을 조정하세요 (기본값: `0.65`).
