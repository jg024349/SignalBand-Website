# SignalBand OTA releases

## Release 1.0.9 build 10

This repository serves two separate signed firmware update feeds:

| Watch | Manifest |
| --- | --- |
| Standard T-Watch S3, SX1262/US915 | `/updates/latest.json` |
| Rugged T-Watch Ultra, SX1262/US915 | `/updates/ultra/latest.json` |

Each feed contains a versioned application binary beside `latest.json`. Keep
older binaries available. Do not rename, edit or mix manifests and images.
No signing keys or USB flash images belong in the website repository.

## Deploy

Push this repository to GitHub, then pull/clone it into the existing website
server's document root. Retain all existing website content and update files.
Make sure BOTH paths above are served by `https://www.signalbandllc.com` with
HTTP 200, no login or redirects, and valid public TLS. These files are update
feeds and do not require changing the visible marketing pages.

`_headers` supplies rules for Cloudflare Pages/Static Assets. If Cloudflare
proxies a separate Nginx or Apache server, that origin does not read `_headers`;
configure these equivalent response rules there if they are not already set:

- `latest.json`: Content-Type application/json; Cache-Control no-store, no-transform.
- `.bin`: Content-Type application/octet-stream; Cache-Control public, max-age=31536000, immutable, no-transform.
- Preserve Content-Length and disable compression/transformation on update files
  for compatibility with earlier S3 updater versions.

After deployment, run both live checks from the firmware project:

```powershell
node tools/firmware-hosting.mjs verify-live --target s3
node tools/firmware-hosting.mjs verify-live --target ultra
```

## Enable existing Ultra watches

Ultra development firmware had OTA disabled. Upload the new Ultra sketch via
USB once, using LilyGo T-Watch-Ultra, Radio-SX1262, Hardware CDC/JTAG, USB CDC
enabled, and the 16 MB / 3 MB app partition setting. Do not erase saved settings.
The S3 uses its own S3 sketch and board selection. No watch has been flashed by
this release preparation.

A watch already running 1.0.9 will report up to date until a newer release is
published. A release does not automatically install: open Firmware Update,
connect to internet Wi-Fi/hotspot, check, then tap Install on the watch. An
actual firmware installation restarts the watch to load the new application.

Builds and signed packages are verified locally. Test a watch of each model
after deployment; local verification does not prove real-device connectivity.
