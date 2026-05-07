# Zabbix Template — Kyocera ECOSYS MA5500ifx (SNMPv3)

Monitoring template for the **Kyocera ECOSYS MA5500ifx** monochrome MFU via SNMPv3 for Zabbix 7.4.

Developed and verified against a live device using a full SNMP OID dump.
All counter and supply values were cross-checked against the device's built-in web interface.

---

## Device

| Field | Value |
|-------|-------|
| Model | Kyocera ECOSYS MA5500ifx (A4 mono MFU) |
| Tested firmware | `C0V_S000.005.054` |
| Serial number | `WDC5756001` |
| Host | `kyocera.local` / `10.100.113.51` |
| Template file | `zbx_template_kyocera_ma5500ifx.yaml` |
| Zabbix version | 7.4+ |
| SNMP version | v3 (authPriv, SHA + AES) |
| Vendor OID | `1.3.6.1.4.1.1347.41` |

---

## SNMPv3 Configuration

| Parameter | Value |
|-----------|-------|
| Security name | `zabbixUser` |
| Auth protocol | SHA |
| Priv protocol | AES |

---

## Macros

| Macro | Default | Description |
|-------|---------|-------------|
| `{$SNMP_USER}` | `zabbixUser` | SNMPv3 username |
| `{$SNMP_AUTH_PASS}` | *(secret)* | Authentication password |
| `{$SNMP_PRIV_PASS}` | *(secret)* | Privacy (encryption) password |
| `{$TONER_LOW_WARN}` | `20` | Toner warning threshold (%) |
| `{$TONER_LOW_CRIT}` | `10` | Toner critical threshold (%) |
| `{$PAPER_LOW_WARN}` | `20` | Cassette paper low threshold (%) |

---

## Items

### System Information

| Name | Key | OID | Interval |
|------|-----|-----|----------|
| Model Name | `printer.model` | `1.3.6.1.2.1.43.5.1.1.16.1` | 1h |
| Serial Number | `printer.serial` | `1.3.6.1.2.1.43.5.1.1.17.1` | 1h |
| Firmware Version (Main) | `printer.firmware.main` | `1.3.6.1.4.1.1347.43.5.4.1.5.1.1` | 1h |
| System Uptime | `printer.uptime` | `1.3.6.1.2.1.1.3.0` | 10m |
| CPU Model | `system.cpu.model` | `1.3.6.1.4.1.1347.43.5.4.1.2.1.1` | 1h |
| CPU Frequency | `system.cpu.freq` | `1.3.6.1.4.1.1347.43.5.4.1.3.1.1` | 1h |
| RAM Total | `system.ram.total` | `1.3.6.1.4.1.1347.43.5.1.1.4.1` | 1h |
| RAM Free | `system.ram.free` | `1.3.6.1.4.1.1347.43.5.1.1.10.1` | 5m |
| RAM Used | `system.ram.used` | `ram.total − ram.free` *(CALCULATED)* | 5m |
| First Use Date | `printer.first.use.date` | `1.3.6.1.4.1.1347.42.2.6.3.0.0` | 1h |

> **CPU Frequency** is stored as a plain integer (e.g. `1400`) with no units assigned to prevent Zabbix SI auto-scaling from displaying `1.4 KMHz`. The description states the value is in MHz.

> **RAM** OIDs return values in KB. A ×1024 multiplier is applied to convert to bytes, allowing Zabbix to display values in MB/GB automatically.

### Status

| Name | Key | OID | Interval |
|------|-----|-----|----------|
| Printer Status | `printer.status` | `1.3.6.1.2.1.25.3.5.1.1.1` | 1m |
| Printer Error State | `printer.error.state` | `1.3.6.1.2.1.25.3.5.1.2.1` | 1m |

`printer.status` uses the `hrPrinterStatus` value map: `3=Idle`, `4=Printing`, `5=Warmup`.

`printer.error.state` receives a raw hex bitmap and decodes it to a human-readable string via JavaScript preprocessing. Returns `"OK"` when no errors are present.

### Network

| Name | Key | OID | Interval |
|------|-----|-----|----------|
| MAC Address | `net.if.mac` | `1.3.6.1.2.1.2.2.1.6.1` | 1h |
| IP Address | `net.if.ip` | `1.3.6.1.2.1.4.20.1.1.10.100.113.51` | 10m |

### Supplies — Toner

