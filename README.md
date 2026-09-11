# SignalBand Website

Static marketing site for SignalBand.

Open `index.html` in a browser to preview locally.

## Watch Firmware Updates

The watch checks `https://www.signalbandllc.com/updates/latest.json`.
The `updates/` directory contains a signed release manifest and versioned
application binaries. The initial staged release is **1.0.0 (build 1)**.
It has compiled and passed local release/hosting checks, but still needs live
HTTPS verification and physical watch testing before commercial release.
This repository change does not deploy the site or flash
any watches. Existing watches need an initial USB installation of the updater
configured for this address before they can update over Wi-Fi.

Only generated release files go in `updates/`. Never edit `latest.json` by hand,
replace an existing versioned binary, or upload a private signing key, merged
flash image, bootloader, partition image or firmware source folder. The release
tool needs the original private key for future releases; keep it backed up
privately, outside this website repository.

### Prepare A Release

From the **Softball Pitch Calling firmware project**, increase `version` and
`versionCode` in `receiver-firmware/release.json`, run the configure command,
build and test the watch firmware, then package the application binary:

```powershell
node tools/firmware-release.mjs configure
# Compile the watch sketch with the documented T-Watch S3 Arduino settings.
node tools/firmware-release.mjs package --bin .ota-build/PitchCallReceiver.ino.bin
node tools/firmware-hosting.mjs stage --release firmware-releases/1.0.0-1 --site C:/path/to/SignalBand-Website
node tools/firmware-hosting.mjs verify --site C:/path/to/SignalBand-Website
```

Replace the example checkout path and release directory with the actual ones.
The `stage` command verifies the signature, version, image identity and checksum,
copies only the two public release files, and replaces the manifest last. It
retains older binaries and rejects downgrades or conflicting versioned files.
It never commits, pushes or deploys. The initial release is already staged in
this checkout; do not rebuild a different image under the same version code.

### Deploy To Cloudflare

Keep the existing static-site deployment method. No npm install, database,
new build command or website redesign is required.

- For Git-connected hosting, review and commit the website changes, including
  `_headers` and the complete `updates/` directory, then push through your usual
  production deployment workflow after testing.
- For Pages Direct Upload, deploy the **complete website folder**, including
  `_headers`, the existing pages/images, and all update binaries. Do not deploy
  only the updates folder over the website. Exclude `.git`, `.vs` and private
  files from any upload archive.
- Keep earlier versioned `.bin` files in future deployments, including website
  rollbacks, so a watch that already fetched a manifest can finish its download.
- If a previously published release has a problem, publish a corrected release
  with a higher version code. Pointing `latest.json` at an older release will
  not downgrade watches that already installed the newer version.

The root `_headers` config applies only to `/updates/`. It prevents caching of
`latest.json`, permits long caching of immutable binaries, supplies file types,
and sets `no-transform` so Cloudflare can preserve `Content-Length`. This format
is supported by [Pages static assets](https://developers.cloudflare.com/pages/configuration/headers/)
and [Workers Static Assets](https://developers.cloudflare.com/workers/static-assets/headers/).
It does not configure an unrelated origin server or custom Worker responses.
If Cloudflare only proxies another host, configure equivalent headers there.

The watch needs HTTP 200 with a positive `Content-Length` over HTTP/1.1, without
redirects, compression, login prompts or bot challenges. Keep the exact `www`
update address working; don't redirect it to the bare domain. If dashboard rules
override these paths, adjust only `/updates/` as needed, not site-wide security.
The checker below will identify incompatible responses. Cloudflare documents
the `no-transform` behavior in its [compression guide](https://developers.cloudflare.com/speed/optimization/content/compression/).

### Verify After Deployment

From the firmware project, run:

```powershell
node tools/firmware-hosting.mjs verify-live
```

This downloads the public manifest and binary with HTTPS certificate checking
and HTTP/1.1, rejects redirects/compression/chunked responses, checks headers,
and verifies the signature, size, checksum and compiled release identity. It
also rejects a published version that differs from `release.json`. It
does not install firmware. Test real Master and Player watches over a phone
hotspot, including interrupted downloads and rollback, before shipping.
