# Rogue Access Point Detection & Monitor Mode Setup

**Category:** Wireless Security | **Tools:** Linux, aircrack-ng suite, custom bash scripting

## Problem
Open/unmanaged wireless networks are vulnerable to rogue access points (evil
twins) impersonating legitimate SSIDs to intercept client traffic. I wanted
to build a lightweight detection workflow that could run on commodity
hardware, and to understand the driver/chipset constraints that make this
harder in practice than it looks on paper.

## Approach
1. Configured a wireless adapter into monitor mode to passively capture
   beacon frames (`airmon-ng`, later scripted for repeatability).
2. Hit early blockers with Realtek chipsets — several don't expose full
   monitor-mode support without patched drivers, and regulatory domain
   settings silently restrict which channels are scannable depending on
   the country code set in the OS.
3. Wrote a script to log SSID, BSSID, channel, and signal strength for all
   visible APs at fixed intervals, then flag any SSID seen broadcasting
   from more than one BSSID within the capture window — a common rogue-AP
   signature.
4. Validated the detector by spinning up a second AP broadcasting the same
   SSID as a "legitimate" one and confirming the script flagged it.

## Findings
- Realtek RTL8811AU/RTL88x2BU chipsets required out-of-tree drivers
  (`rtl88xxau-aircrack`) to get reliable monitor mode — stock drivers
  silently dropped injection support.
- Regulatory domain (`iw reg set`) affected which channels the adapter
  would even listen on, meaning a misconfigured region could make the
  detector blind to APs on channels 12–14 without any visible error.
- Duplicate-SSID detection alone gives false positives on legitimate
  mesh/repeater setups — needed to also compare BSSID vendor OUI ranges
  and beacon interval fingerprints to reduce noise.

## Reflection
The gap between "monitor mode should just work" and actually getting
clean packet capture across chipsets was the real lesson — most guides
skip driver/regulatory issues entirely. Next step: extend this into a
persistent background service with alerting, and test against WPA3
transition-mode APs specifically.