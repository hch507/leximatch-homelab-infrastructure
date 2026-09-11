# LexiMatch Homelab Infrastructure

기업 사내 인프라 환경을 가정하여 VMware 기반의 네트워크 및 서버 인프라를 구축했습니다. 네트워크 망분리, GW01을 통한 라우팅과 NAT, IPTables 기반 접근 제어, 내부 DNS 구성 및 서버별 네트워크 정책을 직접 설계하고 테스트했습니다.

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
| **GW01**     | Ubuntu / IPTables | 192.168.10.1<br>192.168.20.1<br>192.168.30.1<br>192.168.40.1 | L3 Routing, NAT & Firewall Gateway      |
| **DNS01**    | BIND9             | `192.168.30.10` | `dns.momantle.internal` (Internal DNS)  |
| **WEB01**    | Nginx             | `192.168.30.20` | `web.momantle.internal` (Reverse Proxy) |
| **DB01**     | MySQL 8.0         | `192.168.30.30` | `db.momantle.internal` (Database)       |
| **WAS01**    | Spring Boot       | `192.168.30.40` | `was.momantle.internal` (Backend API)   |
| **DEV01**    | Dev Server        | `192.168.40.10` | `dev.momantle.internal` (Build/Test)    |

## Project Goals

- **보안 망분리:** 4개 역할별 L2 브로드캐스트 도메인 격리 (직원/개발/서버/개발서버)
- **중앙 라우팅 & NAT:** GW01 단일 접점을 통한 네트워크 간 트래픽 라우팅 및 Outbound SNAT 구성
- **최소 권한 접근 제어:** IPTables 기반 망 간 Default Deny ACL 방화벽 정책 구현
- **내부 인프라 서비스 구성:** Kea DHCP 및 BIND9 기반 내부 네트워크 서비스 구축
- **체계적인 검증:** `ping`, `nslookup`, `curl`, `ssh`, `nc`, `tcpdump`를 활용한 통신 및 접근 차단 검증

## Repository Structure

```text
.
├── README.md
├── docs/                      # 상세 설계 및 검증 문서
   ├── 01-topology.md         # 전체 인프라 아키텍처 다이어그램
   ├── 02-network.md          # IP, Subnet, Routing & NAT 설계서
   ├── 03-access-policy.md    # IPTables ACL 접근 제어 정책서
   ├── 04-test-results.md     # 망 간 통신 및 방화벽 검증 결과
   ├── 05-services.md         # DNS, WEB, WAS, DB 서비스 구축 명세
```

## Documentation Links

| **문서 명세** | **설명** | **바로가기** |
|---|---|---|
| **01. Topology** | 네트워크 구조 및 구성도 | [문서 보기](docs/01.%20topology.png) |
| **02. Network Design** | IP 구성, DHCP, 라우팅 및 NAT 명세 | [문서 보기](docs/02.%20network.md) |
| **03. Access Policy** | IPTables 매트릭스 및 방화벽 룰셋 | [문서 보기](docs/03.%20access%20policy.md) |
| **04. Test Result** | 서비스 동작 및 방화벽 차단 검증 | [문서 보기](docs/04.%20test%20result.md) |
| **05. Service** | BIND9, Nginx, Spring Boot, MySQL 설정 | [문서 보기](docs/05.%20service.md) |


## Environment & Tech Stack

- **Virtualization:** VMware Workstation
- **OS:** Ubuntu Server 22.04 LTS
- **Network & Security:** IPTables, Netplan, Kea DHCP
- **Services:** BIND9 (DNS), Nginx (Web/Reverse Proxy), Spring Boot (WAS), MySQL 8.0 (DB)
