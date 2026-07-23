# Event Tracking & Attribution SDKs in Flutter (Facebook, TikTok, and Friends)

*What packages like the Facebook and TikTok SDKs actually do in a Flutter app — and why growth teams care so much about them.*

---

If you work on a consumer app, sooner or later someone from marketing asks: "Can you add the Facebook SDK? And TikTok? And AppsFlyer?" These are **event tracking and attribution SDKs**. They don't build features users see — they answer two business questions: *"What are users doing?"* and *"Which ad brought them here?"* Let me explain what they do and how they fit into Flutter.

## Two jobs: events and attribution

- **Event tracking** — record what the user does: signed up, started a trial, made a purchase, reached level 5. Each of these is an "event".
- **Attribution** — connect an install or a purchase back to the ad that caused it. If a user tapped a TikTok ad, installed the app, and subscribed, attribution tells the marketing team *that TikTok ad made money*.

Ad platforms (Meta, TikTok, Google) use these events to **optimize your ads**. When you send them "this user purchased", their algorithm learns to show your ad to more people like that user. So these SDKs are the loop that makes paid growth work.

![Event tracking vs attribution — two jobs](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/event-tracking/fig-1.png)
*Event tracking records what users do; attribution connects it to the ad that caused it.*

## Standard events vs custom events

Most SDKs split events into two kinds:

- **Standard events** — a fixed list the ad platform understands: `Purchase`, `CompleteRegistration`, `StartTrial`, `AddToCart`. Use these when they fit, because the ad platform can optimize for them directly.
- **Custom events** — your own named events for anything specific: `onboarding_step_2`, `shared_result`, `used_filter`. Good for your own analysis.

```dart
// Facebook (Meta) — facebook_app_events package
final facebook = FacebookAppEvents();

// standard-style events
await facebook.logPurchase(amount: 9.99, currency: 'USD');

// custom event with parameters
await facebook.logEvent(
  name: 'completed_onboarding',
  parameters: {'steps': 3},
);
```

The TikTok, AppsFlyer, and Adjust packages follow the same shape: an `init` call at startup, then `logEvent` / `logPurchase` calls where things happen.

## The common packages

- **facebook_app_events** — Meta (Facebook + Instagram) ads.
- **tiktok_business_sdk** — TikTok ads events.
- **appsflyer_sdk** / **adjust_sdk** — attribution platforms that sit *above* the ad networks and combine data from all of them into one dashboard. Teams often use one of these as the single source of truth.

A common setup is: one attribution SDK (AppsFlyer or Adjust) plus the ad-network SDKs it forwards events to. You log an event once, and it fans out to the right places.

![Standard vs custom events and the common packages](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/event-tracking/fig-2.png)
*Standard events the ad platform optimizes for, custom events for your own analysis — and the packages that send them.*

## Attribution and deep links

Attribution is closely tied to **deep links**. When a user taps an ad, the link can carry campaign data and also send them to a specific screen after install. This is called a **deferred deep link**: the app wasn't installed yet when they tapped, so the SDK "remembers" where to send them once the app opens for the first time.

```dart
// AppsFlyer-style: listen for the attribution data on first open
appsflyerSdk.onInstallConversionData((data) {
  // data tells you the campaign, and where to route the user
});
```

## The iOS privacy part: ATT and SKAdNetwork

On iOS this is where it gets tricky, and interviewers like to check if you know it.

- **ATT (App Tracking Transparency)** — since iOS 14.5, you must show a system popup asking permission before you can track a user across apps for ads. If they say no, you can't use their advertising ID.

```dart
// app_tracking_transparency package
final status = await AppTrackingTransparency.requestTrackingAuthorization();
```

- **SKAdNetwork** — Apple's privacy-friendly way to still measure ad conversions *without* tracking the individual user. Attribution SDKs handle most of this for you, but you should know it exists and that iOS attribution is more limited than Android because of it.

Always request ATT at a sensible moment (not on the very first screen), and remember that if the user declines, your event data on iOS will be less complete.

![iOS ATT permission and SKAdNetwork](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/event-tracking/fig-3.png)
*On iOS: ask permission with ATT, and measure conversions privately with SKAdNetwork.*

## Why this matters (and why it's a Flutter concern)

These SDKs sit at the boundary between the app and the business. As the mobile developer you own:

- **Correct placement** — logging `Purchase` in the exact spot where the purchase truly succeeds, with the right amount and currency.
- **No double-counting** — firing an event once, not on every rebuild.
- **Privacy compliance** — ATT prompt, consent, and not logging events before consent where required.
- **Performance** — initializing SDKs without blocking app startup.

Get these wrong and the marketing team optimizes on bad data and wastes ad budget — so this is a place where clean, careful mobile work has direct business impact.

## Key points

- Event tracking = **what users do**; attribution = **which ad caused it**.
- Ad platforms **optimize your ads** using the events you send (like `Purchase`).
- **Standard events** are understood by ad platforms; **custom events** are for your own analysis.
- Common packages: **facebook_app_events**, **tiktok_business_sdk**, **appsflyer_sdk** / **adjust_sdk**.
- **Attribution + deep links** connect an ad tap to an install and a screen.
- On iOS, know **ATT** (permission popup) and **SKAdNetwork** (privacy-safe conversion measurement).
- Your job: log the right event, once, in the right place, with consent — because the business optimizes on it.
