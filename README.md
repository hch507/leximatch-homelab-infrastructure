# LexiMatch Homelab Infrastructure

VMware 기반으로 기업 사내 인프라 환경을 구축하고, 네트워크 망분리, 라우팅, 방화벽(ACL), DNS, 3-Tier Web Architecture(WEB/WAS/DB) 운영을 직접 설계하고 검증한 시스템 엔지니어링 홈랩 프로젝트입니다.

---

## Architecture

```text
                                GW01 (Gateway)
                                      │
        ┌─────────────────────────────┼─────────────────────────────┼─────────────────────────────┐
        │                             │                             │                             │
     직원망                         개발망                       개발서버망                       서버망
    VMnet10                        VMnet20                        VMnet13                        VMnet30
 192.168.10.0/24                192.168.20.0/24                192.168.40.0/24                192.168.30.0/24
        │                             │                             │                             │
     직원 PC                        개발자 PC                     Dev Server                         │
                                                                                                  │
                                                                                 ┌────────┬────────┬────────┬
                                                                                 │        │        │        │
                                                                               DNS01    WEB01    WAS01    DB01
                                                                               BIND9    Nginx  Spring Boot MySQL
                                                                           192.168.30.10  192.168.30.20  192.168.30.40  192.168.30.30
```

## Core Services & IP Summary

| **Hostname** | **Service**       | **IP Address**  | **Domain / Role**                       |
| ------------ | ----------------- | --------------- | --------------------------------------- |
| **GW01**     | Ubuntu / IPTables | `192.168.30.1`  | L3 Routing, NAT & Firewall Gateway      |
| **DNS01**    | BIND9             | `192.168.30.10` | `dns.momantle.internal` (Internal DNS)  |
| **WEB01**    | Nginx             | `192.168.30.20` | `web.momantle.internal` (Reverse Proxy) |
| **DB01**     | MySQL 8.0         | `192.168.30.30` | `db.momantle.internal` (Database)       |
| **WAS01**    | Spring Boot       | `192.168.30.40` | `was.momantle.internal` (Backend API)   |
| **DEV01**    | Dev Server        | `192.168.40.10` | `dev.momantle.internal` (Build/Test)    |

## Project Goals

- **보안 망분리:** 4개 역할별 L2 브로드캐스트 도메인 격리 (직원/개발/서버/개발서버)
- **중앙 라우팅 & NAT:** GW01 단일 접점을 통한 트래픽 라우팅 및 Outbound SNAT 통제
- **최소 권한 접근 제어:** IPTables 기반 망 간 Default Deny ACL 방화벽 정책 구현
- **3-Tier 웹 서비스 연동:** Nginx - Spring Boot - MySQL 서비스 체인 및 BIND9 내부 DNS 구축
- **체계적 문서화 & 검증:** 설정 파일 버전 관리 및 `tcpdump`, `nc` 기반 통신/차단 검증

## Repository Structure

```text
.
├── README.md
├── docs/                      # 상세 설계 및 검증 문서
│   ├── 01-topology.md         # 전체 인프라 아키텍처 다이어그램
│   ├── 02-network.md          # IP, Subnet, Routing & NAT 설계서
│   ├── 03-access-policy.md    # IPTables ACL 접근 제어 정책서
│   ├── 04-accounts.md         # OS 계정 및 최소 권한 관리 명세
│   ├── 05-services.md         # DNS, WEB, WAS, DB 서비스 구축 명세
│   ├── 06-test-results.md     # 망 간 통신 및 방화벽 검증 결과
│   └── 07-troubleshooting.md  # 장애 유형별 분석 및 해결 기록
├── config/                    # 인프라 서버 설정 원문 파일
│   ├── gateway/               # Netplan, Kea DHCP, IPTables 룰셋
│   ├── dns/                   # BIND9 named.conf 및 Zone 파일
│   ├── web/                   # Nginx reverse proxy conf
│   └── db/                    # MySQL my.cnf 및 계정 권한 스크립트
└── scripts/                   # 자동화 및 검증 스크립트
```

## Documentation Links

| **문서 명세** | **설명** | **바로가기** |
|---|---|---|
| **01. Topology** | 네트워크 구조 및 구성도 | [문서 보기](docs/01-topology.md) |
| **02. Network Design** | IP 구성, DHCP, 라우팅 및 NAT 명세 | [문서 보기](docs/02-network.md) |
| **03. Access Policy** | IPTables 매트릭스 및 방화벽 룰셋 | [문서 보기](docs/03-access-policy.md) |
| **04. Account & Permission** | Linux 계정 권한 및 Sudoers 정책 | [문서 보기](docs/04-accounts.md) |
| **05. Services** | BIND9, Nginx, Spring Boot, MySQL 설정 | [문서 보기](docs/05-services.md) |
| **06. Test Results** | 서비스 동작 및 방화벽 차단 검증 | [문서 보기](docs/06-test-results.md) |
| **07. Troubleshooting** | 인프라 구축 중 발생한 장애 해결 기록 | [문서 보기](docs/07-troubleshooting.md) |

## Environment & Tech Stack

- **Virtualization:** VMware Workstation
- **OS:** Ubuntu Server 22.04 LTS
- **Network & Security:** IPTables, Netplan, Kea DHCP
- **Services:** BIND9 (DNS), Nginx (Web/Reverse Proxy), Spring Boot (WAS), MySQL 8.0 (DB)
