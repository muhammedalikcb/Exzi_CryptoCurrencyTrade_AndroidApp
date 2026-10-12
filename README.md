# Exzi

## Overview

Exzi is a sample Android app built to show a modern Android project setup. It talks to the Exzi exchange endpoints and has two screens: an order book (home) and a trade screen with a candlestick chart. Under the hood it uses MVVM, use cases, dependency injection with Hilt, and coroutines for polling live prices.

## Features

- Home screen: order book for BTC/USDT with buy and sell lists, pull-to-refresh, a "Check Your Internet Connection" message when offline, and polling that restarts when the connection comes back.
- Trade screen: daily candles for the selected pair drawn with MPAndroidChart, plus lists of recent highs and lows.
- Pairs are defined in `PairName` (BTC/USDT, ETH/USDT, XRP/USDT).
- Data comes from `https://socket.exzi.com` (`/book/list` for the order book, `/graph/hist` for candles).

## Technologies and Architectural Patterns

### MVVM (Model-View-ViewModel)

The UI (View), the business logic (ViewModel) and the data (Model) are kept in separate layers, which keeps each part easier to change and test.

- View: fragments such as `HomeFragment` and `TradeFragment` handle the UI and user interaction. Layouts use data binding.
- ViewModel: `HomeViewModel` and `TradeViewModel` hold the screen logic, call the use cases and expose results as `LiveData`.
- Model: data classes such as `CandleResponseModel` and `OrderBookResponseModel` describe the API responses.

### Use Cases

Business logic goes through use cases so it can be reused and tested on its own.

- `GetCandlesUseCase` fetches candle data.
- `GetOrderBookUseCase` fetches order book data.

Both call `GetRepository`, which returns a `Flow` of `NetworkResults` (`Loading`, `Success` or `Error`).

### Dependency Injection with Hilt

Hilt creates and provides the dependencies. `ExziApp` is annotated with `@HiltAndroidApp`, and `HomeViewModel` and `TradeViewModel` use `@HiltViewModel`.

All bindings live in one module, `core/di/NetworkModule.kt`. It provides the OkHttp client (with a 5 MB cache and a logging interceptor), Retrofit, the API client, the repository and the two use cases.

### Testing

Tests are in `app/src/androidTest`. `NegativeCandleAdapterTest` covers the adapter's `DiffUtil` callback and its `calculateInverseScaledValue` scaling logic.

- JUnit for writing and running tests.
- Mockito (`mockito-core`, `mockito-inline`, `mockito-android`) is set up for mocking.
- `androidx.arch.core:core-testing` is set up for testing LiveData and ViewModel code.

There are no ViewModel or use-case tests yet.

## Live Data Refresh

Prices are refreshed every 3 seconds. `HomeViewModel` polls the order book and `TradeViewModel` polls the candles, both with a coroutine loop in `viewModelScope`. The loop stops when the ViewModel is cleared or when `stopFetchingOrderBook()` or `stopFetchingCandles()` is called.

```kotlin
viewModelScope.launch {
    while (isActive) {
        fetchCandles()
        delay(3000)
    }
}
```

## Project Structure

```
com.muhammedalikocabey.exzi
├── core
│ ├── adapters
│ ├── di
│ │ └── NetworkModule.kt
│ └── utils
├── data
│ ├── model
│ ├── network
│ ├── remote
│ └── repository
├── domain
│ ├── repository
│ └── usecases
├── presentation
│ ├── base
│ ├── home
│ ├── trade
│ ├── viewmodels
│ └── MainActivity.kt
└── ExziApp.kt
```

## Setup Instructions

1. Clone the repository:
   ```
   git clone https://github.com/muhammedalikcb/Exzi_CryptoCurrencyTrade_AndroidApp/
   ```
2. Open the project in Android Studio.
3. Sync the project with Gradle files.
4. Run the application on an emulator or a physical device.

Build settings: compileSdk and targetSdk 34, minSdk 24, Kotlin 1.9.22, Android Gradle Plugin 8.2.2.

## Dependencies

- MPAndroidChart: `com.github.PhilJay:MPAndroidChart:v3.1.0`
- SwipeRefreshLayout: `androidx.swiperefreshlayout:swiperefreshlayout:1.1.0`
- Kotlin Serialization: `org.jetbrains.kotlinx:kotlinx-serialization-json:1.3.2`
- AndroidX libraries: Core KTX, AppCompat, Material, ConstraintLayout
- Navigation Components: `androidx.navigation:navigation-fragment-ktx:2.7.6`, `androidx.navigation:navigation-ui-ktx:2.7.6`
- ViewModel and LiveData: `androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0`, `androidx.lifecycle:lifecycle-livedata-ktx:2.7.0`
- DataStore: `androidx.datastore:datastore-preferences:1.1.0-beta01`
- Retrofit: `com.squareup.retrofit2:retrofit:2.9.0`, `com.squareup.retrofit2:converter-gson:2.9.0`
- OkHttp: `com.squareup.okhttp3:okhttp:5.0.0-alpha.3`, `com.squareup.okhttp3:logging-interceptor:5.0.0-alpha.3`
- Dagger Hilt: `com.google.dagger:hilt-android:2.49`, `com.google.dagger:hilt-compiler:2.49`
- Testing: JUnit, Mockito, AndroidX Test

## Screenshots

Trade Screen Design

![Trade Screen Design](screenshots/trade_design.png)

Order Book Screen Design

![Order Book Design](screenshots/order_book_design.png)

Trade Screen Implementation

![Trade Screen Implementation](screenshots/trade_xml.png)

Order Book Screen Implementation

![Order Book Implementation](screenshots/order_book_xml.png)
