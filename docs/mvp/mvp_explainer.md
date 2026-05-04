# MES Platform — MVP Explainer

> **Who is this for?** Anyone joining this project who wants to understand what we're building, why it matters, and how it works — no manufacturing or software background needed.

---

## 1. What Are We Building?

**One sentence:** We're building software that **tracks every step** of how a product is manufactured — who made it, what parts went into it, which machine was used, and whether it passed quality checks.

### The Real-World Analogy: A Restaurant Kitchen

Imagine a high-end restaurant where:
- Every dish has a **ticket** (our *Work Order*)
- The ticket travels through stations: Prep → Grill → Plating → Quality Check
- At **each station**, someone writes down:
  - Who handled it (the chef)
  - What ingredients were used (and which batch they came from)
  - What equipment was used (which oven, which grill)
  - Whether it passed the chef's quality check
- If a dish fails quality check, it goes to a **fix-it station** before trying again
- At the end, the restaurant can look up any dish and know *exactly* how it was made

**That's what our software does — but for electronics manufacturing (circuit boards, not dishes).**

---

## 2. The Problem We're Solving

### How Factories Work Today (The Mess)

Most electronics factories still track production using:
- 📋 **Paper route cards** that travel with the product
- 📊 **Excel spreadsheets** updated manually at end of shift
- 🧠 **People's memory** ("I think we used component batch #42 on that board")

### Why This is a Problem

| Scenario | What Happens Today | What Our Software Does |
|---|---|---|
| A customer reports a defective product | Scramble through paper records for days | Type the serial number → see every detail in seconds |
| A bad batch of components is discovered | No idea which products used them — recall everything | Instant reverse-trace: "These 47 units used that batch" |
| An operator skips a step | Nobody knows until the product fails in the field | System **blocks** the product from moving forward |
| Manager wants to know production status | Walk the floor and ask supervisors | Real-time dashboard on any screen |

### The Industry Term

This type of software is called a **Manufacturing Execution System (MES)**. Think of it as the "operating system" for a factory floor — sitting between the business planning software (ERP, like SAP) and the physical machines.

```
┌─────────────────────────────┐
│  ERP (SAP, Oracle, etc.)    │  ← "What to make and how many"
│  Business Planning          │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│  ★ OUR MES PLATFORM ★      │  ← "How it's being made, RIGHT NOW"
│  Production Tracking        │
│  Quality Control            │
│  Traceability               │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│  Machines & Operators       │  ← The physical factory floor
│  (SMT lines, test stations) │
└─────────────────────────────┘
```

---

## 3. What is a PCB / SMT Line?

Since our first customer makes **electronics**, here's a quick primer:

- **PCB** = Printed Circuit Board — the green board inside every phone, laptop, TV, etc.
- **SMT** = Surface Mount Technology — the automated process of placing tiny electronic components onto a PCB

### How a PCB Gets Made (Simplified)

```
Step 1          Step 2          Step 3         Step 4        Step 5       Step 6
┌─────┐        ┌─────┐        ┌─────┐       ┌─────┐      ┌─────┐     ┌─────┐
│Paste│   →    │Pick │   →    │Heat │  →    │Photo│  →   │Elec.│ →   │Full │
│Print│        │Place│        │(Oven)│      │Check│      │Test │     │Test │
└─────┘        └─────┘        └─────┘       └─────┘      └─────┘     └─────┘
Apply          Robots place   Solder melts   Camera       Electrical   Does the
solder paste   tiny parts     & bonds the    inspects     probes test  whole thing
on the board   on the board   parts to board for defects  connections  actually work?
```

**Our software tracks the board through ALL of these steps.**

---

## 4. The 6 Core Features of Our MVP

### Feature 1: Serial Number & Identity 🏷️

**Analogy:** Every person has a unique national ID number. Every PCB gets a unique serial number.

**What it does:**
- When a work order says "make 500 boards," the system generates 500 unique serial numbers
- Each serial number is printed as a barcode/QR code on the board
- From this moment, the system tracks everything that happens to this specific board

---

### Feature 2: Production Routing (The Dispatcher) 🗺️

**Analogy:** GPS navigation — it knows the route and tells you the next turn.

**What it does:**
- We define the "recipe" for each product: Step 1 → Step 2 → Step 3, etc.
- When a board is scanned at a station, the system checks: "Is this the right station for this board right now?"
- If yes → proceed. If no → "Wrong station! This board should be at Station 3."
- The route is **not hard-coded** — it's configurable. Different products can have different routes. We can change routes without changing code.

**Why this matters:** This is what makes our platform **process-agnostic**. The same software can handle electronics, automotive, pharma — just change the route configuration.

---

### Feature 3: Interlocks (The Bouncers) 🚫

