# Analytics and Crash Reporting in Flutter: Knowing What Happens in Your App

*How analytics packages tell you what users do, and how crash/error reporting tells you what broke — plus how to wire them up in Flutter.*

---

Once your app is in real users' hands, you're flying blind without two kinds of tools. **Analytics** tells you *what users do* — which screens, which features, where they drop off. **Crash and error reporting** tells you *what broke* — which line threw, on which device, for how many users. Every serious production app has both. Let me explain them and show the Flutter setup.

## Two different jobs

- **Analytics** answers product questions: *How many users finished onboarding? Where do people quit the paywall? Which feature is used most?*
- **Crash/error reporting** answers stability questions: *What crashed? What's the stack trace? How many users hit it? On which OS version?*

They're separate tools because they serve different teams — analytics helps product and growth, crash reporting helps engineering keep the app stable.

![Analytics funnel vs crash reporting](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/analytics-crash/fig-1.png)
*Analytics shows where users drop off; crash reporting shows what broke and for how many.*

## Analytics: logging events and funnels

Analytics works by logging **events** — named things that happen — often with parameters and user properties. The common packages are **firebase_analytics**, **mixpanel_flutter**, and **amplitude_flutter**.

```dart
// firebase_analytics
final analytics = FirebaseAnalytics.instance;

await analytics.logEvent(
  name: 'paywall_viewed',
  parameters: {'source': 'settings'},
);

// user properties describe the user, not a single action
await analytics.setUserProperty(name: 'plan', value: 'free');
```

From a series of events you build a **funnel**: `paywall_viewed` → `checkout_started` → `purchase_completed`. If lots of users reach step 1 but few reach step 3, you've found where you lose money. This is exactly how teams improve conversion.

Two rules that matter:

- **Name events consistently.** Decide a scheme (`object_action`, like `paywall_viewed`) and stick to it. Messy names make the data useless.
- **Log once, in the right place.** Fire the event where the thing actually happens — not inside `build`, which runs many times.

## Crash reporting: catching what breaks

Crash reporting tools capture errors automatically, with the stack trace and device info, and group them so you see "this crash hit 300 users" instead of 300 separate reports. The common tools are **firebase_crashlytics** and **sentry_flutter**.

The important part in Flutter is **catching every kind of error**. There are a few sources, and you need to hook all of them:

```dart
import 'dart:async';
import 'dart:ui';

void main() {
  WidgetsFlutterBinding.ensureInitialized();

  // 1. Errors thrown inside the Flutter framework (build, layout, etc.)
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;

  // 2. Errors from outside Flutter (async, platform)
  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    return true;
  };

  runApp(const MyApp());
}
```

- **`FlutterError.onError`** catches errors inside the Flutter framework.
- **`PlatformDispatcher.instance.onError`** catches async errors that escape the framework.

Together they make sure a crash almost never disappears silently.

![Catch both Flutter error sources](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/analytics-crash/fig-2.png)
*Hook both error sources so a crash never disappears silently — and record handled (non-fatal) errors too.*

## Fatal vs non-fatal errors

Not every error crashes the app. Sometimes you *handle* an error but still want to know it happened — a failed network call, a parsing error you recovered from. These are **non-fatal** errors, and you report them by hand:

```dart
try {
  await fetchProfile();
} catch (e, stack) {
  // handled, but still recorded so you can see how often it happens
  await FirebaseCrashlytics.instance.recordError(e, stack, fatal: false);
}
```

Fatal errors tell you what killed the app; non-fatal errors reveal problems users quietly hit without complaining.

## Breadcrumbs and user context

A stack trace alone often isn't enough — you want to know *what led up to* the crash. Both tools let you attach context:

- **Logs / breadcrumbs** — a trail of recent actions (`log('opened checkout')`) that show up next to the crash.
- **Keys / user id** — attach the current screen, the user's plan, or an anonymous user id, so you can see who and where.

```dart
FirebaseCrashlytics.instance.log('tapped subscribe');
FirebaseCrashlytics.instance.setCustomKey('screen', 'paywall');
```

Now when a crash comes in, you don't just see the line — you see the path the user took to get there.

![Breadcrumbs and context keys show the path to a crash](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/analytics-crash/fig-3.png)
*Breadcrumbs and keys turn a bare stack trace into the full path the user took.*

## Sentry: an alternative in one init

Sentry does both crash reporting and performance monitoring and is set up in a single wrapper:

```dart
await SentryFlutter.init(
  (options) => options.dsn = 'https://...',
  appRunner: () => runApp(const MyApp()),
);
```

Using `appRunner` lets Sentry wrap your whole app so it catches errors automatically, without wiring each handler yourself.

## How they work together

A healthy production app usually runs **both**: analytics (Firebase Analytics or Mixpanel) to understand behavior, and crash reporting (Crashlytics or Sentry) to stay stable. Analytics tells you a feature is popular; crash reporting tells you that same feature crashes on old Android versions. You need both views to ship confidently and fix the right things first.

## Key points

- **Analytics** = what users do (events, funnels, user properties). Packages: **firebase_analytics**, **mixpanel_flutter**, **amplitude_flutter**.
- **Crash/error reporting** = what broke (stack traces, grouped, device info). Tools: **firebase_crashlytics**, **sentry_flutter**.
- In Flutter, catch **both** error sources: `FlutterError.onError` and `PlatformDispatcher.instance.onError`.
- **Non-fatal** errors (`recordError(..., fatal: false)`) reveal handled problems users don't report.
- Add **breadcrumbs and keys** (screen, user id) so a crash shows the path that led to it.
- Real apps run analytics **and** crash reporting together — behavior plus stability.
