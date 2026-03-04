# 옥수주조 스마트스토어 자동화 — 테크니컬 리포트

## 1. 프로젝트 개요

**클라이언트:** 옥수주조 (막걸리 제조회사)
**목표:** 네이버 스마트스토어 판매 다각화를 위한 운영 자동화
**전략:** 운영 리스크가 낮은 영역부터 단계적 도입

옥수주조는 스마트스토어 운영 경험이 없으므로, 파일럿 프로젝트로 기술 검증 후 본 프로젝트로 전환하는 안전 우선 전략을 채택합니다.

---

## 2. 자동화 유즈케이스 (3건)

| Phase | 유즈케이스 | 리스크 | 핵심 API | 도입 시점 |
|-------|-----------|--------|----------|----------|
| **P1** | 고객문의 자동응답 | LOW | 문의조회/답변등록/답변수정 | 즉시 착수 |
| **P2** | 시즌상품 등록/관리 | MID | 상품등록/수정, 카테고리조회 | P1 안정화 후 |
| **P3** | 주문→발송 자동처리 | HIGH | 주문수집/발주확인/송장등록 | 운영 숙련 후 |

### P1. 고객문의 자동응답 (파일럿)

막걸리 특성상 반복 문의가 대부분입니다:
- 유통기한 (~25%), 보관방법 (~20%), 도수 (~15%), 배송 (~15%), 선물포장 (~10%)

AI(Claude API)로 문의 의도를 분류하고, 신뢰도 0.85 이상일 때만 자동 답변합니다.
오답변 시 수정 API로 즉시 보정 가능하여 운영 리스크가 가장 낮습니다.

### P2. 시즌상품 등록/관리

봄 매실막걸리, 여름 수박막걸리 등 시즌 한정주를 스프레드시트에서 자동 등록합니다.
주류 법적 표기사항(도수, 용량, 원산지 등)을 템플릿으로 자동 매핑합니다.

### P3. 주문→발송 자동처리

주문 접수 → 발주확인 → 택배 송장 발급 → 발송처리 전체를 자동화합니다.
유통기한이 짧은 막걸리 특성상 당일출고가 핵심이지만, 오류 시 배송 지연/클레임으로 직결되므로 충분한 운영 경험 축적 후 도입합니다.

---

## 3. 기술 스택

| 구성요소 | 기술 | 역할 |
|----------|------|------|
| Orchestration | n8n (self-hosted or cloud) | 워크플로우 자동화 엔진 |
| Database | Supabase (PostgreSQL) | FAQ 템플릿, 이력 로그, 상품 마스터 |
| AI | Claude API (Sonnet) | 문의 의도 분류, FAQ 매칭 |
| External API | 네이버 커머스 API v2.71.0 | 스마트스토어 연동 |
| 알림 | Slack Webhook / 카카오 알림톡 | 에스컬레이션, 출고 알림 |

---

## 4. 네이버 커머스 API 엔드포인트 (공식 문서 기반)