| Name | Key | OID / Formula | Interval |
|------|-----|---------------|----------|
| Toner TK-3430S: Level (raw) | `toner.black.raw` | `1.3.6.1.2.1.43.11.1.1.9.1.1` | 15m |
| Toner TK-3430S: Level (%) | `toner.black.pct` | `raw / 10000 × 100` *(DEPENDENT + JS)* | — |
| Waste Toner Box: Status | `toner.waste.status` | `1.3.6.1.2.1.43.11.1.1.9.1.2` | 15m |

Toner percentage uses the standard Printer MIB (RFC 3805): `prtMarkerSuppliesLevel` / `prtMarkerSuppliesMaxCapacity`. Confirmed value: `9800 / 10000 = 98%`.

Waste toner box status decoding: `−3 = OK`, `0 = Full (replace)`. The maximum capacity OID returns `−2` (unknown) for this consumable — percentage is not available, status only.

### Supplies — Maintenance Units

| Name | Key | OID | Interval |
|------|-----|-----|----------|
| Imaging Unit: Status | `imaging.unit.status` | `1.3.6.1.4.1.1347.43.24.1.1.5.1` | 15m |
| Document Processor: Status | `doc.processor.status` | `1.3.6.1.4.1.1347.43.24.1.1.5.2` | 15m |

Status decoding for Imaging Unit: `128 (0x80) = OK`, `64 = Near End`, `0 = Replace`.
Status decoding for Document Processor: `1 = OK`, `0 = Replace`.

> The device does not expose remaining life percentage for these units via SNMP. The web interface shows `100%` but this value is not available in the OID tree. Only status codes are monitored.

### Paper Trays

| Name | Key | OID / Formula | Interval | Notes |
|------|-----|---------------|----------|-------|
| Cassette 1: Level (raw) | `paper.cassette1.raw` | `1.3.6.1.2.1.43.8.2.1.10.1.2` | 5m | Sheets, max=500 |
| Cassette 1: Level (%) | `paper.cassette1.pct` | `raw / 500 × 100` *(DEPENDENT + JS)* | — | 0–100% |
| MP Tray: Status (raw) | `paper.mptray.raw` | `1.3.6.1.2.1.43.8.2.1.11.1.1` | 5m | 0=present, 19=empty |
| MP Tray: Status | `paper.mptray.status` | `raw → text` *(DEPENDENT + JS)* | — | "Paper present" / "Empty" |

**Cassette 1** supports sheet counting: `prtInputCurrentLevel` (field `.10`) returns the exact number of sheets remaining. Negative values (e.g. `−3`) are clamped to 0 via JS preprocessing.

**MP Tray** does not support sheet counting. Field `.10.1.1` returns only `0` (empty) or `−3` (paper present but quantity unknown). Therefore, the status field `.11.1.1` is used instead: `0 = Paper present`, `19 = Empty`.

### Print Counters

| Name | Key | OID / Formula | Interval |
|------|-----|---------------|----------|
| Total Pages Printed | `print.pages.total` | `1.3.6.1.2.1.43.10.2.1.4.1.1` | 5m |
| B&W Pages: Print | `print.pages.bw.print` | `1.3.6.1.4.1.1347.42.3.1.1.1.1.1` | 5m |
| B&W Pages: Copy | `print.pages.bw.copy` | `1.3.6.1.4.1.1347.42.3.1.1.1.1.2` | 5m |
| B&W Pages: FAX | `print.pages.bw.fax` | `1.3.6.1.4.1.1347.42.3.1.1.1.1.3` | 5m |
| B&W Pages: Total | `print.pages.bw.total` | `bw.print + bw.copy + bw.fax` *(CALCULATED)* | 5m |
| Pages: 1-Sided (Simplex) | `print.pages.simplex.total` | `1.3.6.1.4.1.1347.42.2.5.1.1.1.1` | 5m |
| Pages: 2-Sided (Duplex) | `print.pages.duplex.total` | `1.3.6.1.4.1.1347.42.2.5.1.1.2.1` | 5m |
| Pages: A4 Printed | `print.pages.a4` | `1.3.6.1.4.1.1347.42.3.1.6.1.1.1.1` | 5m |

Simplex/Duplex counters are sourced from `1347.42.2.5` — confirmed values: simplex=78, duplex=50, total=128.

### Scan Counters

