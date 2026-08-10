# Operation Follow Me (FM 8+)

[이스트캠프] 가디언즈 정보보호 및 인프라 관리 10기 3차 팀 프로젝트
**모의해킹 기반 FM 8+ 정보보호 진단 · 팔로팔로미(= "Follow Follow me")**

## 프로젝트 개요

| 항목 | 내용 |
|---|---|
| 프로젝트명 | Operation Follow Me (FM 8+) |
| 진행 기간 | 2026.07.23 ~ 2026.08.04 |
| 팀 구성 | 팔로팔로미 \| 김혜미(팀장) 외 4명 |
| 핵심 기술 | Firewall(pfSense), IDS·IPS(Suricata), WAF(ModSecurity), SIEM(ELK Stack) |

**선정 배경**
2025년 4월 S사 HSS 침해사고(USIM 인증키 평문 저장, 탐지까지 약 22시간 지연, 다수 서버 확산, 축소 보고)를 근본 원인 관점에서 재구성함.

**프로젝트 목표**
가상의 통신사 인프라(FM 8+)를 직접 구축한 뒤, Red Team(모의해킹)과 Blue Team(보안관제)이 같은 환경을 서로 다른 관점에서 진단해, 취약점 진단 → 실시간 탐지·차단 → 로그 통합분석(SIEM)까지 이어지는 침해사고 대응 체계를 실증함.

## 팀 구성 (WBS)

| 이름 | 역할 | 담당 업무 |
|---|---|---|
| 김혜미 | 팀장 / 모의해킹·악성코드 분석 | OWASP Top 10 취약점 선정, 악성코드 동적 분석, 보고서 작성 |

## 기술 스택

| 구분 | 기술 |
|---|---|
| OS | Ubuntu Server 24.04 |
| 가상화 | GNS3 |
| 공격 도구 | Kali Linux |
| DB | MariaDB |
| 보안 장비 | pfSense (Firewall) · Suricata (IDS/IPS) · ModSecurity (WAF) |
| SIEM | ELK Stack (Elasticsearch, Logstash, Kibana) |

## 저장소 구성

| 문서 | 내용 |
|---|---|
| [`1_팔로팔로미_3차팀프로젝트최종보고서.pdf`](<./1_팔로팔로미_3차팀프로젝트최종보고서.pdf>) | 최종 보고서 — 개요, 팀 구성/역할, 수행절차, 기술스택, 인프라·네트워크 구성도, 악성코드/모의해킹/기업정보보호 점검 결과 종합, 자체평가 |
| [`2_팔로팔로미_3차팀프로젝트계획서.pdf`](<./2_팔로팔로미_3차팀프로젝트계획서.pdf>) | 프로젝트 계획서 — 선정배경, 최종목표, 수행범위, 기술스택, 네트워크 구성도, WBS, 일정계획 |
| [`3_팔로팔로미_모의해킹결과보고서.pdf`](<./3_팔로팔로미_모의해킹결과보고서.pdf>) | 모의해킹(Red Team) 결과보고서 — FM 8+ 웹 애플리케이션 취약점 5건 재현·근본원인·대응방안 |
| [`4_팔로팔로미_악성코드분석보고서.pdf`](<./4_팔로팔로미_악성코드분석보고서.pdf>) | 악성코드 분석보고서 — BPFDoor 기반 리버스쉘 백도어 정적/동적 분석 및 IoC |
| [`5_팔로팔로미_시스템점검보고서.pdf`](<./5_팔로팔로미_시스템점검보고서.pdf>) | 시스템 점검보고서 — KISA 가이드 기반 U-01~U-67 서버 취약점 진단 및 조치 결과 |
| [`6_팔로팔로미_시스템점검쉘스크립트.txt`](<./6_팔로팔로미_시스템점검쉘스크립트.txt>) | 시스템 점검 자동화 스크립트 원본(Bash) — U-01~U-67 항목 자동 점검 |
| [`7_팔로팔로미_기업정보보호점검결과보고서.pdf`](<./7_팔로팔로미_기업정보보호점검결과보고서.pdf>) | 기업 정보보호 점검 결과보고서 — Red Team 진단 + Blue Team 관제 검증 통합, 탐지 커버리지 분석 |

