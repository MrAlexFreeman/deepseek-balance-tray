# DeepSeek Balance in Taskbar

A [Windhawk](https://windhawk.net/) mod that shows your DeepSeek API account
balance right in the Windows taskbar clock.

![Taskbar clock with the balance](screenshots/clock.png)

![Notification area](screenshots/taskbar.png)

## Features

* Balance is appended to the taskbar clock: `22.10.2026 | ≈1 058 ₽`.
* Balance in its original currency (`native`), or converted to Chinese yuan
  (`¥`), Russian rubles (`₽`) or US dollars (`$`) with the official Bank of
  Russia rates. Conversion is opt-in: with `native` no extra request is made.
* Configurable refresh interval, display currency and which clock line to use.
* Networking uses the built-in WinHTTP stack; JSON and XML are parsed by hand,
  no libcurl/OpenSSL/JSON libraries.
* All network work happens on a background thread with Win32 timeouts, so
  Explorer never blocks; the mod unloads cleanly.

## Installation

### From the Windhawk catalog

Once the mod is published, open Windhawk → **Browse** → search for
**DeepSeek Balance in Taskbar** → **Install**.

### From source

1. Install [Windhawk](https://windhawk.net/) 1.4 or newer.
2. Windhawk → **Create new mod**, paste the contents of
   [`deepseek-balance-tray.wh.cpp`](deepseek-balance-tray.wh.cpp), press
   **Compile**.
   The first local build may ask to download the development tools (LLVM).
3. Open the mod settings and paste your DeepSeek API key (`sk-...`).

Or with the CLI:

```powershell
windhawk-cli mod install --file .\deepseek-balance-tray.wh.cpp
```

## Settings

| Setting | Default | Description |
| --- | --- | --- |
| `apiKey` | empty | DeepSeek API key. If empty, the `DEEPSEEK_API_KEY` environment variable is used. |
| `refreshInterval` | `5` | How often to poll the API, in minutes (1…1440). |
| `clockLine` | `date` | Append to the bottom (date) or top (time) clock line. |
| `displayCurrency` | `native` | `native` keeps the API currency and makes no extra request; `CNY`, `RUB` and `USD` convert with the CBR rates. |

## How it works

* The mod hooks `GetTimeFormatEx` and `GetDateFormatEx` from `kernelbase.dll`,
  the functions the taskbar clock uses to format its text. On Windows 10 the
  clock lives in `ClockButton::UpdateTextStringsIfNecessary` (explorer.exe), on
  Windows 11 in `ClockSystemTrayIconDataModel::RefreshIcon`
  (`SystemTray.dll` / `Taskbar.View.dll` / `ExplorerExtensions.dll`).
* Those functions only flag "we are inside a clock refresh" (thread-local), so
  the decoration is applied to the clock and nothing else — dates in file
  properties and other Explorer UI stay untouched.
* A background `std::thread` polls
  `https://api.deepseek.com/user/balance` over WinHTTP and updates the suffix.
* To show the balance in a currency other than the API one, the mod downloads
  `https://www.cbr.ru/scripts/XML_daily.asp` (windows-1251, `Value / Nominal`),
  caches the rates for 6 hours and converts the amount. This happens only when
  `displayCurrency` is set to `CNY`, `RUB` or `USD` and differs from the
  currency returned by the API; the default `native` makes no such request.
* On network errors the last known value is kept; if a rate is missing, the
  balance is shown in its original currency instead of an empty line.

## Privacy

* The API key is stored in Windhawk's mod settings in plain text (Windhawk has
  no secret setting type). It is sent only to `api.deepseek.com` and is never
  written to the log.
* In a regular (non-portable) Windhawk installation the mod settings are
  machine-wide, so every Windows account on the PC shares the same key.
* The only other network endpoint is `www.cbr.ru` (Bank of Russia), used only
  when the selected display currency differs from the API currency. No
  telemetry, no analytics, no third-party services.

## Compatibility

* Windows 10 (stock taskbar) and Windows 11 (XAML taskbar), x86-64 only: the
  internal clock symbols and calling conventions used here are 64-bit.
* Requires symbols (PDB) for `explorer.exe` / `SystemTray.dll`; Windhawk
  downloads them automatically on first start.
* The old Windows 10 taskbar on Windows 11 via ExplorerPatcher is not supported.
* Windhawk 1.4+.
* Do not enable it together with **Taskbar Clock Customization**: both hook the
  same clock formatting functions, and that mod replaces the line text, so only
  one of them can win.

## Building

The mod is a single self-contained `.wh.cpp` file. Windhawk compiles it locally
with its own clang toolchain and MSVC headers:

1. Windhawk → **Create new mod** → paste the source → **Compile**.
2. Or `windhawk-cli mod compile deepseek-balance-tray` for an installed copy.

No additional libraries are required; `@compilerOptions -lwinhttp -lversion`
links the WinHTTP and version import libraries.

## License

MIT — see [LICENSE](LICENSE).
