# android-inapp-billing-v3-guide
android-inapp-billing-v3-guide (korean translation)

이 문서는 [android inapp-billing-v3 라이브러리](https://github.com/anjlab/android-inapp-billing-v3)의 공식 문서를 한국어로 번역한 것입니다.

오역을 발견하신다면 Issue에 남겨 주세요.

직접 수정하셔서 Pull Request를 주셔도 좋습니다.

# Android In-App Billing v3 Library [![Build Status](https://travis-ci.org/anjlab/android-inapp-billing-v3.svg?branch=master)](https://travis-ci.org/anjlab/android-inapp-billing-v3)  [![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.anjlab.android.iab.v3/library/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.anjlab.android.iab.v3/library)

매우 간단명료한 Android v3 In-app billing API를 구현한 라이브러리입니다.

구독, 인앱 결제 상품에 대한 인앱 결제를 지원합니다.

## Getting Started

* 적어도 프로젝트가 Android 2.2 이상의 SDK를 사용해야 합니다 

* 이 라이브러리를 프로젝트에 추가하십시오
  - 만약 이클립스를 사용중이라면, [이 링크](https://github.com/anjlab/android-inapp-billing-v3/releases)의 releases섹션에서 최신 jar 버전을 다운받아 dependency에 추가하십시오
  - 만약 Android Studio+Gradle을 사용중이라면, build.gradle 파일에 다음을 추가하십시오
```groovy
repositories {
  mavenCentral()
}
dependencies {
  implementation 'com.anjlab.android.iab.v3:library:1.0.44'
}
```
* *AndroidManifest.xml* 파일을 열어 다음 권한을 추가하십시오
```xml
  <uses-permission android:name="com.android.vending.BILLING" />
```
* BillingProcessor 클래스의 인스턴스를 만들고 액티비티 소스코드에서 콜백을 구현하십시오.
생성자는 3개의 파라미터를 갖습니다.
  - **Context**
  - **개발자 콘솔에서 받은 라이선스 키** 이것은 결제 유효성을 검증하기 위해 사용됩니다. 검증 과정을 생략하려면 null을 전달할 수 있습니다.(라이선스 키는 개발자 콘솔-> 앱 네임-> 서비스&API 에서 확인할 수 있습니다)
  - **구매 결과와 에러를 처리하기 위해 IBillingHandler Interface를 구현하십시오** (아래를 참조)
```java
public class SomeActivity extends Activity implements BillingProcessor.IBillingHandler {
  BillingProcessor bp;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    bp = new BillingProcessor(this, "YOUR LICENSE KEY FROM GOOGLE PLAY CONSOLE HERE", this);
    bp.initialize();
    // or bp = BillingProcessor.newBillingProcessor(this, "YOUR LICENSE KEY FROM GOOGLE PLAY CONSOLE HERE", this);
    // See below on why this is a useful alternative
  }
	
  // IBillingHandler implementation
	
  @Override
  public void onBillingInitialized() {
    /*
    * Called when BillingProcessor was initialized and it's ready to purchase 
    */
  }
	
  @Override
  public void onProductPurchased(String productId, TransactionDetails details) {
    /*
    * Called when requested PRODUCT ID was successfully purchased
    */
  }
	
  @Override
  public void onBillingError(int errorCode, Throwable error) {
    /*
    * Called when some error occurred. See Constants class for more details
    * 
    * Note - this includes handling the case where the user canceled the buy dialog:
    * errorCode = Constants.BILLING_RESPONSE_RESULT_USER_CANCELED
    */
  }
	
  @Override
  public void onPurchaseHistoryRestored() {
    /*
    * Called when purchase history was restored and the list of all owned PRODUCT ID's 
    * was loaded from Google Play
    */
  }
}
```

* Activity의 onActivityResult를 오버라이드 합니다
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
  if (!bp.handleActivityResult(requestCode, resultCode, data)) {
    super.onActivityResult(requestCode, resultCode, data);
  }  
}
```

* 구매를 시작하기 위해 `purchase` 메소드를 호출하거나, 구독을 시작하기 위해 `subscribe`를 호출하십시오

_개발자가 지정한 임의의 페이로드 값이 필요 없을 경우:_
```java
bp.purchase(YOUR_ACTIVITY, "YOUR PRODUCT ID FROM GOOGLE PLAY CONSOLE HERE");
bp.subscribe(YOUR_ACTIVITY, "YOUR SUBSCRIPTION ID FROM GOOGLE PLAY CONSOLE HERE");
```
_개발자가 지정한 임의의 페이로드 값이 필요 할 경우:_
```java
bp.purchase(YOUR_ACTIVITY, "YOUR PRODUCT ID FROM GOOGLE PLAY CONSOLE HERE", "DEVELOPER PAYLOAD HERE");
bp.subscribe(YOUR_ACTIVITY, "YOUR SUBSCRIPTION ID FROM GOOGLE PLAY CONSOLE HERE", "DEVELOPER PAYLOAD HERE");
```
_중요: payload를 제공하여 purchase/subscribe를 호출했을 때, 내부적으로 라이브러리가 페이로드에 문자열을 추가합니다. 구독의 경우, `"subs:\<productId\>:"`가 추가되고, 구매의 경우 `"inapp:\<productId\>:\<UUID\>:"` 가 추가됩니다. 결제 성공 후 구글 플레이로부터 넘겨받은 페이로드를 검증하는지 알고 있는 것이 중요합니다._

_추가적인 파라미터를 포함하는 번들이 필요할 경우_

```java
Bundle extraParams = new Bundle()
extraParams.putString("accountId", "MY_ACCOUNT_ID");
bp.purchase(YOUR_ACTIVITY, "YOUR PRODUCT ID FROM GOOGLE PLAY CONSOLE HERE", null /*or developer payload*/, extraParams);
bp.subscribe(YOUR_ACTIVITY, "YOUR SUBSCRIPTION ID FROM GOOGLE PLAY CONSOLE HERE", null /*or developer payload*/, extraParams);
```

추가적인 파라미터를 전달하기 원한다면, 다음 메소드를 사용하여 번들 객체를 전달할 수 있습니다 [문서](https://developer.android.com/google/play/billing/billing_reference.html#getBuyIntentExtraParams), you can provide a Bundle object.

_이 특징은 오직 타겟 디바이스가 In App Billinmg API 7버전 이상을 지원할 때만 사용 가능하다는 것을 명심하십시오_

* **이제 끝입니다. 세상에 없던 엄청 작고 빠른 인앱 라이브러리입니다. **

* 그리고 **BillingProcessor 인스턴스를 릴리즈하는 것을 잊지 마십시오**
 
```java
@Override
public void onDestroy() {
  if (bp != null) {
    bp.release();
  }		
  super.onDestroy();
}
```

### 늦은 초기화로 BillingProcessor 초기화하기
`new BillingProcessor(...)`는 생성자 안에서 Play Service에 바인딩합니다. 이것은 아주 드물게 Play Serivce에 바인딩될 때 생성자보다 먼저 onBillingInitialized가 호출되어 race condition을 일으키며, NullPointerException을 발생시킵니다. 이를 피하기 위해, 다음을 따르십시오:
```java
bp = BillingProcessor.newBillingProcessor(this, "YOUR LICENSE KEY FROM GOOGLE PLAY CONSOLE HERE", this); // doesn't bind
bp.initialize(); // binds
```

## 인앱 결제 테스트

여기에 완벽한 [가이드](https://developer.android.com/google/play/billing/billing_testing.html)가 있습니다.
시작하기 전에 반드시 가이드를 읽으십시오.

## 플레이스토어 서비스가 사용 가능한지 확인

이 라이브러리를 사용하기 전에 인앱 결제 서비스가 사용 가능한지 확인해야 합니다.
오래된 디바이스나 중국에서는 플레이스토어가 사용 불가능하거나 곧 제거될 예정이라 인앱 결제를 지원하지 않습니다.

다음 메소드를 사용하여 확인하십시오 `BillingProcessor.isIabServiceAvailable()`:
```java
boolean isAvailable = BillingProcessor.isIabServiceAvailable();
if(!isAvailable) {
  // continue
}
```
`BillingProcessor.isIabServiceAvailable()`가 true라고 해서 인앱 결제가 100%가능하다는 것은 아닙니다.
해당 메소드는 플레이스토어 앱이 설치되었는지만 확인하기 때문에 다른 이유로 결제가 실패할 수도 있습니다.

그러므로 BillingProcessor가 초기화된 후 `isOneTimePurchaseSupported()`를 호출하는 것이 좋습니다:
```java
boolean isOneTimePurchaseSupported = billingProcessor.isOneTimePurchaseSupported();
if(isOneTimePurchaseSupported) {
  // launch payment flow
}
```
또는 구독 갱신 사례를 업데이트하기 위해 `isSubscriptionUpdateSupported()` 를 호출합니다:
```java
boolean isSubsUpdateSupported = billingProcessor.isSubscriptionUpdateSupported();
if(isSubsUpdateSupported) {
  // launch payment flow
}
```

## 구매한 상품 소비

구매한 상품을 사용(소비)처리할 수도 있고, 같은 상품을 여러번 구매하게 할 수도 있습니다:
```java
bp.consumePurchase("YOUR PRODUCT ID FROM GOOGLE PLAY CONSOLE HERE");
```

## 구매/구독내역 복원

```java
bp.loadOwnedPurchasesFromGoogle();
```

## 상품 리스트 조회

Google Play에 등록한 상품/구독에 대한 가격/설명 정보를 보려면 다음 메소드를 사용하십시오.

```java
bp.getPurchaseListingDetails("YOUR PRODUCT ID FROM GOOGLE PLAY CONSOLE HERE");
bp.getSubscriptionListingDetails("YOUR SUBSCRIPTION ID FROM GOOGLE PLAY CONSOLE HERE");
```

위 메소드 실행 결과로 아래의 데이터를 포함하는 `SkuDetails`객체를 얻을 수 있습니다:
```java
public final String productId;
public final String title;
public final String description;
public final boolean isSubscription;
public final String currency;
public final Double priceValue;
public final String priceText;
```

여러 개의 상품/구독에 대한 정보를 얻으려면, id를 리스트로 전달하십시오:
```java
bp.getPurchaseListingDetails(arrayListOfProductIds);
bp.getSubscriptionListingDetails(arrayListOfProductIds);
```

arrayListOfProductIds의 타입은 `ArrayList<String>`이며, 대상 상품/구독 제품에 대한 id를 포함합니다.

위 메소드 실행 결과로 `List<SkuDetails>`타입의 데이터를 얻게 됩니다. SkuDetails 객체가 포함하는 데이터는 위에서 묘사한 것과 같습니다.

## 결제 정보 상세
1.0.9버전의 변경 사항으로 Transaction 객체가 핸들러 클래스의 onProductPurchased 메소드로 전달됩니다. 그러나, 구매 시점이 아니더라도 아래의 메소드를 이용하여 결제 정보에 대한 상세 정보를 얻을 수 있습니다.
```java
bp.getPurchaseTransactionDetails("YOUR PRODUCT ID FROM GOOGLE PLAY CONSOLE HERE");
bp.getSubscriptionTransactionDetails("YOUR SUBSCRIPTION ID FROM GOOGLE PLAY CONSOLE HERE");
```

위 메소드를 호출하여 얻은 데이터는 `TransactionDetails` 객체이며 아래의 데이터를 포함합니다.
```java
public final String productId;
public final String orderId;
public final String purchaseToken;
public final Date purchaseTime;
    
// containing the raw json string from google play and the signature to
// verify the purchase on your own server
public final PurchaseInfo purchaseInfo;
```

## 구매 이력 보기
가장 최근의 구매 이력은 `getPurchaseHistory`를 이용하여 요청할 수 있습니다. 

type 매개변수에 일회성 구매 상품의 경우 "inapp" 또는 `Constants.PRODUCT_TYPE_MANAGED`를 전달하고,
구독 상품의 경우 "subs"또는 `Constants.PRODUCT_TYPE_SUBSCRIPTION`을 전달하십시오.
```java
public List<BillingHistoryRecord> getPurchaseHistory(String type, Bundle extraParams)
```
호출 결과로써 `List<BillingHistoryRecord>`를 얻습니다.
BillingHistoryRecord는 다음 데이터를 포함합니다.
```java
public final String productId;
public final String purchaseToken;
public final long purchaseTime;
public final String developerPayload;
public final String signature;
```
이 API는 Billing API 버전 6 이상이 필요하다는 점을 명심하십시오. 다음 메소드를 이용하여 지원 여부를 확인하고 사용해야 합니다:
```java
public boolean isRequestBillingHistorySupported(String type)
```

## 취소된 구독 처리

`bp.getSubscriptionTransactionDetails(...)`를 호출하고 `purchaseInfo.purchaseData.autoRenewing` 값을 확인하십시오.
만약 구독이 취소되었을 경우 값은 `False`가 됩니다.

물론 이 값을 확인하기 이전에 `bp.loadOwnedPurchasesFromGoogle()`를 이용해 구독 정보를 업데이트해야 합니다.

## 프로모션 코드 지원

프로모션 코드는 구매 다이얼로그나 구글 플레이 앱 내에 입력될 수 있습니다. 다음 URL을 사용하면(https://play.google.com/redeem?code=YOUR_PROMO_CODE) 프로모션 코드가 입력된 채로 앱이 시작됩니다. 이는 사용자에게 앱 내에 프로모션 코드를 입력 하는 옵션을 제공하려는 경우 유용합니다.

## 가짜 마켓으로부터 보호

구글 플레이 마켓의 취약점을 활용하는 공격이 많습니다. 그 중 Freedom Attack은 특정 한드로이드 앱의 Play Market Service로의 호출을 탈취하여 가짜로 바꿔치기합니다. 결국 당신의 앱은 마켓으로부터 결제에 대해 유효한 응답을 받았다고 생각하게 됩니다.

이러한 종류의 공격으로부터 보호하기 위해, `merchantId`라는 값을 특정해야 합니다.
`merchantId`는 [이곳](https://payments.google.com/merchant)에서 *설정->공개 프로필*에서 고유한 `merchantId` 값을 찾을 수 있습니다.

**WARNING:** `merchantId` 값을 안전한 곳에 보관하십시오.

    그리고 `merchantId`를 사용하여 생성자를 호출하십시오:

    public BillingProcessor(Context context, String licenseKey, String merchantId, IBillingHandler handler);

    추후 다음 메소드를 활용하여 결제 유효성을 쉽게 검사할 수 있습니:

    public boolean isValid(TransactionDetails transactionDetails);

P.S. 이러한 종류의 방어는 오직 2012년 12월~ 2015년 7월 사이에 이루어진 결제에 대해서만 유효합니다. 2012년 12월 이전 결제 상세에서는 `orderId`가 포함되지 않고, 2015년 7월 이후에는 `orderId`의 포맷이 변경되었습니다.
 
## Proguard

필수적인 Proguard 설정은 이 라이브러리에 이미 추가되었습니다. 추가적인 설정은 필요하지 않습니다.

consumer proguard file 내에 포함된 정보는 다음과 같습니다:
```
-keep class com.android.vending.billing.**
```

[IABv3 문서](https://developer.android.com/google/play/billing/billing_best_practices.html#validating-purchase-device)


## License

Copyright 2014 AnjLab

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. **Create New Pull Request**
