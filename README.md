# tr-imei-checker

CLI tool for querying IMEI registration status via Turkey's e-government portal
(`turkiye.gov.tr/imei-sorgulama`), built in C++17 with libcurl.

The public IMEI query portal lets anyone verify whether a device is officially
registered with BTK (Bilgi Teknolojileri ve İletişim Kurumu). This is relevant
when buying or selling secondhand phones in Turkey — an unregistered or
blacklisted IMEI means the device may be blocked from GSM networks.

This project was a learning exercise in C++ HTTP client programming: handling
stateful web sessions, CSRF token extraction, and HTML response parsing without
a DOM library.

## What it does

For each IMEI in `list.txt`, the tool:

1. `GET /imei-sorgulama` → extracts the hidden CSRF token from the response HTML
2. `POST /imei-sorgulama?submit` → submits the IMEI with the token
3. Parses the result page and prints:

```
IMEI:
Durum:
Kaynak:
Sorgu Tarihi:
Marka/Model:
```

## Technical highlights

- **CSRF handling** — token extracted from hidden form field via string search;
  URL-encoded and injected into the POST body
- **Cookie lifecycle** — libcurl manages the session cookie between GET and POST
  so the portal accepts the submission
- **Batch mode** — reads one IMEI per line from `list.txt`, queries sequentially
- **Proxy support** — optional; reads `proxy.txt` (`ip:port` or
  `ip:port:user:pass`). Intended for cases where the same machine is checking
  a large number of devices (e.g. a repair shop's incoming stock)
- **Platform** — Windows, MSVC, C++17, statically linked libcurl

## Flow

```
list.txt (one IMEI per line)
       │
       ▼
GET /imei-sorgulama  →  extract CSRF token
       │
       ▼
POST /imei-sorgulama?submit  →  parse Durum block
       │
       ▼
stdout: IMEI / Durum / Kaynak / Sorgu Tarihi / Marka-Model
```

## Files

| File | Description |
|---|---|
| `list.txt` | IMEI numbers to query, one per line |
| `proxy.txt` | Optional proxy pool (`ip:port` or `ip:port:user:pass`) |
| `cookies.txt` | Session cookie store managed by libcurl at runtime |

## Build

Requires libcurl (static linkage recommended). A Visual Studio solution is
included (`*.vcxproj`).

Open in Visual Studio, set to Release x64, build.

Or link manually:

```
cl /std:c++17 tr-imei-checker.cpp /link libcurl.lib ...
```

## Usage

```
tr-imei-checker.exe

▎ Proxy listenizdeki proxyler şifreli mi?

1. Evet   (ip:port:user:pass)
2. Hayır  (ip:port)
▎ ▎ 2
```

The tool reads `list.txt` and prints results to stdout sequentially.

## Note on responsible use

This tool queries a public government service. Use it only for devices you own
or are evaluating for purchase. Respect the portal's rate limits; the proxy
option exists for legitimate bulk scenarios (e.g. a reseller checking incoming
stock), not for circumventing access controls.

## License

MIT