| Name | Key | OID | Interval | Notes |
|------|-----|-----|----------|-------|
| Scanned Pages: Total | `scan.pages.total` | `1.3.6.1.4.1.1347.46.10.1.1.5.3` | 5m | Confirmed: 14 ✅ |
| Scanned Pages: Other | `scan.pages.other` | `1.3.6.1.4.1.1347.46.10.1.1.7.3` | 5m | Confirmed: 3 ✅ |

> The **Copy scan** count does not have a dedicated OID. It can be derived as `Total − Other − FAX`.
> FAX scan is 0 on this device and is not exposed as a separate item.

---

## Triggers

| Name | Item | Severity | Notes |
|------|------|----------|-------|
| Device is not responding via SNMP | `printer.status` | AVERAGE | No data for 10 minutes |
| Printer status unknown | `printer.status` | WARNING | hrPrinterStatus = other(1) or unknown(2) |
| Printer error: `{ITEM.LASTVALUE}` | `printer.error.state` | HIGH | Error bitmap non-zero; manual close |
| Device has been restarted (uptime < 10 min) | `printer.uptime` | WARNING | Manual close |
| Firmware version changed | `printer.firmware.main` | INFO | Manual close |
| MAC address changed | `net.if.mac` | HIGH | Unexpected hardware change; manual close |
| IP address changed | `net.if.ip` | WARNING | Manual close |
| Toner level low | `toner.black.pct` | WARNING | Level ≤ `{$TONER_LOW_WARN}`% |
| Toner critically low | `toner.black.pct` | HIGH | Level ≤ `{$TONER_LOW_CRIT}`% |
| Waste toner box needs replacement | `toner.waste.status` | HIGH | Status ≠ "OK"; manual close |
| Imaging unit requires attention | `imaging.unit.status` | WARNING | Status ≠ "OK"; manual close |
| Cassette 1: paper low | `paper.cassette1.pct` | WARNING | Level ≤ `{$PAPER_LOW_WARN}`% |
| Cassette 1: paper empty | `paper.cassette1.pct` | AVERAGE | Level = 0% |
| MP Tray: no paper | `paper.mptray.status` | INFO | Status = "Empty" |

---

## Value Maps

| Name | Mappings |
|------|----------|
| `hrPrinterStatus` | 1=Other, 2=Unknown, 3=Idle, 4=Printing, 5=Warmup |
| `kyoceraPaperLevel` | 1=Full, 2=High, 3=Medium, 4=Low, 5=Empty, 19=No Paper |

---

## Known Limitations

| Area | Reason |
|------|--------|
| **Color printing** | The MA5500ifx is a monochrome device. No color counters exist. |
| **Scan counters by function (Copy / FAX)** | The web interface shows a breakdown of Copy=11 / FAX=0 / Other=3, but these session-level counters are not individually available via SNMP. Only Total and Other are accessible as separate OIDs. Copy count can be derived as `Total − Other − FAX`. |
| **Imaging Unit / Document Processor life (%)** | Kyocera does not expose a remaining-life percentage for these units via SNMP. The OID `1347.43.24.1.1.6` contains the maximum page count (500,000 / 200,000), but the current page count is not available. Only a status code is monitored. |
| **Additional paper cassettes** | The template assumes a two-tray configuration (MP Tray + Cassette 1). If additional cassettes are installed, extra items with index `.1.3`, `.1.4`, etc. must be added manually. |
| **MP Tray paper level (%)** | The MP Tray cannot count sheets. `prtInputCurrentLevel` returns only `0` (empty) or `−3` (paper present, quantity unknown). A percentage is therefore not available — presence/absence is monitored instead. |
| **FAX counters in detail** | FAX session logs, send/receive history, and error counts are not monitored. |
| **Print job queue** | Active job status and queue monitoring are not supported via SNMP on this model. |
| **Fuser temperature and internal sensors** | Not exposed via SNMP. |

---

## MIBs Used

| MIB | OID Prefix | Usage |
|-----|-----------|-------|
| Printer MIB (RFC 3805) | `1.3.6.1.2.1.43.*` | Toner, paper trays, page counters, status |
| HOST-RESOURCES-MIB | `1.3.6.1.2.1.25.*` | Printer status, error state |
| IF-MIB | `1.3.6.1.2.1.2.*` | MAC address |
| IP-MIB | `1.3.6.1.2.1.4.*` | IP address |
| Kyocera private MIB | `1.3.6.1.4.1.1347.*` | CPU, RAM, firmware, toner, imaging unit, simplex/duplex, scan counters, first use date |
