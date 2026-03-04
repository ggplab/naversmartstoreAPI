# 네이버 커머스 API 레퍼런스 — 옥수주조 자동화용

> 출처: [커머스API 공식 문서](https://apicenter.commerce.naver.com/docs/commerce-api/current) (v2.71.0, 2026-01-20)
> Base URL: `https://api.commerce.naver.com`

---

## 1. API 전체 카테고리

| 카테고리 | 설명 | 우리 활용 Phase |
|----------|------|----------------|
| **인증** | OAuth 2.0 토큰 발급/갱신 | 공통 |
| **문의** | 고객문의/상품문의 조회·답변 | P1 |
| **상품** | 상품 등록/수정/삭제, 카테고리 | P2 |
| **주문** | 주문 조회, 발주/발송, 클레임(교환/반품/취소) | P3 |
| **정산** | 건별/일별 정산 내역 조회 | 참고용 |
| **커머스솔루션** | 비즈월렛 등 | 미사용 |
| **판매자정보** | 판매자 기본정보 | 참고용 |
| **API데이터솔루션** | 통계/분석 | 참고용 |

---

## 2. P1 — 고객문의 자동응답 API

### 2-1. 고객 문의 조회

```
GET /v1/pay-user/inquiries
```

- 고객이 네이버페이 주문 관련으로 남긴 문의 목록 조회
- 미답변 문의 필터링에 사용

### 2-2. 상품 문의 목록 조회

```
GET /v1/contents/qnas
```

- 상품 상세페이지에 남긴 Q&A 목록 조회
- 고객 문의와 별도 — 상품 자체에 대한 질문

### 2-3. 상품 문의 답변 템플릿 목록 조회

```
GET /v1/contents/qnas/templates
```

- 판매자가 미리 등록해둔 답변 템플릿 조회
- FAQ 자동응답 시 템플릿 매칭에 활용 가능

### 2-4. 고객 문의 답변 등록

```
POST /v1/pay-merchant/inquiries/:inquiryNo/answer
```

- 특정 문의에 대한 답변 최초 등록
- `inquiryNo`: 문의 번호 (조회 API에서 획득)

**에러 코드:**
| 코드 | 설명 |
|------|------|
| ERR-NC-101001 | 정상적인 요청이 아닌 경우 |
| ERR-NC-101004 | 문의 번호에 해당하는 문의가 없거나 삭제된 경우 |
| ERR-NC-101005 | 문의가 유효하지 않은 경우 |
| ERR-NC-101007 | 답변 권한이 없는 경우 |
| ERR-NC-101008 | 판매자 번호가 유효하지 않은 경우 |
| ERR-NC-101010 | 해당 문의에 이미 답변이 존재하는 경우 |

### 2-5. 고객 문의 답변 수정

```
PUT /v1/pay-merchant/inquiries/:inquiryNo/answer/:answerContentId
```

- 기존 답변 수정 (오답변 보정용)
- `answerContentId`: 답변 등록 시 반환된 ID

### 2-6. 상품 문의 답변 등록/수정

```
PUT /v1/contents/qnas/:questionId
```

- 상품 Q&A에 대한 답변 등록 또는 수정

---

## 3. P2 — 시즌상품 등록/관리 API

### 3-1. 카테고리 목록 조회

```
GET /v1/categories
```

- 전체 카테고리 트리 조회
- 상품 등록 시 필요한 `leafCategoryId` 획득

### 3-2. 카테고리 상세 조회

```
GET /v1/categories/:categoryId
```

- 특정 카테고리의 속성, 필수 입력 항목 확인
- 주류 카테고리의 법적 표기 필수항목 확인에 사용

### 3-3. 상품 등록 (v2)

```
POST /v2/products
```

- 신규 상품 등록 (원상품 + 채널상품 동시 생성)
- 주류 필수 항목: 주류종류, 알코올도수, 용량, 제조원, 원산지, 유통기한

### 3-4. 채널상품 조회 (v2)

```
GET /v2/products/channel-products/:channelProductNo
```

- 등록된 채널상품 상세 정보 조회

### 3-5. 채널상품 수정 (v2)

```
PUT /v2/products/channel-products/:channelProductNo
```

- 가격, 재고, 상세 설명 등 채널상품 정보 수정

### 3-6. 원상품 수정 (v2)

```
PUT /v2/products/origin-products/:originProductNo
```

- 원상품 레벨의 정보 수정

### 3-7. 상품 상태 변경

```
PUT /v1/products/origin-products/:originProductNo/change-status
```

- 판매중/판매중지/품절 등 상태 전환
- 재고 소진 시 자동 판매중지에 활용

---

## 4. P3 — 주문→발송 자동처리 API

### 4-1. 변경 상품주문 내역 조회

```
GET /v1/pay-order/seller/product-orders/last-changed-statuses
```

- 상태가 변경된 주문 목록 조회 (폴링용)
- `PAYED` 상태 필터로 신규 결제완료 주문 수집

### 4-2. 상품주문 목록 조회

```
GET /v1/pay-order/seller/product-orders
```

- 조건 기반 상품주문 목록 조회

### 4-3. 상품주문 상세 조회

```
POST /v1/pay-order/seller/product-orders/query
```

- 여러 상품주문 ID로 상세 정보 일괄 조회

### 4-4. 주문 상품주문 ID 조회

```
GET /v1/pay-order/seller/orders/:orderId/product-order-ids
```

- 주문번호로 하위 상품주문 ID 목록 조회

### 4-5. 발주 확인

```
POST /v1/pay-order/seller/product-orders/confirm
```

- 결제완료 주문에 대한 발주 확인 처리
- 이후 발송처리가 가능한 상태로 전환

### 4-6. 발송 처리 (송장 등록)

```
POST /v1/pay-order/seller/product-orders/dispatch
```

- 택배사 코드 + 송장번호를 포함하여 발송 처리
- 고객에게 배송 시작 알림 발송됨

### 4-7. 배송 희망일 변경

```
POST (endpoint는 seller-change-hope-delivery-pay-order-seller)
```

- 배송 희망일 변경 처리

---

## 5. 클레임 처리 API (참고)

주문 카테고리 하위에 교환/반품/취소 API가 있음. P3 이후 필요 시 확장.

| 기능 | 엔드포인트 slug |
|------|----------------|
| 교환 수거 완료 | `seller-approve-collected-exchange-pay-order-seller` |
| 교환 재배송 | `seller-re-delivery-exchange-pay-order-seller` |
| 교환 보류/해제 | `seller-holdback-exchange-pay-order-seller` |
| 교환 거부 | `seller-reject-exchange-pay-order-seller` |
| 반품 승인 | `seller-approve-return-pay-order-seller` |
| 반품 보류/해제 | `seller-holdback-return-pay-order-seller` |
| 반품 거부 | `seller-reject-return-pay-order-seller` |
| 반품 요청 | `seller-request-return-pay-order-seller` |
| 취소 승인 | `seller-approve-cancel-application-pay-order-seller` |
| 취소 요청 | `seller-request-cancel-pay-order-seller` |

---

## 6. 인증 (OAuth 2.0)

- 커머스API센터에서 애플리케이션 등록 → `client_id` / `client_secret` 발급
- Token 발급 후 API 호출 시 Bearer Token 사용
- Rate Limit: 내스토어 앱 기준 **~2 req/sec**
- 상세: [인증 문서](https://apicenter.commerce.naver.com/docs/auth)

---

## 7. 유즈케이스별 자동화 시나리오 예시

### UC-1. 고객문의 자동응답 (P1)

**시나리오: "유통기한이 언제까지인가요?" 문의 자동 처리**

```
1. [n8n Cron] 3분마다 실행
2. [GET /v1/pay-user/inquiries] 미답변 문의 목록 조회
3. [GET /v1/contents/qnas] 상품 Q&A 미답변 목록 조회
4. [Claude API] 문의 내용 → 의도 분류
   - Input: "매실막걸리 유통기한이 언제까지인가요?"
   - Output: { category: "유통기한", confidence: 0.94 }
5. [Supabase] faq_templates 테이블에서 "유통기한" 카테고리 답변 조회
   - "안녕하세요, 옥수주조입니다. 매실막걸리의 유통기한은 제조일로부터
     30일입니다. 냉장보관(2~8°C) 시 최상의 맛을 즐기실 수 있습니다."
6. [POST /v1/pay-merchant/inquiries/:inquiryNo/answer] 답변 자동 등록
7. [Supabase] inquiries 테이블에 처리 이력 로깅
8. confidence < 0.85인 경우 → Slack 알림으로 담당자에게 에스컬레이션
```

**예상 FAQ 카테고리 (막걸리 특화):**
| 카테고리 | 예시 질문 | 예상 비율 |
|----------|----------|----------|
| 유통기한 | "유통기한이 언제까지?" | ~25% |
| 보관방법 | "냉장보관 해야하나요?" | ~20% |
| 도수/성분 | "도수가 어떻게 되나요?" | ~15% |
| 배송 | "제주도도 배송되나요?" | ~15% |
| 선물포장 | "선물세트 포장 가능한가요?" | ~10% |
| 기타 | 분류 불가 → 사람 처리 | ~15% |

---

### UC-2. 시즌상품 자동 등록 (P2)

**시나리오: "여름 수박막걸리" 신제품 등록**

```
1. [Google Sheets] 담당자가 상품 정보 입력
   - 상품명: 옥수주조 수박막걸리
   - 가격: 8,900원
   - 도수: 6%
   - 용량: 750ml
   - 원산지: 국내산
2. [n8n Webhook] Sheets 변경 감지 → 워크플로우 트리거
3. [GET /v1/categories] "식품 > 전통주/막걸리" 카테고리 ID 조회
4. [n8n Function] 법적 표기사항 자동 매핑
   - 주류종류: 탁주
   - 알코올도수: 6%
   - 내용량: 750ml
   - 제조원: 옥수주조
   - 원산지: 국내산
5. [POST /v2/products] 상품 등록 API 호출
6. [Supabase] products 테이블에 channelProductNo 저장
7. [Slack] "수박막걸리 등록 완료 (채널상품번호: xxxxx)" 알림

--- 재고 소진 시 ---
8. [Cron] 재고 모니터링 (외부 재고 시스템 연동 또는 수동 트리거)
9. [PUT /v1/products/origin-products/:id/change-status] 판매중지로 전환
```

---

### UC-3. 주문→발송 자동처리 (P3)

**시나리오: 신규 주문 접수 → 당일 출고**

```
1. [n8n Cron] 5분마다 실행
2. [GET /v1/pay-order/seller/product-orders/last-changed-statuses]
   - lastChangedType: PAYED (결제완료)
3. [POST /v1/pay-order/seller/product-orders/query]
   - 주문 상세 정보 조회 (상품명, 수량, 배송지)
4. [POST /v1/pay-order/seller/product-orders/confirm]
   - 발주 확인 처리
5. [택배사 API] 송장번호 발급 요청
   - CJ대한통운 POST /shipments
   - → trackingNumber: "1234567890"
6. [POST /v1/pay-order/seller/product-orders/dispatch]
   - deliveryCompanyCode: "CJGLS"
   - trackingNumber: "1234567890"
7. [Supabase] orders 테이블에 주문/출고 이력 로깅
8. [Slack] "주문 3건 출고 완료" 알림

--- 에러 처리 ---
- API 실패 시: Supabase retry_queue에 저장, 다음 Cron에서 재시도
- Rate Limit 초과 시: exponential backoff (2초 → 4초 → 8초)
```

---

### UC-추가. 활용 가능한 추가 유즈케이스 아이디어

| # | 유즈케이스 | 활용 API | 난이도 |
|---|-----------|----------|--------|
| A | **정산 리포트 자동화** — 일별/건별 정산 내역을 Sheets에 자동 기록 | `GET /v1/pay-settle/settle/daily` | 낮음 |
| B | **재고 알림** — 특정 상품 재고가 임계치 이하일 때 Slack 알림 | 채널상품 조회 + Slack | 낮음 |
| C | **클레임 자동 분류** — 교환/반품 요청을 AI로 분류하고 처리 가이드 제공 | 교환/반품 API + Claude | 중간 |
| D | **상품 문의 답변 템플릿 관리** — 자주 묻는 질문을 분석해 템플릿 자동 업데이트 | `GET /v1/contents/qnas/templates` | 중간 |
| E | **주문 통계 대시보드** — 일별 주문량, 매출 추이를 자동 시각화 | 주문조회 + Supabase + Sheets | 중간 |

---

## 8. API 문서 링크 모음

| 카테고리 | URL |
|----------|-----|
| 소개 | https://apicenter.commerce.naver.com/docs/introduction |
| 인증 | https://apicenter.commerce.naver.com/docs/auth |
| 제약사항 | https://apicenter.commerce.naver.com/docs/restriction |
| 문의 API | https://apicenter.commerce.naver.com/docs/commerce-api/current/고객-문의-답변-등록-수정 |
| 상품 API | https://apicenter.commerce.naver.com/docs/commerce-api/current/그룹상품 |
| 주문 API | https://apicenter.commerce.naver.com/docs/commerce-api/current/교환 |
| 정산 API | https://apicenter.commerce.naver.com/docs/commerce-api/current/부가세-내역 |
| GitHub Q&A | https://github.com/commerce-api-naver/commerce-api |
