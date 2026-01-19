# 🏛️ System Architecture Design

이 문서는 '지능형 사내 업무 지원 서비스'의 전체적인 구조와 데이터 흐름, 그리고 보안 설계 원칙을 기술합니다.

> 💡 기술 스택별 상세 선정 이유(ADR)는 [`./tech-stack/`](./tech-stack/) 디렉토리의 개별 문서에서 확인하실 수 있습니다.

---

## 🖼️ Architecture Diagram

<p align="center">
    <img src="./docs/images/system-architecture.png" width="100%" alt="System Architecture">
</p>

---

## 🔄 Data & Request Flow (시스템 흐름)

시스템의 데이터는 사용자의 요청에 따라 **On-demand** 방식으로 실시간 처리되며, 각 계층은 유기적으로 연결되어 있습니다.

1. **Service Entry (Frontend)**
   - 사용자가 도메인에 접속하면 **Vercel**의 Edge Network를 통해 최적화된 프론트엔드 리소스가 서빙됩니다.
   - **Next.js App Router** 기반의 SSR(Server-side Rendering) 페이지가 즉시 렌더링되어 최상의 초기 로딩 성능을 제공합니다.

2. **Authentication & Interaction (Backend)**
   - 사용자가 로그인을 완료하고 서비스를 이용하면, 모든 요청은 **AWS API Gateway**를 거칩니다.
   - 백엔드의 심장부인 **AWS Lambda(NestJS)**로 요청이 전달되어 비즈니스 로직이 실행됩니다.

3. **On-Demand Data Collection (Edge)**
   - 특정 데이터가 필요한 시점에 클라우드 서버는 **Raspberry Pi(Collector)**에 명령을 전달합니다.
   - 필요 시에만 정보를 가져오는 구조로 리소스를 최적화했습니다.

4. **Intelligence Processing (AI)**
   - 수집된 데이터는 다시 클라우드로 전송되어 **Deepgram(STT)** 및 **Multi-Model LLM Orchestration** 레이어를 거칩니다.
   - 가공된 원천 정보는 AI 모델을 통해 고차원의 인사이트로 변환됩니다.

5. **Persistent Storage (Database)**
   - 가공 완료된 정보는 **MongoDB Atlas**에 정형화되어 저장됩니다.
   - 대용량 미디어 파일 및 자막 데이터 등은 **AWS S3**에 안전하게 아카이빙됩니다.

---

## 🛡️ Security & Connectivity (보안 설계)

본 시스템은 사내망 보안을 유지하면서 클라우드의 유연함을 활용하기 위해 **3중 보안 계층**을 설계했습니다.

* **Inbound Zero-Exposure**: 라즈베리 파이 서버는 **ufw(방화벽)** 설정을 통해 모든 외부 인바운드 포트를 엄격히 차단합니다.
* **Secure Tunneling**: **Cloudflare Tunnel**을 활용하여 고정 IP나 포트 포워딩 없이 아웃바운드 기반의 암호화된 터널을 생성, 사내망을 외부 위협으로부터 완전하게 격리했습니다.
* **Encrypted Communication**: 모든 데이터 이동 구간(Edge to Cloud, Cloud to Database)은 **TLS/SSL 기반의 암호화 통신**을 기본으로 하여 데이터 무결성을 보장합니다.