> 출처: [커머스API 공식 문서](https://apicenter.commerce.naver.com/docs/commerce-api/current) v2.71.0

### 4-1. 문의 API (P1)

| Method | Endpoint | 설명 |
|--------|----------|------|
| `GET` | `/v1/pay-user/inquiries` | 고객 문의 조회 |
| `GET` | `/v1/contents/qnas` | 상품 문의 목록 조회 |
| `GET` | `/v1/contents/qnas/templates` | 답변 템플릿 목록 조회 |
| `POST` | `/v1/pay-merchant/inquiries/:inquiryNo/answer` | 고객 문의 답변 등록 |
| `PUT` | `/v1/pay-merchant/inquiries/:inquiryNo/answer/:answerContentId` | 고객 문의 답변 수정 |
| `PUT` | `/v1/contents/qnas/:questionId` | 상품 문의 답변 등록/수정 |

### 4-2. 상품 API (P2)

| Method | Endpoint | 설명 |
|--------|----------|------|
| `GET` | `/v1/categories` | 카테고리 목록 조회 |
| `GET` | `/v1/categories/:categoryId` | 카테고리 상세 조회 |
| `POST` | `/v2/products` | 상품 등록 |
| `GET` | `/v2/products/channel-products/:channelProductNo` | 채널상품 조회 |
| `PUT` | `/v2/products/channel-products/:channelProductNo` | 채널상품 수정 |
| `PUT` | `/v2/products/origin-products/:originProductNo` | 원상품 수정 |
| `PUT` | `/v1/products/origin-products/:originProductNo/change-status` | 상품 상태 변경 |

### 4-3. 주문 API (P3)

| Method | Endpoint | 설명 |
|--------|----------|------|
| `GET` | `/v1/pay-order/seller/product-orders/last-changed-statuses` | 변경 주문 내역 조회 |
| `GET` | `/v1/pay-order/seller/product-orders` | 상품주문 목록 조회 |
| `POST` | `/v1/pay-order/seller/product-orders/query` | 상품주문 상세 조회 |
| `GET` | `/v1/pay-order/seller/orders/:orderId/product-order-ids` | 주문별 상품주문 ID 조회 |
| `POST` | `/v1/pay-order/seller/product-orders/confirm` | 발주 확인 |
| `POST` | `/v1/pay-order/seller/product-orders/dispatch` | 발송 처리 (송장 등록) |

### 4-4. 인증

- OAuth 2.0 기반 (client_id / client_secret)
- Rate Limit: ~2 req/sec (내스토어 앱 기준)

---

## 5. 업무 진행 절차

```
[파일럿 프로젝트]                    [본 프로젝트]
요건정의 → 개발 → 검수  ──Go/No-Go──→  요구사항정의 → 계약 → 용역 → 유지보수
 (임정)   (윤소정) (임정)              (임정)    (임정+옥수주조) (설계→개발→테스트→인수) (공동)
```

### 팀 구성
- **임정**: 설계/검수 담당
- **윤소정**: 개발 담당

### Go/No-Go 판단 기준
- 자동응답 정확도
- 응답시간 단축률
- 클라이언트 만족도
- 운영 안정성

---

## 6. 데이터베이스 스키마 (Supabase)

| 테이블 | 용도 | Phase |
|--------|------|-------|
| `faq_templates` | FAQ 답변 템플릿 (카테고리, 패턴, 답변 본문) | P1 |
| `inquiries` | 문의/답변 이력 (AI 분류 결과 포함) | P1 |
| `products` | 상품 마스터 (channelProductNo, 주류정보) | P2 |
| `orders` | 주문/출고 이력 | P3 |
| `audit_log` | 워크플로우 실행 로그 | 공통 |

---

## 7. 산출물 목록

| 파일 | 설명 | 용도 |
|------|------|------|
| `README.md` | 테크니컬 리포트 (본 문서) | 프로젝트 전체 개요 |
| `usecase-overview.html` | 유즈케이스 요약 장표 (1장) | PPT 삽입용 |
| `appendix-architecture.html` | 아키텍처 + API 상세 (3장) | PPT 삽입용 |
| `process-diagram.html` | 업무 진행 절차 다이어그램 (2장) | PPT 삽입용 |
| `api-reference.md` | 커머스 API 레퍼런스 + UC 시나리오 예시 | 개발 참조용 |

> 모든 HTML 장표는 **흰색 배경**, **1280x720px** 기준으로 제작되어 PPT에 바로 삽입 가능합니다.

---

## 8. 참고 링크

- [네이버 커머스 API 센터](https://apicenter.commerce.naver.com)
- [커머스 API 공식 문서](https://apicenter.commerce.naver.com/docs/commerce-api/current)
- [커머스 API GitHub (기술지원)](https://github.com/commerce-api-naver/commerce-api)
- [인증 문서](https://apicenter.commerce.naver.com/docs/auth)
- [제약사항](https://apicenter.commerce.naver.com/docs/restriction)