**Analogy:** Airport security — you can't board the plane without a valid ticket and ID check.

**What it does:**
Before a board can enter a station, the system checks 3 things:

| Check | Question | Example |
|---|---|---|
| **Sequence Check** | Did it complete the previous step? | Can't enter the oven without paste printing done first |
| **Material Check** | Are the right parts loaded on the machine? | Verify the component reel matches what the recipe needs |
| **Operator Check** | Is this person qualified to run this station? | Only certified operators can run the pick & place machine |

If ANY check fails → the board is **blocked**. The operator sees an error on screen. A supervisor can override it (with a logged reason), but the system never silently lets a bad board through.

---

### Feature 4: Traceability — The "Digital Birth Certificate" 📜

**Analogy:** A passport with every stamp from every country you've visited.

**What it does:**
For every single board, the system records:
- **Man** — Which operator worked on it at each step
- **Machine** — Which specific machine processed it
- **Material** — Which exact batch of components went into it
- **Method** — Which process/program was used

This is called **4M Tracking** in manufacturing.

**The killer query:** Type in any serial number and get the complete life story:

```
Serial: PCB-2026-00001
├── Station 1: Paste Print
│   ├── Operator: John (EMP-042)
│   ├── Machine: Printer-01
│   ├── Solder Paste: Lot SP-2026-0042 (Kester NXG1)
│   ├── Time: 2026-05-03 09:14:22 (12.5 sec)
│   └── Result: PASS
├── Station 2: Pick & Place
│   ├── Operator: John (EMP-042)
│   ├── Machine: PP-Line1
│   ├── Components Used:
│   │   ├── Resistor 100Ω: Lot R-2026-001, Reel #42
│   │   ├── Capacitor 10µF: Lot C-2026-088, Reel #17
│   │   └── IC U100: Lot IC-2026-045, Reel #03
│   ├── Time: 2026-05-03 09:15:10 (45.2 sec)
│   └── Result: PASS
├── Station 3: Reflow Oven
│   ├── Machine: Oven-01
│   ├── Peak Temperature: 245.5°C (spec: 240-250°C ✓)
│   └── Result: PASS
├── Station 4: AOI (Automatic Optical Inspection)
│   ├── Machine: AOI-01
│   └── Result: PASS
├── Station 5: ICT (In-Circuit Test)
│   ├── Machine: ICT-01
│   ├── Test Points: 1247/1247 passed
│   └── Result: PASS
└── Station 6: FCT (Functional Test)
    ├── Machine: FCT-01
    ├── All functions verified
    └── Result: PASS ✅

Status: PASSED → Ready for packing
```

**Why this matters:** If a customer reports a failure, or a component supplier issues a recall, we can trace EXACTLY which units are affected in seconds, not days.

---

### Feature 5: Rework Loop (The Second Chance) 🔄

**Analogy:** If your exam answer is wrong, you get sent back to study that topic and retake it.

**What it does:**
- If a board fails at any inspection (e.g., AOI sees a missing component):
  1. Board is flagged as **Failed**
  2. Routed to a **Rework Station**
  3. Technician diagnoses & fixes the issue
  4. Board re-enters the line at the station where it failed
  5. Re-inspected — if it passes this time, it continues normally
