# Four builds for the second office

All four machines share one **platform**: AMD socket **AM5**, the same motherboard (MSI PRO A620AM-B Evo, Micro-ATX, 2 × DDR5 slots, one M.2 slot, one PCIe x16 slot), the same CPU cooler, the same 1 TB NVMe SSD, the same 550 W power supply, the same case, keyboard and mouse. One spare board or power supply fits any of the four, and one person can service all of them. What differs is where the money goes: each machine gets more of the resource that runs out first for that person.

Prices: shop.kz (Belyi Veter), 23.11.2026, see [budget.csv](budget.csv) and [evidence/vendor-quotes/](evidence/vendor-quotes/).

**Shared part of every machine: 229,560 KZT**

| Part | Model | KZT |
|---|---|---|
| Motherboard | MSI PRO A620AM-B Evo, AM5, DDR5 | 45,590 |
| CPU cooler | ID-Cooling SE-904-XT ARGB, tower, AM5, up to 180 W | 8,800 |
| SSD | Samsung 990, 1 TB, M.2 NVMe PCIe 4.0 | 121,600 |
| Power supply | MSI MAG A550BN, 550 W, 80 PLUS Bronze | 22,590 |
| Case | Ocypus Gamma C60, Micro-ATX/ATX | 16,690 |
| Keyboard + mouse | Logitech MK270, wireless | 14,290 |

**Compatibility (checked as in Lab 3):** every CPU below is AM5 = the board's socket; every memory module is DDR5 = the board's slots; the SSD is M.2 2280 NVMe = the board's M.2 slot; the case takes Micro-ATX boards and an ATX power supply. Power: PCPartPicker estimates 144–170 W for the four builds (the Ryzen 7 7700 machines are the highest), so 550 W leaves room for a graphics card later (an RTX 5060 adds about 145 W). The PCPartPicker lists are in [evidence/pcpartpicker-links.md](evidence/pcpartpicker-links.md). PCPartPicker gave one real note: with a **Phoenix 2** CPU (the Ryzen 5 8500G) the board's PCIe x16 slot runs in **x4** mode. That does not matter for the accountant and sales machines, which will never get a graphics card, but it is the reason the designer — whose first upgrade is a graphics card — has a Ryzen 7 7700 (full x16) and not the cheaper 8500G. PCPartPicker also caught a mistake: for the **Ryzen 5 9600X** it warned that the processor **comes without a cooler**, and there was no cooler in my budget. Checking the shop pages, I saw that **all** my processors are sold as **OEM** (tray) versions — the 8500G and 7700 too — and an OEM processor never has a cooler, even though PCPartPicker lists the boxed version's "included cooler". So every machine got a separate **ID-Cooling SE-904-XT** (AM4/AM5 mount, rated for 180 W, far above the 65 W of these CPUs). For the designer I also switched from the 9600X to the Ryzen 7 7700 (+1,000 KZT): 8 cores instead of 6 for exports, the same CPU as the developer's (one kind of spare part), and no BIOS-version warning (PCPartPicker said the 9600X needs board BIOS 7E28vB2 or newer).

---

### Build 1 — accountant

**Load:** accounting software (1C) and Excel all day, 8 hours, plus a browser with the bank client and tax portal (5–10 tabs), e-mail, PDF invoices. One heavy thing at a time; long hours of reading numbers on screen.
**Bottleneck:** none of the hardware is under pressure — this load fits in 6–8 GB and uses one or two CPU cores. What actually limits the accountant is **reliability and the screen**: 1C needs Windows, the database must not be lost, and eight hours of small figures need a sharp, flicker-free IPS panel.
**Trade-off:** the cheapest AM5 processor with built-in graphics (Ryzen 5 8500G, no graphics card) and the minimum 16 GB as **one** module (single channel, about half the memory bandwidth). Acceptable for this person: spreadsheets are not bandwidth-bound, and the second slot stays free — a second 16 GB module later doubles memory and switches to dual channel without throwing anything away. The money saved went to the sales and developer machines.

| Part | Model | KZT |
|---|---|---|
| CPU | AMD Ryzen 5 8500G (6 cores, Radeon 740M graphics) | 63,990 |
| Memory | Netac Basic DDR5-5600 16 GB (1 × 16 GB) | 129,990 |
| Monitor | Acer EK241YP6bi 23.8", IPS, 1920×1080 | 49,990 |
| OS | Windows 11 Pro (OEM, DVD) | 102,990 |
| Shared part | see above | 229,560 |

**Total: 576,520 KZT** (budget.csv, rows 2–11)

### Build 2 — sales manager

