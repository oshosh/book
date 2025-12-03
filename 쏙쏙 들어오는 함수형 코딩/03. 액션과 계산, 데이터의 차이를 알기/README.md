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


  임시
  # PR Description

`es-toolkit/compat`'s `merge()` produces different results than `lodash-es/merge` when merging objects that contain `Date` values.

Given the reproduction from [#1547](https://github.com/toss/es-toolkit/issues/1547):

```
import { merge as esMerge } from 'es-toolkit/compat';
import { merge as ldMerge } from 'lodash-es';

const target = { a: 1, b: { x: 1, y: 2 }, c: new Date('2025-01-01') };
const source = { b: { y: 3, z: 4 }, c: new Date('2000-01-01') };

const resultEs = esMerge({}, target, source);
const resultLd = ldMerge({}, target, source);

console.log('es-toolkit/compat', resultEs);
// { a: 1, b: { x: 1, y: 3, z: 4 }, c: 2025-01-01T00:00:00.000Z } ❌

console.log('lodash-es', resultLd);
// { a: 1, b: { x: 1, y: 3, z: 4 }, c: 2000-01-01T00:00:00.000Z } ✅**Actual:** `merge` from `es-toolkit/compat` keeps the `Date` from the target.  
```
**Expected:** it should use the `Date` from the source, like `lodash-es`.


---

## Root Cause

The compat implementation of `merge` is based on `mergeWith`:
-  `src/compat/object/merge.ts`
```
export function merge(object: any, ...sources: any[]): any {
  return mergeWith(object, ...sources, noop);
}Inside `mergeWithDeep` (`mergeWith.ts`), when both `targetValue` and `sourceValue` are “object-like”, the code recursively deep‑merges them regardless of their concrete type:

const merged = merge(targetValue, sourceValue, key, target, source, stack);

if (merged !== undefined) {
  target[key] = merged;
} else if (Array.isArray(sourceValue)) {
  target[key] = mergeWithDeep(targetValue, sourceValue, merge, stack);
} else if (isObjectLike(targetValue) && isObjectLike(sourceValue)) {
  // <-- this branch also handles Date / RegExp / other non-plain objects ❌
  target[key] = mergeWithDeep(targetValue, sourceValue, merge, stack);
} else {
  // ...
}
```
As a result, when both values are `Date` instances, the logic performs a deep merge instead of simply replacing the value, and the original `Date` from the target is preserved.

---

## Solution

Restrict the “deep merge” branch so that it only applies to mergeable types (plain objects / typed arrays), and let other object-like values (such as `Date`) fall through to the simple assignment path.

- `src/compat/object/mergeWith.ts`
```
const merged = merge(targetValue, sourceValue, key, target, source, stack);

if (merged !== undefined) {
  target[key] = merged;
} else if (Array.isArray(sourceValue)) {
  target[key] = mergeWithDeep(targetValue, sourceValue, merge, stack);
} else if (
  isObjectLike(targetValue) &&
  isObjectLike(sourceValue) &&
  (
    isPlainObject(targetValue) ||
    isPlainObject(sourceValue) ||
    isTypedArray(targetValue) ||
    isTypedArray(sourceValue)
  )
) {
  // Only deep-merge “mergeable” object-like values
  target[key] = mergeWithDeep(targetValue, sourceValue, merge, stack);
} else if (targetValue == null && isPlainObject(sourceValue)) {
  // ...
}
```
With this change:
- `Date` vs `Date` no longer goes through the deep‑merge branch.
- The value from the source is assigned in the final `else if (targetValue === undefined || sourceValue !== undefined)` branch.
- Existing behavior for plain objects, arrays, and typed arrays is preserved.

---


## Tests

I added tests for both `merge` and `mergeWith` to cover the reported scenario.

- **`src/compat/object/merge.spec.ts`**
```
it('should overwrite Date values from source like lodash-es', () => {
  const target = { a: 1, b: { x: 1, y: 2 }, c: new Date('2025-01-01') };
  const source = { b: { y: 3, z: 4 }, c: new Date('2000-01-01') };

  const actual = merge({}, target, source);

  expect(actual).toEqual({
    a: 1,
    b: { x: 1, y: 3, z: 4 },
    c: new Date('2000-01-01'),
  });
});
```
- **`src/compat/object/mergeWith.spec.ts`**
```
it('should overwrite Date values from source when customizer is noop', () => {
  const target = { a: 1, b: { x: 1, y: 2 }, c: new Date('2025-01-01') };
  const source = { b: { y: 3, z: 4 }, c: new Date('2000-01-01') };

  const actual = mergeWith({}, target, source, noop);

  expect(actual).toEqual({
    a: 1,
    b: { x: 1, y: 3, z: 4 },
    c: new Date('2000-01-01'),
  });
});
```
All existing tests continue to pass.
---
