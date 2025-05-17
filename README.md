Example of a Custom Analytics Library Using a Content Provider

Usage Log Example:

  2025-04-28 10:00:56.122 - AnalyticsLibrary -        Analytics library initialized for com.bask0xff.myanalytics  
  2025-04-28 10:00:56.124 - AnalyticsLibrary -        Event: app_started, Params: {timestamp=1745827256122}  
  2025-04-28 10:00:56.124 - AnalyticsProvider -       Analytics initialized  
  2025-04-28 10:00:56.688 - AnalyticsLibrary -        Event: main_activity_opened, Params: {screen=MainActivity}

As you can see, the library is initialized before MainActivity is started.
