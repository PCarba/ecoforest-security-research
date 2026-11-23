# Security Research: Ecoforest "Ecoforest Home" Stove Platform

> Independent security research — responsibly disclosed via INCIBE-CERT (Spanish National CERT)

## Summary

Security assessment of the WiFi control module of a commercial Bio+ pellet stove (Ecoforest
firmware base). The research identified outdated base software affected by publicly known
critical CVEs and a predictable administrative credential scheme. Public documentation
indicates the vulnerable web server, control API, and credential scheme are shared across
the Ecoforest product line ("Ecoforest Home" platform), not specific to the tested model.

## Affected Product

| | |
|---|---|
| **Device tested** | Bio+ pellet stove WiFi module |
| **Model** | MX2013 |
| **Firmware** | v2a (04Oct2022) — Ecoforest base |
| **Platform** | Embedded Linux |
| **Exposed services** | SSH (22), HTTP/HTTPS (80/443/8000) |

## Findings Overview

| ID | Component | Issue | Severity |
|----|-----------|-------|----------|
| F1 | Dropbear SSH 2012.55 | CVE-2016-7406 — pre-auth remote code execution as root | Critical |
| F2 | thttpd 2.25b (2003) | CVE-2009-4491 (CVSS 3.1: 9.8) | High |
| F3 | Admin web panel | Predictable / derivable credentials | Medium |

See [advisory.md](advisory.md) for full technical detail.

## Methodology

1. **Reconnaissance** — network mapping and port scanning of the module's access point.
2. **Service fingerprinting** — identification of SSH and web server versions.
3. **API reverse engineering** — analysis of the local control API from its JavaScript client (single RPC-style CGI endpoint with an operation selector).
4. **Control verification** — tested whether the control API enforced authentication.
5. **CVE correlation** — mapping detected versions to known public vulnerabilities.

## Verified Controls (what was implemented correctly)

Rigorous assessment means documenting what works, not only what fails:

- **Control API enforces authentication** — read and control operations return `401` without credentials. The device cannot be controlled unauthenticated.
- **Cloud portal uses HTTPS** — remote-management login is encrypted.
- **Default WiFi key is strong** — long, alphanumeric, independent from the SSID/serial.

## Platform Scope (OSINT)

The tested device was a Bio+ MX2013 module. Public documentation indicates findings F2 and
F3 are **not specific to this model** but affect the broader Ecoforest product line. This
section relies **solely on public sources**; no active testing was performed against
third-party devices.

- **Credential scheme (F3) is platform-wide.** The official Home Assistant Ecoforest
  integration documents the standard login as: username = device serial number, password =
  first 8 characters of the WiFi key — the exact scheme identified here. Documented as
  compatible with "any Ecoforest device working with Ecoforest Home" (e.g. Cordoba Glass,
  firmware 30Abr19_v2z).
- **Web server & API (F2) shared across models.** An independent reverse-engineering project
  (jangelfdez/Ecoforest-Api) documents the same thttpd 2.25b (2003) and CGI control API on
  the "Bolonia" air stove, noting the same web application is used on other devices including
  air-water units.

**Conclusion:** public evidence suggests the vulnerable components are shared across the
product range. This is based on third-party documentation and is **not verified through
direct testing** of those models.

## Impact

Findings F1 and F2 require network access to the module. By default the device operates on
its own local access point; risk increases significantly if the device is exposed to the
Internet (e.g. port forwarding). The device runs base software 13–22 years old on hardware
that controls a combustion appliance.

## Responsible Disclosure

All testing was performed **exclusively on hardware owned by the researcher, on a private
network**, with no destructive actions and no interaction with third-party (vendor cloud)
infrastructure. Findings were coordinated through INCIBE-CERT. See [timeline.md](timeline.md).

## Disclaimer

This research is published for **educational and defensive purposes only**. It does not
include a functional exploit. Do not attempt to access devices you do not own or are not
explicitly authorized to test. The author assumes no liability for misuse.

## Author

**Pablo** — Junior Penetration Tester / Security Researcher · eJPT certified
[LinkedIn](https://www.linkedin.com/in/pablo-c-163260380)