- Everything is tracked: what the defect was, who fixed it, how many times it's been reworked
- If rework count exceeds a limit → board is **scrapped** (can't keep fixing forever)

---

### Feature 6: Edge Gateway (The Translator) 🔌

**Analogy:** A universal power adapter that lets you plug any device into any wall outlet.

**What it does:**
Factory machines speak different "languages":
- Some machines send data via **MQTT** (a messaging protocol — like WhatsApp for machines)
- Some drop **CSV files** in a folder (like old-school email attachments)
- Some use **OPC-UA** (an industrial data protocol)

Our Edge Gateway:
1. **Listens** to all these different sources
2. **Translates** machine-specific data into a standard format
3. **Feeds** it into the MES system automatically

**Without this:** An operator would have to manually type "AOI result: PASS" into the system for every single board. With thousands of boards per day, that's impossible.

---

## 5. Tech Stack (Explained Simply)

| Component | Technology | Plain English |
|---|---|---|
| **Backend** (the brain) | Node.js + TypeScript | The server that runs all the business logic. TypeScript adds strict type checking — like spell-check but for code, preventing bugs. |
| **Database** (the memory) | PostgreSQL + Prisma | Where all data lives permanently. Prisma is a tool that makes it easy to read/write data without writing complex database queries. |
| **Time-series DB** (the speedometer) | TimescaleDB | Special database optimized for machine sensor data (temperatures every second, thousands of events per minute). |
| **Cache** (the sticky note) | Redis | Ultra-fast temporary storage. When a board is scanned, we need to know its status in milliseconds — Redis holds that. |
| **Message Broker** (the postman) | Mosquitto (MQTT) | Delivers messages between machines and our system in real-time. |
| **Edge Gateway** (the translator) | Python + FastAPI | Connects to physical machines, translates their data into a format our system understands. |
| **Frontend** (the face) | React.js + Tailwind | The screens operators and managers actually see — dashboards, scan interfaces, reports. |
| **Containerization** (the packaging) | Docker | Bundles everything so the system runs the same way everywhere — developer laptop, test server, or factory server. |

---

## 6. How the MVP Demo Works

### The "Happy Path" (everything goes right)

```
1. PLANNER creates Work Order: "Make 10 units of PCBA-X100"
        ↓
2. SYSTEM generates 10 unique serial numbers (PCB-001 through PCB-010)
        ↓
3. OPERATOR scans PCB-001 at Station 1 (Paste Print)
   → System checks: Correct station? ✅ Operator certified? ✅ Right paste loaded? ✅
   → Processing happens → System records everything
        ↓
4. Board moves through Stations 2-6, same scan-check-record at each
        ↓
5. After Station 6 (FCT): All tests passed → Board status = PASSED ✅
        ↓
6. ANYONE can now search "PCB-001" and see the complete birth certificate
```

### The "Failure Path" (something goes wrong)

```
1. PCB-005 arrives at Station 4 (AOI)
        ↓
2. AOI camera detects: Missing component at position C47
        ↓
3. System marks PCB-005 as FAILED, routes to Rework Station
        ↓
4. TECHNICIAN scans PCB-005 at Rework
   → System shows: "Defect: Missing component C47 (detected by AOI-01)"
   → Technician manually places the component
   → Marks repair as complete
        ↓
5. PCB-005 re-enters the line at Station 4 (AOI) for re-inspection
        ↓
6. AOI re-check: PASS ✅ → Board continues to Station 5 and 6
        ↓
7. Birth certificate now shows: "Rework performed, attempt 1 of 3 max"
```

---

## 7. Why This Product Has Big Potential

### The "Modular" Advantage

Our MVP handles **production tracking**. But the database and architecture are designed so we can **plug in** additional modules later without rebuilding:

```
                        ┌─────────────────┐
                        │  Quality        │
                        │  Planning       │  ← Phase 3
                        └────────┬────────┘
                                 │
┌──────────┐  ┌─────────┐  ┌────▼────┐  ┌──────────┐
│ Warehouse │  │ Mainte- │  │ ★ MES  │  │ Analytics│
│ Mgmt     │  │ nance   │  │ (MVP)  │  │ & OEE    │
│ (WMS)    │  │ Mgmt    │  │        │  │          │
└──────────┘  └─────────┘  └────────┘  └──────────┘
     ↑             ↑            ↑            ↑
     └─────────────┴────────────┴────────────┘
              All share the same core
              Equipment, Material, and
              Personnel data tables
```

This full suite is called **MOM** (Manufacturing Operations Management). We're building the MES core first — but the architecture supports the full MOM vision.

### Industry Agnostic

Because process steps are **configured, not coded**, the same platform can serve:
- 📱 Electronics (our first customer)
- 🚗 Automotive
- 💊 Pharmaceuticals
- 🔧 General discrete manufacturing

---

## 8. Key Terms Glossary

| Term | Meaning |
|---|---|
| **MES** | Manufacturing Execution System — software that manages and tracks production in real-time |
| **MOM** | Manufacturing Operations Management — the full suite (MES + Maintenance + Quality + Warehouse) |
| **ISA-95** | International standard that defines how manufacturing systems should be structured. We follow this so our system "speaks the same language" as any other industrial software |
| **PCB / PCBA** | Printed Circuit Board / PCB Assembly — the green boards inside electronics |
| **SMT** | Surface Mount Technology — automated process of placing components on PCBs |
| **Work Order** | An instruction to produce a specific quantity of a product |
| **Serial Number** | Unique ID for each individual product unit |
| **Routing** | The sequence of steps a product must go through |
| **Interlock** | A validation check that blocks the product if something is wrong |
| **Traceability** | The ability to trace every detail of how a product was made |
| **4M** | Man, Machine, Material, Method — the four things tracked at every step |
| **AOI** | Automatic Optical Inspection — camera that checks for defects |
| **ICT/FCT** | In-Circuit Test / Functional Circuit Test — electrical testing |
| **ERP** | Enterprise Resource Planning — business software (SAP, Oracle) that tells the factory WHAT to make |
| **HAL** | Hardware Abstraction Layer — our translator between machines and software |
| **MQTT** | A lightweight messaging protocol machines use to send data |
| **OPC-UA** | An industrial communication standard for machine data |
| **Digital Birth Certificate** | Our name for the complete traceable history of a single product unit |
