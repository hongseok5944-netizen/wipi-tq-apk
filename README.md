# 택티컬퀘스트용 WIPI Emulator APK 자동 빌드 패키지

이 폴더는 **실제 APK를 GitHub Actions에서 자동으로 컴파일**하기 위한 최소 프로젝트입니다.
현재 작업 환경에는 Rust/Android/Gradle 빌드 도구가 없어 여기서 APK 바이너리를 직접 만들 수 없기 때문에, GitHub의 Linux 빌드 환경을 사용합니다.

## 이 빌드가 하는 일

1. 공식 `ParkJeongseop/WIPI-Emulator` 저장소의 현재 소스를 가져옵니다.
2. Android용 Rust JNI 라이브러리를 `cargo-ndk`로 `arm64-v8a`와 `x86_64`용으로 빌드합니다.
3. 저장소에 있던 macOS 전용 NDK 경로 설정을 CI 환경에 맞게 제거합니다.
4. 현재 발생 중인 `Option::unwrap()` 네이티브 panic을 추적할 수 있도록
   - JNI `keyDown/keyUp` 경계를 `catch_unwind`로 감싸고
   - Rust backtrace를 logcat에 남기도록 진단 패치를 적용합니다.
5. Android Debug APK를 생성하고 GitHub Actions의 Artifact로 올립니다.

**중요:** 이 버전은 '원인 확인용 진단 APK'입니다. 기존 게임 파일을 다시 테스트하는 APK가 아니라, 에뮬레이터의 Rust 입력 경로를 수정해서 실제 panic 위치를 확인하기 위한 빌드입니다.

## 휴대폰에서 빌드하는 방법

1. GitHub에서 새 Repository를 하나 만듭니다.
2. 이 폴더의 `.github/workflows/build-apk.yml` 파일과 `README.md`를 올립니다.
3. Repository의 `Actions` → `Build WIPI Emulator APK`를 실행합니다.
4. 작업이 끝나면 workflow 하단의 **Artifacts → wipi-emulator-tq-diagnostic-apk**에서 APK를 받습니다.
5. APK 설치 후 기존에 가지고 있는 `택티컬퀘스트_WIPI_실행용.zip`을 에뮬레이터에 불러옵니다.

### GitHub Actions가 사용하는 공식 프로젝트

- WIPI Emulator: https://github.com/ParkJeongseop/WIPI-Emulator
- wie: WIPI Emulator가 지정한 Git revision을 그대로 사용
- RustJava: WIPI/wie의 `0.1.1` 계열 의존성을 그대로 사용

현재 업로드했던 `RustJava-main.zip`의 0.2.0 소스를 강제로 끼워 넣지 않습니다. 이것은 0.1.1 의존성과 버전이 달라서 호환성을 보장할 수 없기 때문입니다.

## APK가 나오지 않을 경우

Actions의 실패 단계 화면을 그대로 보내주시면 됩니다. 특히 다음 단계의 오류가 중요합니다.

- `Build Rust JNI`
- `Build Android APK`
- `Upload APK`

빌드 성공 여부와 별개로, 설치 후 키를 눌렀을 때 발생하는 `panic:` / `backtrace:` 로그가 다음 수정의 핵심 자료입니다.