**Load:** CRM in a browser, 30–40 tabs, Excel, WhatsApp/Telegram desktop, four hours of video calls a day, often all at once.
**Bottleneck:** **memory**, not the CPU. Chrome with forty tabs plus a video call and Excel needs 11–12 GB; with 16 GB Windows is already near the limit, and on less it starts swapping to the SSD in the middle of a customer call — the call stutters, not the spreadsheet.
**Trade-off:** the same cheap Ryzen 5 8500G as the accountant (the CPU idles in this role) and the same 24" Full HD screen; the money went into **32 GB as 2 × 16 GB** (dual channel), the most expensive part of this machine. No graphics card: video calls are decoded by the built-in graphics.

| Part | Model | KZT |
|---|---|---|
| CPU | AMD Ryzen 5 8500G | 63,990 |
| Memory | AMD Radeon R5 Gamer DDR5-5600 32 GB (2 × 16 GB) | 282,990 |
| Monitor | Acer EK241YP6bi 23.8", IPS, 1920×1080 | 49,990 |
| OS | Windows 11 Pro (OEM, DVD) | 102,990 |
| Shared part | see above | 229,560 |

**Total: 729,520 KZT** (budget.csv, rows 12–21)

### Build 3 — web developer

**Load:** code editor, 3–5 Docker containers (app, database, cache), sometimes a virtual machine, two or three browsers with developer tools, builds and test runs several times an hour, a terminal on screen next to the code.
**Bottleneck:** **memory first, then CPU cores.** Each container and the VM keep their memory reserved; Docker + a 4 GB VM + browsers + editor is 20–24 GB, so 16 GB would swap. Builds and tests use every core, so 8 cores finish them faster than 6.
**Trade-off:** the only 8-core processor of the four (Ryzen 7 7700, 65 W, so the shared 550 W supply and the shared tower cooler are enough) and 32 GB; a 27" 1440p screen, because code and a browser side by side is the developer's normal view. In return **no Windows licence**: the developer gets **Ubuntu 26.04 LTS**, which is free and legally licensed (GPL and other open-source licences, no per-seat fee), and it is the same OS the servers and Docker images run on. That saved 102,990 KZT. Accepted risk: if the office's accounting software or a client tool needs Windows, it runs in a VM or on another machine.

| Part | Model | KZT |
|---|---|---|
| CPU | AMD Ryzen 7 7700 (8 cores / 16 threads, Radeon graphics) | 100,990 |
| Memory | AMD Radeon R5 Gamer DDR5-5600 32 GB (2 × 16 GB) | 282,990 |
| Monitor | iiyama G-Master GB2770QSU-B5 27", IPS, 2560×1440 | 169,990 |
| OS | Ubuntu 26.04 LTS | 0 |
| Shared part | see above | 229,560 |

**Total: 783,530 KZT** (budget.csv, rows 22–31)

### Build 4 — designer

**Load:** Photoshop, Illustrator and Figma at the same time, large layered files (1–3 GB PSDs), hundreds of fonts and images, exports for print and web; occasionally a short video for social media.
**Bottleneck:** **memory and the screen.** Large PSDs with many layers live in RAM — on 16 GB Photoshop moves its "scratch" to the SSD and every brush stroke lags. The screen decides whether the colours the designer sees are the colours that get printed, so the monitor is a working tool here, not an accessory.
**Trade-off:** **no separate graphics card now.** An RTX 5060 costs 329,990 KZT on 23.11.2026 — 11 % of the whole budget — and the whole office does not fit into 3,000,000 KZT with it (3,306,080 KZT). For 2-D design the card speeds up only some filters and video export; memory and the monitor matter more, so the money went there: 32 GB, the 27" 2560×1440 IPS screen, and a Ryzen 7 7700 (8 cores for exports). Its built-in graphics drive the 1440p screen. The board's PCIe x16 slot and the 550 W supply are ready for a card — it is the first planned upgrade (README, question 1).

| Part | Model | KZT |
|---|---|---|
| CPU | AMD Ryzen 7 7700 (8 cores / 16 threads, Radeon graphics) | 100,990 |
| Memory | AMD Radeon R5 Gamer DDR5-5600 32 GB (2 × 16 GB) | 282,990 |
| Graphics card | — (built-in; RTX 5060 planned as the first upgrade) | 0 |
| Monitor | iiyama G-Master GB2770QSU-B5 27", IPS, 2560×1440 | 169,990 |
| OS | Windows 11 Pro (OEM, DVD) | 102,990 |
| Shared part | see above | 229,560 |

**Total: 886,520 KZT** (budget.csv, rows 32–41)

---

| Role | KZT |
|---|---|
| Accountant | 576,520 |
| Sales manager | 729,520 |
| Web developer | 783,530 |
| Designer | 886,520 |
| **Grand total** | **2,976,090** |
| Left of 3,000,000 | 23,910 (reserve for cables / a UPS / price changes) |
