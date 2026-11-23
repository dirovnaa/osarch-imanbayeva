# Ticket 2 — Kitting out the new office

Author: Imanbayeva Daiana · individual work · Date: 23.11.2026 · Prices: shop.kz (Belyi Veter, delivers across Kazakhstan), taken 23.11.2026

## 1. What was asked

> **From:** Daniyar K. (Director)
> **Subject:** computers for the new office
>
> We are opening the second office and need four computers — accountant, sales, web developer, designer. Budget 3,000,000 tenge for everything. Everyone says we should just buy the same four machines, is that right? Make sure they last a few years. Monitors included.

## 2. What was actually needed

**Four identical machines is the wrong answer.** The four people do completely different work, so a different resource runs out first for each of them (see [builds.md](builds.md)):

- the **accountant** needs reliability, Windows for 1C and a readable screen — the hardware is barely loaded;
- the **sales manager** runs out of **memory** (40 browser tabs + video calls);
- the **web developer** runs out of **memory and CPU cores** (Docker containers, a VM, builds);
- the **designer** runs out of **memory** (large PSD files) and needs a **good screen** for colour.

If all four are identical, there are only two ways to do it, and both waste money. Make them all as strong as the designer and developer need, and the accountant gets 32 GB and an 8-core CPU they will never use, while the total goes over 3,000,000 KZT. Make them all as cheap as the accountant needs, and the sales manager's calls stutter and the developer's builds swap from the first week. The director's colleagues are right about one thing, though: **the same platform** makes the office easier to run. So all four share the same socket (AM5), motherboard, SSD, power supply and case. Spare parts fit any machine and one technician learns one design. Only the CPU, memory, screen and OS differ.

**What the request left out, what I would ask Daniyar, and what I assumed instead**

| Left out | What I would ask | What I assumed |
|---|---|---|
| What each person runs all day | "Which programs does each person use, how many hours, and how many at once?" | Accountant: 1C + Excel + bank client. Sales: browser CRM, 30–40 tabs, 4 h of video calls. Developer: editor, Docker, a VM, browsers. Designer: Photoshop/Illustrator/Figma, mostly 2-D print and web. |
| What "everything" in the budget covers | "Is 3,000,000 KZT for the computers only, or also the OS, monitors, keyboards, mice, cables, a UPS, installation?" | Computers + monitors (he said so) + **operating systems** + **keyboard and mouse** — a computer cannot be used without them. Cables, a UPS, desks, the network and MS Office are **not** included; 23,910 KZT is left as a reserve for cables. |
| Licensed software | "Does the office already have Windows / Office / Adobe licences?" | No licences exist: Windows 11 Pro is bought for three machines. Adobe/Office subscriptions are monthly costs, not part of this hardware budget. |
| Whether any work needs Windows | "Does any accounting or client software require Windows?" | 1C and the bank client need Windows (accountant), Adobe needs Windows (designer). The developer gets Ubuntu Linux — free and legally licensed — because his servers and Docker images run Linux. |
| "Last a few years" — how many | "How many years before you want to replace them?" | **4–5 years.** That decided the platform: AM5 + DDR5. AMD supports AM5 with new CPUs (Ryzen 7000, 8000, 9000 so far), and every board has a free memory slot or a PCIe x16 slot for later. An older AM4/DDR4 platform would be cheaper today but is at the end of its life. |
| The designer's kind of work | "Print/web design only, or also video and 3-D?" | Mostly 2-D. This is why the designer has no graphics card yet (see the trade-off in builds.md). |
| Laptops or desktops | "Does anyone need to work from home or travel?" | Desktops — "computers" and "monitors included" point to fixed desks. |

## 3. What I did