## 수행 결과 요약

### 모의해킹(Red Team) — FM 8+ 웹 애플리케이션

총 5건의 취약점을 모두 실증(성공률 100%)했으며, 이 중 3건(SQL Injection, Stored XSS, Unrestricted File Upload)은 치명(Critical) 등급입니다. Stored XSS와 Session Fixation을 결합하면 인증 절차 없이 관리자 세션을 탈취할 수 있음을 확인했습니다.

| # | 취약점 | 심각도 | 핵심 근본원인 |
|---|---|---|---|
| 1 | SQL Injection | Critical | 입력값 미검증 쿼리 결합 (Prepared Statement 미적용) |
| 2 | Session Fixation | High | 로그인 성공 시 세션 재발급(session_regenerate_id) 누락 |
| 3 | Stored XSS | Critical | 출력 시 HTML 인코딩 미적용 |
| 4 | IDOR | High | 리소스 소유자 검증(인가) 로직 부재 |
| 5 | Unrestricted File Upload | Critical | 확장자/파일 유형 서버측 검증 부재 |

### 보안관제(Blue Team) — 탐지·차단 검증

pfSense(Firewall) → Suricata(IDS/IPS) → ModSecurity(WAF) → ELK Stack(SIEM) 파이프라인으로 Red Team의 공격을 탐지·차단하고 대시보드로 시각화했습니다.

- 커스텀 탐지·차단 룰 18종(IDS/IPS 15 · WAF 3) 적용, 이 중 16종이 실제 차단(IPS drop 13 · WAF 403 거부 3) 동작 확인
- SIEM 대시보드로 정찰(포트 스캔)부터 웹 애플리케이션 공격(SQLi/XSS)까지 정상 트래픽과 분리된 실시간 탐지 검증
- 탐지 커버리지: 확실 탐지 2건 · 제한적 탐지 2건 · 사각지대 1건(IDOR — 정상 요청과 형태가 유사해 alert 전용으로 운영)
- 주요 미비점: 자동 경보(Alert) 체계 및 공격자 IP 동적 차단(SOAR) 미구축, 고정 시그니처 기반 탐지로 인코딩 변형·명령어 치환 등 우회에 취약

### 시스템 점검(서버 하드닝)

KISA 서버 취약점 점검 가이드 기반 자동화 스크립트(U-01~U-67)로 점검한 9개 핵심 항목(U-01/02/03/06/07/12/57/62/66) 모두 조치 전 '취약' → 조치 후 '양호'로 전환(조치율 100%).

| 위험도 | 항목 수 | 해당 항목 |
|---|---|---|
| 상 | 5 | U-01, U-02, U-03, U-57, U-66 |
| 중 | 1 | U-12 |
| 하 | 3 | U-06, U-07, U-62 |

### 악성코드 분석 — BPFDoor 기반 리버스쉘 백도어

VirusTotal, Detect It Easy, file/strings/nm/readelf/objdump, Ghidra를 이용한 정적 분석과 Wireshark·strace 기반 동적 분석을 병행해 매직 패킷 트리거(`is_magic_packet`) 및 리버스쉘 실행(`spawn_reverse_shell`) 로직, C2 통신 방식, 탐지 회피 특징을 분석하고 IoC와 대응 권고를 도출했습니다.

## 스크립트 사용법

`6_팔로팔로미_시스템점검쉘스크립트.txt`는 KISA 가이드 U-01~U-67 항목을 자동 점검하는 Bash 스크립트입니다(Ubuntu 대상).

```bash
chmod +x check.sh
./check.sh all          # 전체 67개 항목 점검 결과 출력
./check.sh vulnerable   # 취약 판정된 항목만 출력
```
