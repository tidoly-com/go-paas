# paasctl

tidoly PaaS 설치 도구입니다. 서버 한 대를 Kubernetes 기반 사내 PaaS 로 만들어 줍니다 —
Kubernetes 부터 인증(SSO)·데이터베이스·인그레스·HTTPS 인증서·관측(로그/메트릭)까지 한 번에.

이 저장소에는 **실행 파일과 체크섬만** 있습니다. 소스는 별도 저장소에 있습니다.

---

## 설치 전 준비물

| | 무엇 | 비고 |
|---|---|---|
| 1 | **서버 1대** | Ubuntu 계열, x86_64 또는 ARM64. 아무것도 안 깔려 있어도 됩니다 — Kubernetes 도 설치기가 깝니다 |
| 2 | **라이선스 키** | 발급받은 키. 설치에 필요한 구성요소를 이 키로 받아옵니다 |
| 3 | **도메인** | 예: `paas.example.com`. 관리 화면은 `app.도메인`, 배포한 앱은 `앱이름.도메인` 으로 열립니다 |
| 4 | **Cloudflare API 토큰** | `Zone:DNS:Edit` 권한. HTTPS 인증서 발급에 씁니다 — **사내망 서버라면 필수**입니다 |
| 5 | **Git 저장소 + 배포 키** | 배포 상태를 기록할 빈 저장소 하나. SSH 개인키를 서버에 두면 됩니다 |

4번을 비우면 서버의 80 포트가 외부에 열려 있어야 인증서를 받을 수 있습니다.
사내망·사설 IP 환경이면 Cloudflare 토큰 방식(DNS 인증)을 쓰세요.

> 백업용 S3 와 메일 발송(SMTP)은 설치 후 관리 화면에서 등록합니다 — 설치 시점에는 필요 없습니다.

---

## 받기

### Mac 에서 (원격 서버에 설치)

압축을 풀고 **더블클릭**하면 설치 화면이 브라우저에 뜹니다.

```bash
# Apple Silicon
curl -fsSLO https://github.com/tidoly-com/go-paas/releases/latest/download/paasctl_macos_arm64.app.zip
unzip paasctl_macos_arm64.app.zip

# Intel Mac 은 paasctl_macos_amd64.app.zip
```

브라우저로 내려받았다면 macOS 가 실행을 막습니다(서명 전입니다). 한 번만 풀어주세요:

```bash
xattr -dr com.apple.quarantine paasctl.app
```

위처럼 `curl` 로 받으면 이 과정이 필요 없습니다.

### 서버에서 직접

```bash
# ARM64
curl -fsSLO https://github.com/tidoly-com/go-paas/releases/latest/download/paasctl_linux_arm64
# x86_64 는 paasctl_linux_amd64

chmod +x paasctl_linux_*
sudo mv paasctl_linux_* /usr/local/bin/paasctl
```

### 받은 파일 확인

```bash
curl -fsSLO https://github.com/tidoly-com/go-paas/releases/latest/download/SHA256SUMS
sha256sum -c SHA256SUMS --ignore-missing
```

---

## 설치하기

### 화면으로

Mac 이라면 `paasctl.app` 을 더블클릭하세요. 서버에서 직접 한다면 인자 없이 실행합니다.

```bash
paasctl
```

브라우저에 설치 화면이 열립니다. 준비물을 순서대로 입력하고 시작하면 진행 상황이 단계별로 보입니다.
끝나면 **접속 주소와 최초 비밀번호**가 화면에 나오고, 파일로 내려받을 수 있습니다.

### 터미널로

```bash
export DOMAIN=paas.example.com
export ACME_EMAIL=infra@example.com
export CF_API_TOKEN=...          # Zone:DNS:Edit
export CF_DNS_ZONE=example.com   # Cloudflare 에 등록된 영역(Zone) 이름
export LICENSE_KEY=...
export PAAS_LICENSE_CENTRAL_URL=...          # 라이선스와 함께 안내받은 주소
export PAAS_CLUSTER_GITOPS_REPO_URL=git@github.com:회사/저장소.git
export GITOPS_SSH_KEY_PATH=/root/.ssh/gitops

sudo -E paasctl apply
```

Mac 에서 원격 서버를 설치하려면 `--host` 를 줍니다. `~/.ssh/config` 의 별칭이 그대로 통합니다.

```bash
paasctl apply --host myserver
```

> 비밀값은 셸 히스토리에 남지 않도록 파일에 두고 읽는 편이 안전합니다:
> `set -a; . /root/.paas-install.env; set +a`

---

## 설치 후

### 접속 정보 다시 보기

최초 비밀번호는 무작위로 만들어져 클러스터 안에만 있습니다. 언제든 다시 꺼낼 수 있습니다.

```bash
paasctl credentials              # 화면에 표시
paasctl credentials --out 파일   # 파일로 저장 (권한 0600)
paasctl credentials --host myserver
```

관리 화면(`https://app.도메인`)에 이 계정으로 로그인하세요.
**첫 로그인 후 비밀번호를 바꾸시길 권합니다.**

### 중간에 멈췄다면

완료한 단계는 기록되므로, 원인을 고친 뒤 같은 명령을 다시 실행하면 **멈춘 곳부터 이어서** 진행합니다.
처음부터 다시 하지 않아도 됩니다.

### 외부 인터넷이 막힌 환경

설치에 필요한 구성요소는 라이선스로 내려받습니다. 서버가 외부에 나갈 수 없다면
인터넷이 되는 곳에서 미리 받아 옮기세요.

```bash
# 인터넷 되는 곳에서
LICENSE_KEY=... PAAS_LICENSE_CENTRAL_URL=... paasctl assets fetch --out bundle.zip

# 설치 대상 서버에서
sudo paasctl apply --charts-bundle bundle.zip
```

---

## 지원 환경

- **설치 대상 서버**: Linux (x86_64 / ARM64) — Ubuntu 계열
- **설치를 실행하는 곳**: 서버에서 직접, 또는 macOS·Linux 에서 SSH 로 원격
- **컨테이너 이미지**: 별도 인증 없이 받을 수 있습니다

```
ghcr.io/tidoly-com/paas-control-plane
ghcr.io/tidoly-com/paas-web-ui
```

### 노드 확장 — 추후 지원

현재 설치기는 **서버 1대(단일 노드)** 를 구성합니다.

워커 노드를 붙이는 것 자체는 가능하지만 아직 자동화되어 있지 않습니다 — 서버에서 join 토큰을
직접 복사해 워커에서 실행하는 **수동 절차**가 필요하고, `paasctl` 명령으로 제공되지 않습니다.
2대 이상일 때의 인그레스 이중화(VIP) 설정도 마찬가지로 수동입니다.

다음 형태로 지원할 예정입니다. 필요하시면 담당자에게 문의 주세요 — 수동 절차를 안내드립니다.

```bash
paasctl node add --host worker1    # (예정) 토큰을 자동으로 가져와 join
paasctl node list                  # (예정)
```

---

## 문의

설치·라이선스 문의는 담당자에게 연락 주세요.
admin@tidoly.com