1. **Roles → loads → bottlenecks.** For each person I wrote the load, the resource that runs out first, and the trade-off, *before* choosing parts ([builds.md](builds.md)).
2. **One platform, money to the bottleneck.** I chose a shared AM5 base (MSI PRO A620AM-B Evo, ID-Cooling SE-904-XT cooler, Samsung 990 1 TB NVMe, MSI MAG A550BN 550 W Bronze, Ocypus Gamma C60 case, Logitech MK270) = 229,560 KZT per machine. On top of that, each role got its own CPU, memory and screen. Every machine meets the hard requirements: **≥ 16 GB memory, a 1 TB SSD, a licensed OS, a monitor.**
3. **Compatibility** (as in Lab 3): all CPUs are AM5 = the board's socket; all memory is DDR5 = the board's slots; the SSD is M.2 2280 NVMe = the board's M.2 slot; the case takes Micro-ATX. Power: PCPartPicker estimates 144–170 W against a 550 W supply. I checked it in PCPartPicker as well ([evidence/pcpartpicker-links.md](evidence/pcpartpicker-links.md)); PCPartPicker shows US prices, so I used it only for compatibility and wattage, not for prices.
4. **Prices** from one vendor that delivers to all of Kazakhstan, all taken on **23.11.2026**, one dated screenshot per part ([evidence/vendor-quotes/](evidence/vendor-quotes/)), every row in [budget.csv](budget.csv) with link and date.
5. **The budget did not fit at first** — 3,333,480 KZT. DDR5 memory and SSDs are very expensive in Kazakhstan right now: 32 GB of DDR5 costs 282,990 KZT, almost as much as a graphics card. I removed the designer's RTX 5060 (329,990 KZT) and replaced a 32,590 KZT case with a 16,690 KZT one. PCPartPicker then warned that the Ryzen 5 9600X I had planned for the designer comes without a cooler. That made me check the shop pages: **all** my CPUs are OEM (tray) versions with no cooler, so I added an 8,800 KZT tower cooler to every machine (+35,200 KZT) and gave the designer the Ryzen 7 7700 instead (8 cores, +1,000 KZT). The reasoning is in builds.md, Build 4.

**Budget** ([budget.csv](budget.csv), rows 42–46)

| Role | CPU | Memory | Screen | OS | KZT |
|---|---|---|---|---|---|
| Accountant | Ryzen 5 8500G | 16 GB (1 × 16) | 23.8" IPS FHD | Windows 11 Pro | 576,520 |
| Sales manager | Ryzen 5 8500G | 32 GB (2 × 16) | 23.8" IPS FHD | Windows 11 Pro | 729,520 |
| Web developer | Ryzen 7 7700 | 32 GB (2 × 16) | 27" IPS 1440p | Ubuntu 26.04 LTS | 783,530 |
| Designer | Ryzen 7 7700 | 32 GB (2 × 16) | 27" IPS 1440p | Windows 11 Pro | 886,520 |
| **Total** | | | | | **2,976,090** (≤ 3,000,000; 23,910 left) |

### The director's three questions

**1. Which machine will you upgrade first, eighteen months from now, and why?**
The **designer's**: add a **graphics card** (an RTX 5060-class card, ~330,000 KZT today). It is the one trade-off I made on a real bottleneck. GPU-accelerated Photoshop filters, AI tools and video export run on the CPU and built-in graphics now. The machine is ready for it: the board has a free PCIe x16 slot, and the 550 W supply covers the card (~145 W) with room to spare. Second in line is the accountant's second 16 GB module (the free slot, and dual channel).

**2. Pre-built machines from a vendor — what does the company gain and lose?**
Pre-built prices, shop.kz, 23.11.2026 ([prebuilt.csv](prebuilt.csv), [evidence/prebuilt-quotes/](evidence/prebuilt-quotes/)):

