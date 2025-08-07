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
| `%exec` | 실행 | `execve`, `execveat` |
| `%clone` | 복제 | `clone`, `clone3` |
| `%kill` | 종료 | `kill`, `tkill`, `tgkill` |
| `%exit` | 종료 | `exit`, `exit_group` |
| `%epoll` | 이벤트 폴링 | `epoll_create1`, `epoll_ctl`, `epoll_pwait` 등 |
| `%inotify` | 파일 감시 | `inotify_init1`, `inotify_add_watch` 등 |
| `%attr` | 확장 속성 | `setxattr`, `getxattr`, `listxattr` 등 |
| `%dup` | 파일 디스크립터 복제 | `dup`, `dup3` |
| `%sched` | 스케줄링 | `sched_getparam`, `sched_setaffinity` 등 |

## 🔍 사용 예시

### 1. 파일 시스템 모니터링
```bash
# su 바이너리 접근 시도 추적
./stackplz_arm64 -n com.example.app -s %file -f w:su
# 출력 예시: [18320|18320|plefund.million] faccessat(...(/system/xbin/su)...)

# 시스템 디렉토리 접근 감시
./stackplz_arm64 -n com.example.app -s %file -f w:/system
```

### 2. 네트워크 활동 분석
```bash
# 모든 네트워크 활동 추적
./stackplz_arm64 -n com.browser.app -s %net,%send,%recv -o network.log
# connect, bind, sendto, recvfrom 등 모든 네트워크 활동 추적

# 특정 IP 대역 연결만 추적
./stackplz_arm64 -n com.example.app -s %net -f w:192.168
# 192.168로 시작하는 IP 연결만 출력
```

### 3. 프로세스 생명주기 추적
```bash
# 프로세스 생성/종료 모니터링
./stackplz_arm64 -n com.example.app -s %process,%exec,%exit
# clone, execve, exit 등 프로세스 생명주기 추적
```

### 4. 함수 레벨 hooking
```bash
# malloc 함수 호출 추적
./stackplz_arm64 -n com.example.app -w malloc[int] --kill SIGSTOP
# 예시: ./stackplz -n app --point malloc --kill SIGSTOP

# 문자열 함수 추적
./stackplz_arm64 -n com.example.app -w strstr[str,str]
```

## ⚙️ 주요 옵션

### 필수 옵션
- `-n, --name`: 추적할 앱 패키지명 또는 그룹
  - 패키지명: `com.kakao.talk`, `com.starbucks.cn` 등
  - 그룹: `app`(모든앱), `root`, `system`, `shell`, `iso`(격리된프로세스)
- `-s, --syscall`: 추적할 시스템콜 그룹 (예: `%file,%net`, `%all`)

### 타겟 지정 옵션
- `-p, --pid`: 추적할 PID (쉼표로 구분, 예: `1234,5678`)
- `-t, --tid`: 추적할 스레드 ID (쉼표로 구분)
- `-u, --uid`: 추적할 UID (쉼표로 구분, 예: `10084,10085`)
- `--tname`: 추적할 스레드명 (최대 16바이트, 예: `RenderThread,AsyncTask`)

### 제외 옵션 (블랙리스트)
- `--no-pid`: 제외할 PID (쉼표로 구분)
- `--no-tid`: 제외할 스레드 ID
- `--no-uid`: 제외할 UID (쉼표로 구분)
- `--no-tname`: 제외할 스레드명
- `--no-syscall`: 제외할 시스템콜 (최대 20개, 예: `openat,recvfrom`)

### 출력 및 로깅 옵션
- `-o, --out`: 로그 파일로 저장 (예: `network_trace.log`)
- `--json`: JSON 형식으로 출력
- `--color`: 로그 파일에 색상 적용
- `-q, --quiet`: 터미널 출력 숨김 (파일로만 저장)
- `--showtime`: 부팅 후 경과시간 출력 (나노초 단위)
- `--showuid`: 프로세스 UID 정보 출력
- `--showpc`: 스택의 원본 PC 레지스터 값 표시

### 필터링 옵션
- `-f, --filter`: 매개변수 필터 규칙 (배열로 여러 개 지정 가능)
  - `w:<문자열>`: 화이트리스트 - 지정 문자열 포함하는 이벤트만 출력
  - `b:<문자열>`: 블랙리스트 - 지정 문자열 포함하는 이벤트 제외
  - `eq:<값>`: LR 레지스터가 특정값과 같을 때
  - `bx:<hex>`: 버퍼 앞 바이트가 특정값일 때

