# mfp-scout

Passive PMF/MFP/802.11w inspector for Wi-Fi packet captures.

`mfp-scout` is a small Bash utility that parses Wi-Fi captures with `tshark` and checks RSN capabilities related to Protected Management Frames.

It is meant to quickly answer:

- Does the AP advertise PMF/MFP/802.11w?
- Does the client request PMF?
- Is PMF required or optional?
- Is classic `aireplay-ng` deauthentication likely to work?

> Use only on networks and captures you are authorized to assess.

---

## Why this exists

In Wi-Fi assessments, deauthentication is often used to force clients to reconnect so authentication material can be observed. PMF/MFP/802.11w changes that behavior by protecting management frames.

If PMF is required, forged unprotected deauthentication/disassociation frames should be rejected.

`mfp-scout` gives a quick passive read from the capture before wasting time on unreliable deauth attempts.

---

## Requirements

- Bash
- `tshark`

Install `tshark`:

```bash
# Debian / Ubuntu / Kali
sudo apt install tshark

# Arch Linux
sudo pacman -S wireshark-cli
```

---

## Installation

```bash
git clone https://github.com/AleLopezDev/mfp-scout.git
cd mfp-scout
chmod +x mfp-scout
```

Optional system-wide install:

```bash
sudo cp mfp-scout /usr/local/bin/mfp-scout
```

---

## Usage

```bash
./mfp-scout capture.cap
```

Example:

```bash
./mfp-scout /tmp/wifi-corp-01.cap
```

For best results, capture while a client connects or reconnects:

```bash
airodump-ng wlan0mon -c <channel> --essid <ssid> -w mfp-check
```

---

## Example output

```text
[*] Reading capture: /tmp/wifi-corp-01.cap

------------------------------------------------------------
Access points
------------------------------------------------------------

AP: f0:9f:c2:71:22:15
  SSID: wifi-corp
  PMF:  PMF capable / optional
  Note: PMF is supported but not mandatory. A controlled test is needed to confirm deauth behavior.

------------------------------------------------------------
Clients
------------------------------------------------------------

Client: 64:32:a8:ba:6c:41
  Frame:  1288
  SSID:   wifi-corp
  AP:     f0:9f:c2:71:22:15
  PMF:    PMF not advertised
  Aireplay: Classic deauth likely
  Note:   PMF is not advertised in this frame. Classic deauth is more likely to work if radio conditions are good.

Client: 64:32:a8:a9:de:55
  Frame:  1942
  SSID:   wifi-regional-tablets
  AP:     f0:9f:c2:7a:33:28
  PMF:    PMF required
  Aireplay: No classic deauth expected
  Note:   Protected management frames are required. Forged unprotected deauth/disassoc frames should be rejected.

------------------------------------------------------------
Compact view
------------------------------------------------------------
CLIENT 1 64:32:a8:ba:6c:41 -> PMF not advertised (Classic deauth likely)
CLIENT 2 64:32:a8:a9:de:55 -> PMF required (No classic deauth expected)
```

---

## Interpretation

| Result | Meaning | Deauth expectation |
|---|---|---|
| PMF required | Protected management frames are mandatory | Classic forged deauth is unlikely to work |
| PMF capable / optional | PMF is supported but not mandatory | Deauth may or may not work |
| PMF not advertised | PMF is not advertised in the captured frame | Classic deauth is more likely to work |
| unknown | Capture is incomplete | Capture association/reassociation frames |

---

## Capture notes

The most useful client-side evidence comes from:

```text
Association Request
Reassociation Request
```

A quiet capture may only contain AP beacons. In that case, `mfp-scout` can tell you what the AP advertises, but not necessarily what the client negotiated.

To improve results:

```bash
airodump-ng wlan0mon -c <channel> --essid <ssid> -w mfp-check
```

Then wait for a client to connect or reconnect.

---

## Spanish description

`mfp-scout` es una herramienta ligera en Bash para analizar capturas Wi-Fi y revisar PMF/MFP/802.11w.

Sirve para estimar si una desautenticación clásica con `aireplay-ng` tiene pinta de funcionar o si el cliente/AP requiere tramas de gestión protegidas.

---

## License

MIT License.
