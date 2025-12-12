# 액션과 계산, 데이터는 어디에나 적용할 수 있다
  - 장보기 과정을 예를 들면 아래는 전부 액션에 해당된다.
    `냉장고 확인하기 -> 운전해서 상점으로 가기 -> 필요한 것 구입하기 -> 운전해서 집으로 오기`
  - 냉장고 확인하기
    - 냉장고가 가진 제품은 **데이터**
      - 현재 재고 => **데이터**
    - 운전해서 상점으로 가기, 운전해서 집으로 오기
      - 여기도 데이터가 숨겨져 있으나 현재는 사용하지 않음
    - 필요한 것 구입하기
      - 무엇이 필요한지 정확히 알아야함

      | 종류  | 단계            |
      | --- | ------------- |
      | 데이터 | 현재 재고         |
      | 데이터 | 필요한 재고        |
      | 계산  | 재고 “빼기”       |
      | 데이터 | 장보기 목록        |
      | 액션  | 목록에 있는 것 구입하기 |

# 위 과정을 통해 배운 것
  - 액션과 계산, 데이터는 어디에나 적용 가능하다
  - 액션 안에는 계산, 데이터, 또 다른 액션이 숨어 있다.
  - 계산은 더 작은 계산과 데이터로 나누고 연결할 수 있다.
  - 데이터는 데이터만 조합할 수 있다.

# 쿠폰 보내는 과정 구현하기
  ```
  // 데이터베이스에서 가져온 구독자 데이터
  var subscriber = {
    email: 'sample@gmail.com',
    rec_count: 16
  }

  // 쿠폰 등급은 문자열
  var rank1 = 'best';
  var rank2 = 'good';

  // 쿠폰 등금을 결정하는 것은 함수
  function subCouponRank(subscriber) {
    if(subscriber.rec_count >= 10) {
      return = 'best'
    }else{
      return = 'good'
    }
  }

  // 데이터베이스에서 가져온 쿠폰 데이터
  var coupon = {
    code: '10PERCENT',
    rank: 'bad',
  }

  // 특정 등급의 쿠폰 목록을 선택하는 계산은 함수
  function selectCouponsByRank(coupons, rank) {
    var ret = []
    for(var c = 0; c < coupons.length; c++) {
      var coupon = coupons[c]
      if(coupon.rank === rank) {
        ret.push(coupon.code)
      }
      return ret;
    }
  }

  // 구독자가 받을 이메일을 계획하는 계산
  function emailForSubscriber(subscriber, goods, bests) {
    var rank = subCouponRank(subscriber);
    if(rank === "best") {
      return {
        from: "newsletter@coupongdg.co",
        to: subscriber.email,
        subject: "Your best weekly coupons inside",
        body: "Here are the best coupons: " + bests.join(", ")
      };
    } else // rank === "good"
      return {
        from: "newsletter@coupongdg.co",
        to: subscriber.email,
        subject: "Your good weekly coupons inside",
        body: "Here are the good coupons: " + goods.join(", ")
      };
  }

  // 보낸 이메일 목록을 준비하기
  function emailsForSubscribers(subscribers, goods, bests) {
    var emails = [];
    for(var s = 0; s < subscribers.length; s++) {
      var subscriber = subscribers[s];
      var email = emailForSubscriber(subscriber, goods, bests);
      emails.push(email);
    }
    return emails;
  }

  // 이메일 보내기를 위한 액션
  function sendIssue() {
    var coupons = fetchCouponsFromDB();
    var goodCoupons = selectCouponsByRank(coupons, "good");
    var bestCoupons = selectCouponsByRank(coupons, "best");
    var subscribers = fetchSubscribersFromDB();
    var emails = emailsForSubscribers(subscribers, goodCoupons, bestCoupons);
    for(var e = 0; e < emails.length; e++) {
      var email = emails[e];
      emailSystem.send(email);
    }
  }
  ```

# 이미 있는 코드에 함수형 사고 적용하기
  - 아래 코드는 액션에 액션을 타고 올라가는 코드이다.
    - 가능한 액션은 적게 사용
    - 가능한 작게 만든다.
    - 외부 세계와 상호작용하는 것을 제한
    - 호출 시점에 의존하는 것을 제한
```
type Affiliate = {
	sales: number;
	commission: number;
	bank_code: string;
}
function figurePayout(affiliate: Affiliate) {
	const owed = affiliate.sales * affiliate.commission;
	if (owed > 100) {
	// 하나의 액션이 있다고 생각할 수 있다.
	 sendPayout(affiliate.bank_code, owed);
	}
}

// 액션에서 액션을 호출 ...
function affiliatePayout(affiliates: Array<Affiliate>) {
	affiliates.forEach((affiliate) => {
		figurePayout(affiliate);
	})
 }
 
// 액션 호출...
function main(affiliates: Array<Affiliate>) {
	affiliatePayout(affiliates);
}
```