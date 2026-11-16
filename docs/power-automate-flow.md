# Power Automate 쿠폰 발송 흐름 구성

게임(`index.html`)은 쿠폰이 결정되면 `CONFIG.MAIL_ENDPOINT` 로 JSON을 POST 합니다.
요청 본문 스키마: [`request-schema.json`](request-schema.json)

## 1. 기본 흐름 (현재 구성)

```
HTTP 요청을 받은 경우 (트리거할 수 있는 사람: 모든 사용자)
 └ 메일 보내기(V2)   받는 사람=to · 제목=subject · 본문=html
 └ 응답               200 · Access-Control-Allow-Origin: *  · {"ok": true}
```

- 같은 기기에서 같은 날짜로 다시 플레이하면 게임이 `resend: true` 와 함께 **기존 쿠폰**으로 재발송 메일을 만들어 보냅니다.

## 2. 권장: 이메일 × 행사일 1장 서버 검증 (다른 기기·시크릿 창 차단)

SharePoint 목록 **GSTARBugRoyale** (사이트: https://megazone0.sharepoint.com/sites/axjouney)

열: `Title`(참가자 이메일), `Day`(선택: d19/d20), `DayLabel`, `Coupon`, `Prize`, `Severity`, `FullName`, `Company`, `JobRole`, `Phone`, `Marketing`(예/아니요), `ResendCount`(숫자), `Redeemed`(예/아니요, 현장 사용 체크), `IssuedAt`(날짜)

보유 기간: 행사 종료 후 1년 이내 목록 삭제

```
HTTP 요청을 받은 경우
 └ 항목 가져오기 (GSTARBugRoyale)
     필터 쿼리: Title eq '@{triggerBody()?['to']}' and Day eq '@{triggerBody()?['day']}'
 └ 조건: length(outputs('Get_items')?['body/value'])  >  0
     ├ 예 (이미 당첨)
     │   └ 메일 보내기(V2)
     │       제목: [재발송] 이미 당첨되신 @{triggerBody()?['dayLabel']} 쿠폰을 재발송해드립니다
     │       본문: 쿠폰 = first(outputs('Get_items')?['body/value'])?['Coupon']
     │             상품 = first(outputs('Get_items')?['body/value'])?['Prize']
     │             일시/장소 = time, venue, address
     │   └ (선택) 항목 업데이트  ResendCount = add(first(outputs('Get_items')?['body/value'])?['ResendCount'],1)
     │   └ 응답 409 · Access-Control-Allow-Origin: *
     │       {"coupon":"@{first(outputs('Get_items')?['body/value'])?['Coupon']}",
     │        "prize":"@{first(outputs('Get_items')?['body/value'])?['Prize']}"}
     └ 아니요 (첫 발급)
         └ 항목 만들기 (GSTARBugRoyale)
             Title=to · Day Value=day · DayLabel=dayLabel · Coupon=coupon · Prize=prize · Severity=bug
             FullName=user.name · Company=user.company · JobRole=user.role · Phone=user.phone
             Marketing=user.marketing · ResendCount=0 · Redeemed=아니요 · IssuedAt=utcNow()
         └ 메일 보내기(V2)   to · subject · html
         └ 응답 200 · Access-Control-Allow-Origin: *
```

## 3. 개인 메일 차단 (선택, 서버 측 2차 방어)

조건 식:
```
contains('|gmail.com|naver.com|daum.net|hanmail.net|kakao.com|nate.com|hotmail.com|outlook.com|icloud.com|',
         concat('|', toLower(last(split(triggerBody()?['to'],'@'))), '|'))
```
`true` 이면 응답 400, 메일 미발송.

## 4. 행사 종료 후
11/20 행사 종료 후 흐름을 **끄기** 하세요. (엔드포인트 주소가 공개 페이지에 노출되어 있음)