| Instead of | Pre-built | KZT | My build without monitor and keyboard/mouse |
|---|---|---|---|
| Accountant, sales | Lenovo ThinkCentre Neo 50t Gen 6 (Core Ultra 7, **16 GB**) | 859,990 each | accountant 512,240 · sales 665,240 |
| Developer, designer | ASUS TUF Gaming T700MF (Core Ultra 5, **16 GB**) | 939,990 each | developer 599,250 · designer 702,240 |
| **Four machines** | 2 × Lenovo + 2 × ASUS | **3,599,960** | **2,478,970** |

Four pre-built computers alone cost about **3.6 million KZT**, more than the whole budget **before** monitors, and both models have **16 GB**, half of what sales, the developer and the designer need. *Gained:* one warranty from one company for the whole machine, a tested system, often a Windows licence already installed, no assembly time, vendor support (Lenovo's business line has on-site service options). *Lost:* about 1.12 million KZT, the choice of parts (the money goes into an Intel CPU and gaming parts instead of memory), and part of the upgrade path, because pre-built cases, boards and power supplies are often proprietary or small. For this budget, building is the only way to give each person what their load needs. The time saved by pre-built is real, so building only makes sense if someone in the office can assemble four machines and look after them.

**3. One of these four people will complain about performance within a month. Who, and about what?**
The **designer**, the first time they export a video for social media or runs a heavy GPU filter or "Generative fill" in Photoshop. Without a graphics card that falls back to the CPU and the small built-in graphics, so a one-minute export or a filter will take noticeably longer than on the machine they are used to. This is exactly where my own trade-off hurts, and it is why the graphics card is upgrade number one. (Runner-up: the accountant, if they keep the bank client, 1C and 20 tabs open at once — 16 GB in single channel is the minimum.)

## 4. Evidence

- [builds.md](builds.md) — the four builds: load, bottleneck, trade-off, parts, total; compatibility and power
- [budget.csv](budget.csv) — every part per role: role, component, model, price_kzt, source (vendor link), date; totals per role and the grand total 2,976,090 KZT (UTF-8)
- [prebuilt.csv](prebuilt.csv) — two pre-built computers with prices, links and date, for question 2
- [evidence/vendor-quotes/](evidence/vendor-quotes/) — one screenshot per price, with the shop.kz address bar and the date 23.11.2026 in the taskbar: `shared-*.png` (motherboard, CPU cooler, SSD, power supply, case, keyboard+mouse), `accountant-sales-cpu.png`, `accountant-memory.png`, `memory-32gb.png`, `webdev-designer-cpu.png`, `monitor-24.png`, `monitor-27.png`, `windows-11-pro.png`
- [evidence/prebuilt-quotes/](evidence/prebuilt-quotes/) — the two pre-built computers, dated
- [evidence/sold-out/](evidence/sold-out/) — the Ryzen 5 7600 and the MSI Spatium M450 SSD showed "out of stock" on 23.11.2026; this is why the SSD is a Samsung 990 and the 6-core designer CPU was not a 7600
- [evidence/not-used/](evidence/not-used/) — parts priced and then dropped: the RTX 5060 (329,990 KZT, see Build 4) and the Ryzen 5 9600X, which PCPartPicker flagged as having **no cooler in the box**
- [evidence/pcpartpicker-links.md](evidence/pcpartpicker-links.md) — one PCPartPicker list per build, used for compatibility and estimated wattage

## 5. What I would do differently

1. **Compare at least two vendors before fixing the design.** I took every price from shop.kz. The RTX 5060 cost 329,990 KZT there, while other Kazakhstan shops listed the same card from about 210,000 KZT. With that price the designer could have kept a graphics card and the whole office would still have been close to the budget. I would check Technodom, Alser and Kaspi for the three most expensive parts (memory, GPU, SSD) first.
2. **Start from the memory price, not the CPU.** I planned the builds around processors and only found out at the end that two sticks of DDR5 cost more than the processor and the board together. In 2026 memory is the part that decides the budget, so I would price it first.
3. **Check stock before planning.** Two parts I had planned (Ryzen 5 7600, the cheap MSI SSD) were sold out, and I had to change the plan in the middle.
