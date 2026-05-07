# Zabbix Template — Epson L18050 (SNMPv3)

Monitoring template for the **Epson L18050** EcoTank inkjet printer via SNMPv3 for Zabbix 7.4.

Developed and verified against a live device using a full SNMP OID dump.
All counter values were cross-checked against the printer's built-in web interface.

---

## Device

| Field | Value |
|-------|-------|
| Model | Epson L18050 (EcoTank A3+ inkjet) |
| Template file | `zbx_template_epson_l18050.yaml` |
| Zabbix version | 7.4+ |
| SNMP version | v3 (authPriv, SHA + AES) |
| Vendor OID | `1.3.6.1.4.1.1248` |

---

## SNMP Configuration

I highly recommend using snmpv3 in production and create a custom complex context, authentication and authorization name.

---

## Items

### System Information

| Name | Key | OID | Interval |
|------|-----|-----|----------|
| Printer Model Name | `printer.model` | `1.3.6.1.2.1.43.5.1.1.16.1` | 1h |
| Serial Number | `printer.serial` | `1.3.6.1.4.1.1248.1.2.2.2.1.1.2.1.2` | 1h |
| Firmware Version | `printer.firmware` | `1.3.6.1.4.1.1248.1.2.2.2.1.1.2.1.1` | 1h |
| System Uptime | `printer.uptime` | `1.3.6.1.2.1.1.3.0` | 10m |
| Total RAM | `system.ram.total` | `1.3.6.1.2.1.25.2.2.0` | 1h |
| Running Processes | `system.processes` | `1.3.6.1.2.1.25.1.6.0` | 5m |

### Status

| Name | Key | OID | Interval |
|------|-----|-----|----------|
| Printer Status | `printer.status` | `1.3.6.1.2.1.25.3.5.1.1.1` | 1m |
| Printer Error State | `printer.error.state` | `1.3.6.1.2.1.25.3.5.1.2.1` | 1m |

`printer.status` uses the `hrPrinterStatus` value map: `3=Idle`, `4=Printing`, `5=Warmup`.

`printer.error.state` receives a raw hex bitmap and decodes it into a human-readable string via JavaScript preprocessing (e.g. `"Paper Jam, No Toner"`). Returns `"OK"` when no errors are present.

### Network

| Name | Key | OID | Interval |
|------|-----|-----|----------|
| MAC Address | `net.if.mac` | `1.3.6.1.2.1.2.2.1.6.1` | 1h |
| IP Address | `net.if.ip` | `1.3.6.1.2.1.4.20.1.1.10.100.113.68` | 10m |

### Print Counters — Totals

| Name | Key | OID / Formula | Interval |
|------|-----|---------------|----------|
| Total Pages Printed | `print.pages.total` | `1.3.6.1.2.1.43.10.2.1.4.1.1` | 5m |
| Total B&W Pages | `print.pages.bw` | `1.3.6.1.4.1.1248.1.2.2.27.1.1.3.1.1` | 5m |
| Total Color Pages | `print.pages.color` | `1.3.6.1.4.1.1248.1.2.2.27.1.1.4.1.1` | 5m |
| Pages: 2-Sided (Duplex) | `print.pages.duplex` | `1.3.6.1.4.1.1248.1.2.2.27.1.1.30.1.1` | 5m |
| Pages: 1-Sided (Simplex) | `print.pages.simplex` | `total − duplex` *(CALCULATED)* | 5m |
| First Printing Date | `print.first.date` | `1.3.6.1.4.1.1248.1.2.2.27.1.1.13.1.1` | 1h |

> `print.first.date`: raw value is an 8-byte hex timestamp decoded to `YYYY-MM-DD` via JavaScript preprocessing.

### Print Counters — By Paper Size

| Name | Key | OID | Notes |
|------|-----|-----|-------|
| A3 Total | `print.pages.a3.total` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.4.1.1` | B&W + Color |
| A3 Color | `print.pages.a3.color` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.5.1.1` | |
| A3 B&W | `print.pages.a3.bw` | `a3.total − a3.color` *(CALCULATED)* | |
| A4 Total | `print.pages.a4.total` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.4.1.2` | |
| A4 Color | `print.pages.a4.color` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.5.1.2` | |
| A4 B&W | `print.pages.a4.bw` | `a4.total − a4.color` *(CALCULATED)* | |
| A5 Total | `print.pages.a5.total` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.4.1.3` | |
| A6 Total | `print.pages.a6.total` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.4.1.4` | |
| B4 Total | `print.pages.b4.total` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.4.1.5` | |
| B5 Total | `print.pages.b5.total` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.4.1.6` | |
| Envelope Total | `print.pages.envelope.total` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.4.1.7` | |
| Other Total | `print.pages.other.total` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.4.1.8` | |
| Other Color | `print.pages.other.color` | `1.3.6.1.4.1.1248.1.2.2.6.1.1.5.1.8` | |

