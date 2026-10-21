# Publishing checklist

Everything in this folder is ready to be published: the mod source, README with
screenshots, MIT license and this checklist. Two things are left for you,
because they need your GitHub account.

## 1. Replace the placeholder

`deepseek-balance-tray.wh.cpp` has a placeholder repository URL:

```
// @github          https://github.com/MrAlexFreeman/deepseek-balance-tray
```

Replace `MrAlexFreeman` with your GitHub username (the same name you will use for the
repository below), e.g. `https://github.com/octocat/deepseek-balance-tray`.

Also check `@author` — it currently says `DS Balance`; put your name or nick if
you prefer.

## 2. Push the repository to GitHub

The local git repository is already initialized with one commit. Create an empty
repository named `deepseek-balance-tray` on GitHub, then:

```powershell
cd dist
git remote add origin https://github.com/MrAlexFreeman/deepseek-balance-tray.git
git push -u origin main
```

After the push, the `@github` link in the source is valid and the README
screenshots render on GitHub.

## 3. Submit to the Windhawk catalog

Mods from the catalog are installed in two clicks by every Windhawk user, so
this is the step that actually shares the mod with the world.

1. Fork [ramensoftware/windhawk-mods](https://github.com/ramensoftware/windhawk-mods).
2. Copy `deepseek-balance-tray.wh.cpp` into the `mods/` folder of your fork
   (the file name must match the `@id`).
3. Commit with a message like `Add DeepSeek Balance in Taskbar`.
4. Open a pull request against `main` and paste the description below.

The repository's CI (`pr_validation.py`) checks the metadata block, the mod ID,
the version and that the mod compiles for x86 and x64. Common reasons for a
failed check:

* `@id` already exists in the catalog (checked: `deepseek-balance-tray` is free);
* a required field is missing (`@author`, `@version`, `@github`, `@id`);
* the metadata block must start at line 1 and be closed.

## 4. PR title and description (ready to paste)

**Title:** `Add DeepSeek Balance in Taskbar`

**Description:**

```markdown
Adds a mod that shows the current DeepSeek API balance in the taskbar clock.

![screenshot](https://raw.githubusercontent.com/MrAlexFreeman/deepseek-balance-tray/main/screenshots/clock.png)

* Hooks `GetTimeFormatEx` / `GetDateFormatEx` from `kernelbase.dll` and only
  decorates calls made while the taskbar clock is refreshing (Windows 10:
  `ClockButton::UpdateTextStringsIfNecessary`, Windows 11:
  `ClockSystemTrayIconDataModel::RefreshIcon`), so other date formatting in
  Explorer is untouched.
* Polls `https://api.deepseek.com/user/balance` over WinHTTP on a background
  thread; no external libraries, JSON is parsed by hand.
* Optional `RUB` mode converts the balance using the official Bank of Russia
  rate (`https://www.cbr.ru/scripts/XML_daily.asp`, cached for 6 hours).
* Settings: API key, refresh interval, currency, currency prefix, which clock
  line to use, native currency vs rubles.
* The API key is stored in the mod settings in plain text (Windhawk has no
  secret setting type); it is only sent to `api.deepseek.com` and never logged.

Source repository: https://github.com/MrAlexFreeman/deepseek-balance-tray

Tested on Windows 11 build 28000 with Windhawk 2.0-alpha.1.
```

Replace `MrAlexFreeman` in the raw screenshot URL as well.

## 5. After the merge

* The mod appears on <https://windhawk.net/mods> and in the in-app browser.
* For updates, bump `@version`, add a `changelog_override/<id>/<version>.md`
  file if you want custom release notes, and open a new PR (see the
  "Submitting a Mod Update" section of the `windhawk-mods` README).

## Notes for reviewers (and for you)

* Code comments are in Russian; user-facing text (readme, setting names and
  descriptions) is in English.
* The mod is read-only with respect to Explorer: it never modifies the XAML
  tree, only the text returned by the formatting functions.
* Everything on the network side has timeouts (4 s DNS/connect, 8 s send and
  receive) and all WinHTTP handles are closed through RAII.
