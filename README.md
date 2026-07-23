# paasctl

tidoly PaaS 설치 도구 배포 저장소입니다. 바이너리와 체크섬만 여기 있고, 소스는 별도 저장소에 있습니다.

## 설치

최신 버전을 받습니다. 사용하는 OS/아키텍처에 맞는 파일을 고르세요.

```bash
# Linux (x86_64)
curl -fsSLO https://github.com/tidoly-com/go-paas/releases/latest/download/paasctl_linux_amd64

# Linux (ARM64)
curl -fsSLO https://github.com/tidoly-com/go-paas/releases/latest/download/paasctl_linux_arm64

# macOS (Apple Silicon)
curl -fsSLO https://github.com/tidoly-com/go-paas/releases/latest/download/paasctl_darwin_arm64

# macOS (Intel)
curl -fsSLO https://github.com/tidoly-com/go-paas/releases/latest/download/paasctl_darwin_amd64
```

### 무결성 확인

받은 파일이 온전한지 확인합니다.

```bash
curl -fsSLO https://github.com/tidoly-com/go-paas/releases/latest/download/SHA256SUMS
sha256sum -c SHA256SUMS --ignore-missing
```

### 설치

```bash
chmod +x paasctl_*
sudo mv paasctl_* /usr/local/bin/paasctl
paasctl --version
```

## 이게 뭔가요

`paasctl` 은 SSH 로 접속 가능한 서버를 Kubernetes 기반 사내 PaaS 로 만들어 주는 설치 도구입니다.
설치가 끝나면 인증(SSO), 데이터베이스, 인그레스, 관측(로그·메트릭)이 함께 구성됩니다.

컨테이너 이미지는 별도 배포되며 별도 인증 없이 받을 수 있습니다.

```
ghcr.io/tidoly-com/paas-control-plane
ghcr.io/tidoly-com/paas-web-ui
```

## 지원 환경

- Linux (x86_64 / ARM64)
- 설치 대상 서버에 SSH 접근 권한

## 문의

설치·라이선스 문의는 담당자에게 연락 주세요.
