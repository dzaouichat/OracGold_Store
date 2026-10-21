# OracGold Store — Setup Guide

**Your own app store. Your repo. Your rules.**

OracGold Store is an independent, clean-room app store for your device.
It reads its catalog **only** from your repository
(`github.com/dzaouichat/OracGold_Store`) — no third party can add, remove,
lock, or remote-control anything on your device.

---

## ⚡ Quick Start (Arabic) — البداية السريعة

1. انشئ مستودع `dzaouichat/OracGold_Store` على GitHub (إن لم يكن موجوداً) واجعله **Public**.
2. ارفع ملف `apps.json` (من مجلد `repo-files`) إلى جذر المستودع.
3. ارفع ملفات `.apk` الخاصة بك عبر **Releases** (أنشئ Release مثلاً بعنوان `v1` وارفع فيها ملفات APK).
4. انسخ رابط كل APK من Release وضعه في حقل `"url"` داخل `apps.json` مع اسم التطبيق.
5. ثبّت `OracGold_Store.ngz` على جهازك بنفس طريقة تثبيت `UsbSideload.ngz` السابق.
6. افتح المتجر من قائمة الجهاز — تطبيقاتك ستظهر، اختر واحداً واضغط **OK** للتنزيل والتثبيت.

---

## 1. Install the store on your device

Copy `OracGold_Store.ngz` to a USB drive and install it the same way you
installed the original plugin package (same `.ngz` container format):

```
data/plugin/OracGold_Store         <- static ARM binary (v1.0.00, code 1000)
data/plugin/OracGold_Store.descr   <- plugin descriptor (|DZ| format)
data/plugin/OracGold_Store.png     <- golden orca icon (128x128)
```

## 2. Prepare your GitHub repository

Your repo must be **public** and contain `apps.json` at its root:

```
https://github.com/dzaouichat/OracGold_Store
└── apps.json          <- the catalog (this file)
```

The store fetches the catalog with a 3-tier failover, in this order:

1. `https://raw.githubusercontent.com/dzaouichat/OracGold_Store/main/apps.json` (fresh)
2. `https://cdn.jsdelivr.net/gh/dzaouichat/OracGold_Store@main/apps.json` (CDN mirror)
3. `https://api.github.com/repos/dzaouichat/OracGold_Store/contents/apps.json` (API)

So new apps appear within seconds of pushing `apps.json`.

## 3. Add an app to your store

Upload the `.apk` (and an icon `.png`, optional) to a **GitHub Release** in
your repo, then add an entry to `apps.json`:

```json
[
  {
    "name": "NewPipe",
    "version": "1.0",
    "app_icon": "https://github.com/dzaouichat/OracGold_Store/releases/download/v1/newpipe.png",
    "url": "https://github.com/dzaouichat/OracGold_Store/releases/download/v1/NewPipe.apk"
  }
]
```

| Field       | Required | Meaning                                   |
|-------------|----------|-------------------------------------------|
| `name`      | yes      | Display name in the store list            |
| `url`       | yes      | Direct APK download URL (GitHub Releases) |
| `version`   | no       | Version label (default `1.0`)             |
| `app_icon`  | no       | Icon URL (PNG/JPG; placeholder if absent) |
| `description` | no    | Reserved for future use                   |

**To remove an app from your store:** delete its entry from `apps.json`
and push. It disappears from the device on the next refresh (press `0`).

## 4. Optional: update the store itself (self-update)

If you release a newer store binary, tell the devices about it by switching
`apps.json` to the object form:

```json
{
  "store_version": "1.1",
  "store_update_url": "https://github.com/dzaouichat/OracGold_Store/releases/download/v1.1/OracGold_Store",
  "store_desc_url": "https://github.com/dzaouichat/OracGold_Store/releases/download/v1.1/OracGold_Store.descr",
  "store_icon_url": "https://github.com/dzaouichat/OracGold_Store/releases/download/v1.1/OracGold_Store.png",
  "apps": [ ... same app entries as before ... ]
}
```

When a device opens the store and sees `store_version` newer than its own,
it downloads and installs the update automatically, then restarts.
**Only you** can publish this, because only your repo is ever contacted.

## 5. Controls (device remote)

| Key            | Action                                          |
|----------------|-------------------------------------------------|
| `OK` / Enter   | Download + install selected app                 |
| `0`            | Store tab: refresh catalog · USB tab: rescan    |
| `Left/Right`   | Switch Store / USB tab                          |
| `Up/Down`      | Navigate list                                   |
| `EXIT`         | Close (also cancels a running download)         |

The **USB tab** lists every `.apk` on attached USB drives and installs them
directly — always allowed, never remotely lockable.

## 6. What this store will NEVER do

- Contact any server other than **your** repository
- Read remote "allow/deny" gates (no `allow_usb_install`-style locks)
- Push updates you did not publish yourself
- Install anything that is not in your catalog or on your USB drive

## 7. Logs (troubleshooting)

On the device, the store writes a plain log you can inspect:

```
/tmp/oracgold.log               <- main log
/tmp/oracgold_install.log       <- pm install output of last install
/tmp/oracgold_update_log.txt    <- self-updater log (if an OTA ran)
```

## 8. Building from source (optional)

Full clean-room source is included in `source/`:

```
src/util.{h,cpp}    paths, logging, strings, base64, versions
src/json.{h,cpp}    minimal JSON parser
src/http.{h,cpp}    curl/wget wrapper, background download/install
src/gfx.{h,cpp}     framebuffer, stb_truetype font, stb_image icons
src/store.{h,cpp}   catalog, icons, USB scan, install, launcher, OTA
src/input.{h,cpp}   evdev key reader
src/main.cpp        state machine + gold UI
stb/                public-domain stb_image / stb_truetype
```

Build (Bootlin ARMv7 musl toolchain expected at the path in the Makefile):

```
make          # static ARM binary -> package/OracGold_Store
make package  # + OracGold_Store.ngz
```

Host-side logic tests (JSON parser, XML launcher surgery, base64, versions):

```
g++ -std=c++17 -Isrc -o tests/test_logic tests/test_logic.cpp src/util.cpp src/json.cpp
./tests/test_logic
```

---
OracGold Store v1.0.00 · English UI · Golden theme · Sole owner: you
