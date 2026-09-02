# prime-z490p-ops

Private operations repository for the local ASUS PRIME Z490-P workstation.

## Machine profile

| Component | Value |
|---|---|
| Hostname | `DESKTOP-3T2R9GC` |
| Mainboard | ASUS PRIME Z490-P Rev 1.xx |
| CPU | Intel Core i7-10700K, 8 cores / 16 threads |
| GPU | NVIDIA GeForce RTX 3060; Intel UHD Graphics 630 |
| Memory | 16 GB |
| Operating system | Windows 10 Pro 10.0.19045 |

Hardware serial numbers, UUIDs, credentials, tokens, and other secrets must not be committed.

## Task workflow

GitHub Issues is the source of truth for work on this PC.

1. Create an issue with the **Local PC task** form.
2. Apply one `area:*` label and one `priority:*` label.
3. Record the intended change, risk, validation, and rollback before modifying important settings.
4. Put reusable procedures in `runbooks/` and collected evidence in `docs/`.
5. Close the issue only after its acceptance criteria pass.

## Repository layout

- `.github/ISSUE_TEMPLATE/` — structured local-PC task intake.
- `docs/` — investigation reports and durable evidence.
- `runbooks/` — safe, repeatable operating procedures.

## Current investigation

The initial tracked task classifies unexpected power-on after a clean S5 shutdown while preserving Wake-on-LAN by Magic Packet. See [the WOL report](docs/report-windows-wol.md).