### 함수 hooking 옵션
- `-w, --point`: uprobe 함수 hooking 포인트 설정 (배열로 여러 개 지정 가능)
  - 형식: `함수명[매개변수타입들]`
  - 예시1: `strstr[str,str]` (문자열 검색 함수)
  - 예시2: `write[int,buf:128,int]` (파일 쓰기, 128바이트 버퍼)
  - 예시3: `malloc[int]+0x10` (malloc 함수 + 16바이트 오프셋)
  - 예시4: `0x9D150[int,buf:x2,int]0x9D164` (절대주소+종료오프셋)
  - 매개변수 타입: `int`, `str`, `buf:크기` (예: `buf:128`, `buf:x2`)
- `-l, --lib`: hook할 라이브러리 이름 또는 전체 경로 (기본값: `libc.so` - 안드로이드 기본 C 라이브러리)
  - 예시: `libssl.so`, `/system/lib64/libc.so`
- `--maxop`: uprobe 최대 연산 수 (기본값: 64, 문자열 배열은 최소 192)
- `--kill`: uprobe hook 시 프로세스에 시그널 전송 (`SIGSTOP`, `SIGABRT`, `SIGTRAP`)
- `--tkill`: uprobe hook 시 스레드에 시그널 전송

### 하드웨어 브레이크포인트
- `--brk`: 하드웨어 브레이크포인트 주소 설정
  - 형식: `주소:타입` (타입: `r`=읽기, `w`=쓰기, `rw`=읽기쓰기, `x`=실행)
  - 예시: `0x70ddfd63f0:x`, `0x12345678:rw`
- `--brk-len`: 하드웨어 브레이크포인트 길이 (기본 4바이트, 1-8 지원)
- `--brk-lib`: 라이브러리 기준 주소로 브레이크포인트 설정 (`-p` 옵션과 함께 사용)
- `--brk-pid`: 브레이크포인트 적용할 PID (기본값: -1)

### 스택 분석 옵션
- `--stack`: 스택 트레이스 출력 활성화 (함수 호출 체인)
- `--stack-size`: 스택 덤프 크기 (기본 8192바이트, 최대 65528바이트)
- `--mstack`: 수동 스택 파싱 (심볼 정보 없이)
- `--jstack`: Java 스택 분석 시도 (`--kill SIGSTOP`과 함께 사용 필수)
  - 앱이 멈춘 후 'c' + Enter로 재개

### 레지스터 및 메모리 분석
- `--regs`: 모든 레지스터 값 출력
- `--reg`: 레지스터 오프셋 가져오기
- `--getoff`: PC/LR 오프셋 정보 출력 (성능 저하)
- `--dumpret`: 심볼의 return 오프셋 덤프
- `--dumphex`: 버퍼 데이터를 16진수로 출력 (CyberChef 스타일)

### 성능 및 시스템 옵션
- `-b, --buffer`: perf 캐시 버퍼 크기 (기본 8MB)
  - 데이터 손실이 발생하면 증가 (최대 32MB 권장)
  - 예시: `-b 16` (16MB 버퍼)
- `--auto`: `--kill SIGSTOP` 사용 시 자동 재개
- `--full-tname`: 기본 스레드 블랙리스트 해제 (시스템 스레드도 추적)
- `--btf`: BTF 활성화 선언
- `--nocheck`: BPF 기능 검사 비활성화 (`/proc/config.gz` 없을 때)

### 데이터 처리 옵션
- `--dump`: perf 데이터를 파일로 저장 (대용량 데이터 수집용)
- `--parse`: 덤프된 perf 데이터를 JSON/텍스트로 파싱
- `-c, --config`: JSON 형식의 설정 파일 사용 (배열로 여러 개 지정 가능)

### RPC 서버 옵션
- `--rpc`: RPC 서버 활성화 (Frida 연동용)
- `--rpc-path`: RPC 서버 주소 (기본: `127.0.0.1:41718`)

### 기타 옵션
- `--sdk-int`: Android SDK 버전 (선택사항)
- `-d, --debug`: 디버그 로깅 활성화
- `-h, --help`: 도움말 출력

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
