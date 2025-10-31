# MyAnalytics

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Overview

**MyAnalytics** is a lightweight example of a custom analytics library for Android applications. It demonstrates how to use Android's `ContentProvider` mechanism to **initialize analytics tracking early in the app lifecycle** — specifically, **before** the `Application` class's `onCreate()` method or any `Activity` is launched.

This ensures that critical analytics events, such as **app starts**, are captured reliably without being missed due to delays in standard initialization.

> **Why does this matter?**  
> `ContentProvider` is one of the **first components** initialized by the Android system during app startup:
>
> ```
> ContentProvider → Application.onCreate() → Activity.onCreate()
> ```
>
> By placing initialization logic in a `ContentProvider`, you guarantee it runs **first**.

This technique is commonly used in production libraries (e.g., Firebase, Crashlytics, DI frameworks) for early setup.

---

## Key Features

- **Early Initialization** – Starts before `Application` or `Activity`
- **Zero Boilerplate** – No need to call `init()` manually
- **Simple Event API** – Log events with parameters
- **ContentProvider-Powered** – Transparent lifecycle hook
- **Lightweight** – No external dependencies

---

## Project Structure
```
app/
├── src/main/java/com/bask0xff/myanalytics/
│   ├── AnalyticsLibrary.java     → Core analytics engine
│   ├── AnalyticsProvider.java    → ContentProvider bootstrap
│   └── MainActivity.java         → Demo usage
├── AndroidManifest.xml           → Declares provider
└── build.gradle                  → Module config
```

### Key Classes

#### `AnalyticsLibrary.java`
```java
public class AnalyticsLibrary {
    private static AnalyticsLibrary instance;

    public static void init(Context context) {
        if (instance == null) {
            instance = new AnalyticsLibrary(context);
            Log.d("AnalyticsLibrary", "Analytics library initialized for " + context.getPackageName());
        }
    }

    public static AnalyticsLibrary getInstance() {
        return instance;
    }

    public void logEvent(String eventName, Map<String, Object> params) {
        long timestamp = System.currentTimeMillis();
        if (params == null) params = new HashMap<>();
        params.put("timestamp", timestamp);
        Log.d("AnalyticsLibrary", "Event: " + eventName + ", Params: " + params);
        // In production: send to server, store in DB, etc.
    }
}
```
#### `AnalyticsProvider.java`
```java
public class AnalyticsProvider extends ContentProvider {
    @Override
    public boolean onCreate() {
        AnalyticsLibrary.init(getContext());

        // Log app start event immediately
        AnalyticsLibrary.getInstance().logEvent("app_started", null);

        Log.d("AnalyticsProvider", "Analytics initialized");
        return true;
    }

    // Required overrides (return null/false for demo)
    @Override public Cursor query(...) { return null; }
    @Override public Uri insert(...) { return null; }
    @Override public int delete(...) { return 0; }
    @Override public int update(...) { return 0; }
    @Override public String getType(...) { return null; }
}
```
#### `AndroidManifest.xml`
```xml
<provider
    android:name=".AnalyticsProvider"
    android:authorities="${applicationId}.analytics"
    android:enabled="true"
    android:exported="false" />
```
|| Replace ${applicationId} with your actual package name (e.g., com.example.myapp.analytic

## Setup Instructions
1. Add to Your Project

Option A: Use as Library Module

Clone this repo
Add the app module to your project
In your app's build.gradle:

```
dependencies {
    implementation project(':myanalytics')
}
```

Option B: Copy Source Files
Just copy:
- AnalyticsLibrary.java
- AnalyticsProvider.java
Into your project.

2. Update AndroidManifest.xml
Add inside <application>:
```
<provider
    android:name="com.bask0xff.myanalytics.AnalyticsProvider"
    android:authorities="${applicationId}.analytics"
    android:enabled="true"
    android:exported="false" />
```
| Use a unique authority based on your app's package.

3. Use in Code
No manual init required!
```
// In any Activity, Fragment, or Service
Map<String, Object> params = new HashMap<>();
params.put("screen", "HomeFragment");
AnalyticsLibrary.getInstance().logEvent("screen_view", params);
```

## Usage Example (Logcat Output)
```
2025-04-28 10:00:56.122 - AnalyticsLibrary - Analytics library initialized for com.bask0xff.myanalytics
2025-04-28 10:00:56.124 - AnalyticsLibrary - Event: app_started, Params: {timestamp=1745827256122}
2025-04-28 10:00:56.124 - AnalyticsProvider - Analytics initialized
2025-04-28 10:00:56.688 - AnalyticsLibrary - Event: main_activity_opened, Params: {screen=MainActivity}
```
Notice: app_started is logged before MainActivity appears!

## Extending for Production

Feature,How to Add
Send to Backend: Use OkHttp, Retrofit, or WorkManager
Offline Storage: Save events to Room DB
Batching: Flush every N events or on connectivity
User Consent: Add opt-out flag + GDPR check
Testing: Mock ContentProvider with ProviderTestCase2

## Limitations (Demo Only)
Perfect for:
- Crash reporting (like ACRA, Sentry)
- Dependency injection (Hilt, Koin)
- Feature flags
- Remote config prefetch
- Any "must-run-first" logic

## Contributing
Contributions welcome! Feel free to:
- Open issues
- Submit PRs
- Improve logging, add tests, etc.

## License
MIT License

