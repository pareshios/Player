# RevenueCat

## Introduction

In simple, [revenueCat](https://www.revenuecat.com/docs/welcome) is a wrapper around Appstore/PlayStore subscription integration flows. Everything you need to implement and manage in-app purchases and subscriptions

## Configure Subscriptions in App Store Connect [&#x270D;](https://appstoreconnect.apple.com/apps/1572235571/appstore/subscriptions)

Note: Add tax and billing information in the app store connect before configuring products otherwise you will not be able to get any product list from the store API. As revenueCat will throw an error while fetching the products list using their SDK

We need to configure our subscription | In-App Purchases in App store connect. There are mainly four types of subscriptions available.

* In-App Purchase
    * Consumable
    * Non-Consumable
* Subscription
    * Auto renewable
    * No Renewable
    
All other subscriptions except Auto Renewable can be configured individually, But Auto renewable must have a subscription group. Under a subscription group, One can add multiple subscriptions and the user can subscribe to one of them.
For Auto-renewable subscription localisation and grace period, a setting is required.

Every product we configure must have an identifier that is unique across subscriptions. which can be used to make any transaction with Storekit.

Configure the introductory, Promotional, and offer code with a subscription if you need to give the customer a free trial or any discount on a subscription.

1. Pay-up-front — The customer pays once for a period of time, e.g. $0.99 for 3 months. Allowed durations are 1, 2, 3, 6 and 12 months.
2. Pay-as-you-go — The customer pays a reduced rate, each period, for a number of periods, e.g. $0.99 per month for 3 months. Allowed durations are 1-12 months. Can only be specified in months.
3. Free — This is analogous to a free trial, the user receives 1 of a specified period free. The allowed durations are 3 days, 1 week, 2 weeks, 1 month, 2 months, 3 months, 6 months, and 1 year.

** [Offer code] (https://www.revenuecat.com/docs/ios-subscription-offers#offer-codes)

**Note:** - Since launch, Apple's in-app Offer Code redemption sheet has proven to be extremely unstable. For example, the sheet may not connect, may not dismiss after a successful redemption, and may not accept valid codes. Additionally, sandbox and TestFlight behavior has been seen to be inconsistent.
A workaround may be to instead redirect customers to the App Store app to redeem codes as described below.

## Create revenueCat account [&#x270D;](https://www.revenuecat.com/docs/projects)
RevenueCat provides a free trial for up to $10000 in revenue then you need to pay for the services. Create one revenueCat app and configure the platform-specific application for e.g Android and iOS. Application name, bundle identifier, App Store Connect App-Specific Shared Secret and key information need to add in order to complete the application configuration.

Redirect apple subscription event notification directly to revenueCat [configure](https://www.revenuecat.com/docs/server-notifications). This configuration will speed up the event update in revenueCat and the status will be updated fast.

## Configure Products [&#x270D;](https://www.revenuecat.com/docs/entitlements)
RevenueCat has a mapping model to display the products to the customer in the form of offerings. Entitlement, Offering, and Packages in the revenueCat dashboard are the way to manage your display and package group.
* Entitlement
  - An entitlement represents a level of access, features, or content that a user is "entitled" to.
* Offerings
  - Offerings are the selection of products that are "offered" to a user on your paywall. They're an optional, but recommended feature of RevenueCat that can make it easier to change and experiment with your paywalls.
* Packages
  - Each Offering you create should contain at least one Package that holds cross-platform products.
  
### Configure SDK in Platform [&#x270D;](https://www.revenuecat.com/docs/installation)
You can configure SDK for iOS, Android, and react-native. follow the steps in the above link.

### Displaying Products[&#x270D;](https://www.revenuecat.com/docs/displaying-products)
If you've [configured Offerings](https://www.revenuecat.com/docs/entitlements) in RevenueCat, you can control which products are shown to users without requiring an app update. Building paywalls that are dynamic and can react to different product configurations gives you maximum flexibility to make remote updates.

### Making Purchases
Process a transaction with Apple or Google

The SDK has a simple method, purchase(package:), that takes a package from the fetched Offering and purchases the underlying product with Apple, Google, or Amazon.

Purchase without promotional offer
```swift
 Purchases.shared.purchase(package: package) { (transaction, customerInfo, error, userCancelled) in
  if customerInfo.entitlements["your_entitlement_id"]?.isActive == true {
    // Unlock that great "pro" content              
  }
}
```

Purchase with Promotional offer
``` swift
func purchaseProductWith(package: Package, promoOffer: PromotionalOffer, completion: @escaping (Result<Bool, IAPError>) -> Void) {
        Purchases.shared.purchase(package: package, promotionalOffer: promoOffer) {[weak self] transaction, customerInfo, error, userCancelled in
   }
 }
```

### Checking Subscription Status
Get User Information
```swift
func getCustomerInfo(completion: @escaping (Result<Bool, IAPError>) -> Void) {
        Purchases.shared.getCustomerInfo { [weak self] (customerInfo, error) in
        log(customerInfo)
   }
}
```

### Get Entitlement Information
Get entitlement information in the customer info object as shown in the above function. There could be more than one active entitlement. Get isSandbox, willRenew, startDate and eneDate of subscription, Unsubscribe DetectedAt Date, and Billing issue date if any.

Unsubscribe detected and billing issue date will be there if any billing issue, refund initialisation, cancellation are in process and apple detected grace period.

### Checking If A User Is Subscribed
```swift
if customerInfo.entitlements["your_entitlement_id"]?.isActive == true {
  // user has access to "your_entitlement_id"                
}
```

```swift
if !customerInfo.entitlements.active.isEmpty {
    //user has access to some entitlement
}
```

### Listening For CustomerInfo Updates
This delegate method will be called every time we launch an application and when a purchase transaction is done. You can update subscription information when this updates the state of purchases.

```swift
// Additional configure setup
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
  
    Purchases.logLevel = .debug
    Purchases.configure(withAPIKey: "public_sdk_key")
    Purchases.shared.delegate = self // make sure to set this after calling configure
}

extension AppDelegate: PurchasesDelegate {
    func purchases(_ purchases: Purchases, receivedUpdated customerInfo: Purchases.CustomerInfo) {
        // handle any changes to customerInfo
    }
}
```

### Custom Attributes
RevenueCat support custom attribute which becomes very usefull in some scenario to mark a customer or identifing a customer specific tags.

```swift
Purchases.shared.attribution.setAttributes(["isRetainedUser": "True"])
```

## Manage Purchase after sale.
### Refund [&#x270D;](https://www.revenuecat.com/docs/refunds)
RevenueCat can handle refunds across all platforms for both subscription and non-subscription products. As soon as RevenueCat detects a refund, the CustomerInfo will be updated to reflect the correct entitlement status - no action required on your part! If you have questions about refunds, take a look at [our community](https://community.revenuecat.com/general-questions-7/how-do-i-issue-a-refund-115) article covering the topic.

### Upgrades, Downgrades, & Management [&#x270D;](https://www.revenuecat.com/docs/managing-subscriptions)
App Store

There are no code changes required to support upgrades, downgrades, and crossgrades for iOS subscriptions in your app. A customer can choose to upgrade, downgrade, or crossgrade between subscriptions as often as they like.

According to Apple, when a customer changes their subscription level, access to the new product can vary depending on the change:

Upgrade. A user purchases a subscription that offers a higher level of service than their current subscription. They are immediately upgraded and receive a refund of the prorated amount of their original subscription. If you’d like users to immediately access more content or features, rank the subscription higher to make it an upgrade.
Downgrade. A user selects a subscription that offers a lower level of service than their current subscription. The subscription continues until the next renewal date, then is renewed at the lower level and price.
Crossgrade. A user switches to a new subscription of the equivalent level. If the subscriptions are the same duration, the new subscription begins immediately. If the durations are different, the new subscription goes into effect at the next renewal date.
You can refer to this [blog post](https://www-origin.revenuecat.com/blog/engineering/ios-subscription-groups-explained/) for more information on how to set up subscription groups in App Store Connect.

### Price Change [&#x270D;](https://www.revenuecat.com/docs/price-changes)
Price change is trated by apple in certain ways please read this apple documentation for [more detail](https://developer.apple.com/app-store/subscriptions/#managing-prices-for-existing-subscribers)

When a subscription price increases apple will send you notifications, mail, and in-app messages to update users about the price increase If the price change is a big amount apple may present a confirmation sheet to get users to opt-in for the new price. A decrease in price may silently renew without the user’s knowledge.

### Cross-Platform Subscription Status
Multiple platforms may require to fetch the current subscription purchased by the user to entitle him/her to some access to the application. We can configure RevenueCat with user-id if the user is known in advance. When going from an Anonymous ID to a custom App User ID RevenueCat will decide whether the identities should be merged (aliased) into the same CustomerInfo object or not depending on the state of the custom App User ID and if it already has an anonymous alias.

For those applications which require authentication should initialise the revenueCat with nil user-id which will create an Anonymous user when they first launch the application. 

```swift
Purchases.configure(withAPIKey: "my_api_key")
```

Later you can log in to the revenueCat with user-id when the user authenticates successfully.

```swift
func loginRevenueCat(my_app_user_id: String) {
        Purchases.shared.logIn(my_app_user_id) { (customerInfo, created, error) in
            // customerInfo updated for my_app_user_id
            // Check customer info
        }
    }
```

When an identified user logs out you should call the logOut() method. This generates a new Anonymous App User ID for the logged-out state.
```swift
func logoutRevenueCat() {
    Purchases.shared.logOut { info, error in

    }
}
```
