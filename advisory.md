# Security Advisory — Ecoforest "Ecoforest Home" Stove Platform

**Researcher:** Pablo
**Disclosure coordination:** INCIBE-CERT (Spanish National CERT)
**Status:** Coordinated disclosure attempted; vendor non-responsive

---

## Affected product

- **Device tested:** Bio+ pellet stove WiFi module, model MX2013
- **Firmware:** v2a (04Oct2022), Ecoforest base
- **Platform:** Embedded Linux
- **Broader scope:** Ecoforest "Ecoforest Home" platform (see "Affected scope" below)

## Exposed attack surface

| Port | Service | Version |
|------|---------|---------|
| 22/tcp | SSH | Dropbear 2012.55 |
| 80/tcp | HTTP | thttpd 2.25b (29dec2003) |
| 443/tcp | HTTPS | thttpd 2.25b |
| 8000/tcp | HTTPS | thttpd 2.25b |

---

## F1 — Outdated SSH service with known pre-auth RCE (Critical)

The module exposes Dropbear SSH 2012.55, affected by **CVE-2016-7406**: a format-string
vulnerability allowing an unauthenticated remote attacker to execute arbitrary code with
root privileges via format specifiers in the username/host argument. Fixed in Dropbear
≥ 2016.74.

**Note on exploitability:** real-world exploitability depends on the specific compiled
binary; the deployed version is unambiguously within the affected range.

## F2 — Severely outdated web server (High)

The web interface runs thttpd 2.25b (2003), affected by **CVE-2009-4491** (CVSS 3.1: 9.8):
log injection via non-sanitized escape sequences, potentially leading to command execution
or file overwrite when logs are read in a vulnerable terminal.

## F3 — Predictable administrative credentials (Medium)

The local admin panel (HTTP Basic auth) derives its credentials from values that are not
independent secrets: the username corresponds to the device serial number (physically
visible on the stove's own screen), and the password is derived from the module's WiFi key.
There is no separation between credentials and no forced change on first use.

**Impact:** an actor with access to the module's network — or momentary physical/visual
access to the device screen — can authenticate to the administration panel.

---

## Affected scope beyond the tested device

Public documentation indicates findings F2 and F3 affect the broader Ecoforest "Ecoforest
Home" platform, not only the tested MX2013 module. Based exclusively on public sources; no
active testing was conducted against third-party devices.

| Source | What it documents |
|--------|-------------------|
| Home Assistant — Ecoforest integration | Standard login scheme (serial as username, first 8 chars of WiFi key as password) for *any* Ecoforest Home device; lists models e.g. Cordoba Glass |
| jangelfdez/Ecoforest-Api (GitHub) | Same thttpd 2.25b + CGI control API on "Bolonia" and air-water models |

The vulnerable web server, control API, and credential scheme appear shared across the
product line. **Not independently verified** on those specific models — stated as
documentation-based inference.

---

## Correctly implemented controls

- The control API (`recepcion_datos_4.cgi`) enforces authentication (verified: `401`
  returned for an unauthenticated read operation). No unauthenticated device control.
- The cloud management portal operates over HTTPS.
- The default WiFi access-point key is strong and independent of the SSID/serial.

---

## Recommendations (vendor)

1. Update Dropbear to ≥ 2016.74 and migrate thttpd to a maintained web server.
2. Use per-device credentials, independent from the WiFi key, with forced change on setup.
3. Disable the configuration access point when not in use.
4. Implement HSTS on the web portal.

## Recommendations (users)

- Do not expose the module to the Internet (no port forwarding to 22/80/443/8000).
- Keep the device on an isolated network segment (IoT VLAN / guest network).
- Prefer the cloud app (HTTPS) over exposing the local panel.

---

## References

- CVE-2016-7406 — https://nvd.nist.gov/vuln/detail/CVE-2016-7406
- CVE-2009-4491 — https://nvd.nist.gov/vuln/detail/CVE-2009-4491
- Home Assistant — Ecoforest integration
- GitHub: jangelfdez/Ecoforest-Api