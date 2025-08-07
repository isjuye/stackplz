# 🎯 StackPlz - Android eBPF 시스템 추적 도구

Android 애플리케이션의 시스템콜, 함수 호출, 네트워크 활동을 실시간으로 추적하는 강력한 eBPF 기반 도구입니다.

## ⚡ 빠른 시작

```bash
# 기본 사용법
./stackplz_arm64 -n <패키지명> -s <syscall그룹> [-f <필터>] [-o <로그파일>]

# 예시: 카카오톡 파일 접근 추적
./stackplz_arm64 -n com.kakao.talk -s %file -o kakao_file.log
```

## 🚀 주요 기능

- **실시간 시스템콜 추적**: 파일, 네트워크, 프로세스 관련 시스템콜 모니터링
- **함수 레벨 hooking**: uprobe를 통한 특정 함수 호출 추적
- **하드웨어 브레이크포인트**: 메모리 접근 패턴 분석
- **Java 스택 분석**: Android 앱의 Java/Kotlin 스택 추적
- **유연한 필터링**: 화이트리스트/블랙리스트 기반 이벤트 필터링

## 📋 Syscall 그룹

| 그룹 | 설명 | 포함 시스템콜 |
|------|------|---------------|
| `%file` | 파일 접근 | `openat`, `faccessat`, `mkdirat`, `unlinkat` 등 |
| `%net` | 네트워크 | `socket`, `connect`, `bind`, `listen`, `accept` 등 |
| `%process` | 프로세스 관리 | `clone`, `execve`, `exit`, `wait4` 등 |
| `%send` | 네트워크 송신 | `sendto`, `sendmsg`, `sendmmsg` |
| `%recv` | 네트워크 수신 | `recvfrom`, `recvmsg`, `recvmmsg` |
| `%read` | 읽기 작업 | `read`, `readv`, `pread64`, `preadv` 등 |
| `%write` | 쓰기 작업 | `write`, `writev`, `pwrite64`, `pwritev` 등 |
| `%signal` | 시그널 처리 | `rt_sigaction`, `rt_sigprocmask` 등 |
| `%mount` | 마운트 작업 | `mount`, `umount2`, `move_mount` 등 |
| `%stat` | 상태 조회 | `statfs`, `fstat`, `newfstatat`, `statx` 등 |

## 🔍 사용 예시

### 1. 파일 시스템 모니터링
```bash
# su 바이너리 접근 시도 추적
./stackplz_arm64 -n com.example.app -s %file -f w:su

# 시스템 디렉토리 접근 감시
./stackplz_arm64 -n com.example.app -s %file -f w:/system
```

### 2. 네트워크 활동 분석
```bash
# 모든 네트워크 활동 추적
./stackplz_arm64 -n com.browser.app -s %net,%send,%recv -o network.log

# 특정 IP 대역 연결만 추적
./stackplz_arm64 -n com.example.app -s %net -f w:192.168
```

### 3. 프로세스 생명주기 추적
```bash
# 프로세스 생성/종료 모니터링
./stackplz_arm64 -n com.example.app -s %process,%exec,%exit
```

### 4. 함수 레벨 hooking
```bash
# malloc 함수 호출 추적
./stackplz_arm64 -n com.example.app -w malloc[int] --kill SIGSTOP

# 문자열 함수 추적
./stackplz_arm64 -n com.example.app -w strstr[str,str]
```

## ⚙️ 주요 옵션

### 필수 옵션
- `-n, --name`: 추적할 앱 패키지명 (예: `com.kakao.talk`)
- `-s, --syscall`: 추적할 시스템콜 그룹 (예: `%file,%net`)

### 출력 옵션
- `-o, --out`: 로그 파일로 저장
- `--json`: JSON 형식으로 출력
- `--color`: 로그 파일에 색상 적용
- `-q, --quiet`: 터미널 출력 숨김

### 필터링 옵션
- `-f, --filter`: 매개변수 필터 규칙
  - `w:<문자열>`: 화이트리스트 (포함)
  - `b:<문자열>`: 블랙리스트 (제외)
  - `eq:<값>`: LR 레지스터 값 비교
  - `bx:<hex>`: 버퍼 내용 비교

### 고급 옵션
- `-w, --point`: uprobe 함수 hooking 포인트
- `--brk`: 하드웨어 브레이크포인트 설정
- `--stack`: 스택 트레이스 출력
- `--jstack`: Java 스택 분석
- `--kill`: hook 시 프로세스에 시그널 전송

## 🛠️ 설치 요구사항

- **권한**: Root 권한 필수
- **커널**: Linux 5.10 이상
- **플랫폼**: Android
- **아키텍처**: ARM64

## 📝 사용 팁

### 성능 최적화
```bash
# 버퍼 크기 조정 (데이터 손실 방지)
./stackplz_arm64 -n com.app -s %file -b 16

# 특정 스레드만 추적
./stackplz_arm64 -n com.app -s %net --tname RenderThread
```

### 디버깅
```bash
# 디버그 모드 활성화
./stackplz_arm64 -n com.app -s %file -d

# 모든 레지스터 값 출력
./stackplz_arm64 -n com.app -w malloc[int] --regs
```

### 데이터 분석
```bash
# perf 데이터 덤프 및 분석
./stackplz_arm64 -n com.app -s %file --dump trace.data
./stackplz_arm64 --parse trace.data --json > analysis.json
```

## 🔧 고급 사용법

### 하드웨어 브레이크포인트
```bash
# 특정 주소의 실행 추적
./stackplz_arm64 -n com.app --brk 0x70ddfd63f0:x

# 메모리 읽기/쓰기 추적
./stackplz_arm64 -n com.app --brk 0x12345678:rw --brk-len 8
```

### Java 스택 분석
```bash
# Java 스택과 함께 분석
./stackplz_arm64 -n com.app -w malloc[int] --kill SIGSTOP --jstack
# 앱이 멈추면 'c' + Enter로 재개
```

### RPC 서버 모드
```bash
# Frida와 연동을 위한 RPC 서버
./stackplz_arm64 --rpc --rpc-path 127.0.0.1:41718
```

## 🤝 기여하기

버그 리포트나 기능 제안은 Issues 탭을 통해 제출해주세요.

## 📄 라이선스

이 프로젝트는 오픈소스 라이선스를 따릅니다.

---

**⚠️ 주의사항**: 이 도구는 보안 연구 및 앱 디버깅 목적으로만 사용하세요. 악용하지 마시기 바랍니다.