### Print Counters — By Source

| Name | Key | OID / Formula |
|------|-----|---------------|
| Computer/Mobile Total | `print.pages.computer.total` | `1.3.6.1.4.1.1248.1.2.2.27.6.1.4.1.1.1` |
| Computer/Mobile Color | `print.pages.computer.color` | `1.3.6.1.4.1.1248.1.2.2.27.6.1.5.1.1.1` |
| Computer/Mobile B&W | `print.pages.computer.bw` | `computer.total − computer.color` *(CALCULATED)* |
| Memory/Other Total | `print.pages.memory.total` | `1.3.6.1.4.1.1248.1.2.2.27.6.1.4.1.1.2` |
| Memory/Other Color | `print.pages.memory.color` | `1.3.6.1.4.1.1248.1.2.2.27.6.1.5.1.1.2` |
| Memory/Other B&W | `print.pages.memory.bw` | `memory.total − memory.color` *(CALCULATED)* |

### Print Counters — By Print Language

| Name | Key | OID |
|------|-----|-----|
| ESC/P-R Pages | `print.pages.lang.escp` | `1.3.6.1.4.1.1248.1.2.2.49.2.1.5.1.1` |
| Other Language Pages | `print.pages.lang.other` | `1.3.6.1.4.1.1248.1.2.2.49.2.1.5.1.2` |

---

## Triggers

| Name | Item | Severity | Notes |
|------|------|----------|-------|
| Printer is not responding via SNMP | `printer.status` | AVERAGE | No data for 10 minutes |
| Printer status unknown | `printer.status` | WARNING | hrPrinterStatus = other(1) or unknown(2) |
| Printer error: `{ITEM.LASTVALUE}` | `printer.error.state` | HIGH | Error bitmap is non-zero; manual close |
| Printer has been restarted (uptime < 10 min) | `printer.uptime` | WARNING | Manual close |
| Firmware version changed | `printer.firmware` | INFO | Manual close |
| MAC address changed | `net.if.mac` | HIGH | Unexpected hardware change; manual close |
| IP address changed | `net.if.ip` | WARNING | Manual close |

---

## Value Maps

| Name | Mappings |
|------|----------|
| `hrPrinterStatus` | 1=Other, 2=Unknown, 3=Idle, 4=Printing, 5=Warmup |

---

## Known Limitations

| Area | Reason |
|------|--------|
| **Ink levels** | Intentionally excluded. EcoTank printers do not physically measure ink volume in their tanks. The OID `1248.1.8.2.14.7.1.2.2.1.N` returns identical hex values for all six ink colors regardless of actual fill level — the data is unreliable and cannot be used for monitoring. |
| **Copy / Scan / Fax counters** | The L18050 is a printer only — it has no MFU functions. These counters do not exist on this device. |
| **Individual ink colors** | Cyan, Magenta, Yellow, Light Cyan, Light Magenta counters are excluded for the same reason as ink levels. |
| **Temperature and internal sensors** | Not exposed via SNMP on this model. |
| **Print job queue** | Job status and queue monitoring are not supported via SNMP. |
| **Duplex detail by paper size** | Duplex counters by paper format are present in the OID tree (`.8.1.N`) but always return 0 on this model — the L18050 does not support automatic duplex printing. |

---

## MIBs Used

| MIB | OID Prefix | Usage |
|-----|-----------|-------|
| Printer MIB (RFC 3805) | `1.3.6.1.2.1.43.*` | Page counters, status |
| HOST-RESOURCES-MIB | `1.3.6.1.2.1.25.*` | Printer status, RAM, processes |
| IF-MIB | `1.3.6.1.2.1.2.*` | MAC address |
| IP-MIB | `1.3.6.1.2.1.4.*` | IP address |
| Epson private MIB | `1.3.6.1.4.1.1248.*` | Counters by size, source, language; firmware; error state |
