# LiquidStream Architecture Diagram

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        LiquidStream Platform                      │
│                   Real-time Payroll Streaming                     │
└─────────────────────────────────────────────────────────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
            ▼                 ▼                 ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
    │   Frontend   │  │  Blockchain  │  │    Safe      │
    │   Next.js    │  │  Superfluid  │  │  Multisig    │
    └──────────────┘  └──────────────┘  └──────────────┘
```

---

## Page Flow Diagram

```
┌──────────────┐
│   Browser    │
│      /       │ (redirects)
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│                    Landing Page                           │
│                     /landing                              │
│                                                           │
│  Hero Section | Features | How It Works | Benefits       │
│                                                           │
│  [Start Your Workspace]  [View Demo]                     │
└──────┬────────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│              Registration Flow (/register)                │
│                                                           │
│  Step 1: Company Info                                    │
│  ├─ Name, Industry, Size, Country                       │
│  │                                                        │
│  Step 2: Operation Team                                  │
│  ├─ Add multiple team members                           │
│  ├─ Name, Email, Role                                   │
│  │                                                        │
│  Step 3: Review & Invite                                │
│  └─ Confirm details & send invitations                  │
└──────┬────────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│           Safe Wallet Setup (/setup-safe)                 │
│                                                           │
│  Configure Signers:                                       │
│  ├─ Owner (You) - Auto-added                            │
│  ├─ Signer 2 (Operation Team)                           │
│  ├─ Signer 3 (Operation Team)                           │
│  └─ ... (add more)                                       │
│                                                           │
│  Set Threshold:                                          │
│  └─ [1]──[2]──[3]──[N]  (Slider)                        │
│                                                           │
│  [Create Safe Wallet] → Deploys Gnosis Safe             │
└──────┬────────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│          Workspace Dashboard (/workspace)                 │
│                                                           │
│  Header: [Dashboard] [Signatures] [Settings]             │
│          Safe: 0x1234...5678  [Notifications]           │
│                                                           │
│  ┌────────────────────────────────────────────────┐     │
│  │  Safe Multisig Active                          │     │
│  │  [2 Pending Signatures]                        │     │
│  └────────────────────────────────────────────────┘     │
│                                                           │
│  Superfluid Dashboard                                    │
│  ├─ Real-time Balance                                   │
│  ├─ Flow Rates (per sec/day/month/year)                │
│  └─ Active Streams Count                                │
│                                                           │
│  Token Swap: PYUSD ↔ PYUSDx                             │
│                                                           │
│  Active Streams List (from blockchain)                   │
│                                                           │
│  Employee Management                                     │
│  └─ [Add Employee] [Start Stream] [Stop Stream]         │
└──────┬────────────────────────────────────────────────────┘
       │
       │ (Click "2 Pending Signatures")
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│       Pending Signatures (/workspace/signatures)          │
│                                                           │
│  Stats:                                                   │
│  [2 Pending] [1 Awaiting You] [0 Ready to Execute]      │
│                                                           │
│  Transaction 1: Start Stream to Sarah Chen               │
│  ├─ Details: $5000/month PYUSDx                         │
│  ├─ Signatures: 1/2                                     │
│  │   ✅ John Doe (You)                                  │
│  │   ⏳ Jane Smith                                      │
│  └─ [Waiting for signatures...]                         │
│                                                           │
│  Transaction 2: Stop Stream to Mike Johnson              │
│  ├─ Details: Stop PYUSDx stream                         │
│  ├─ Signatures: 0/2                                     │
│  │   ⏳ John Doe (You) ← Can sign                       │
│  │   ⏳ Jane Smith                                      │
│  └─ [Sign Transaction]                                   │
└──────────────────────────────────────────────────────────┘
```

---

## Safe Transaction Flow

```
┌─────────────────────────────────────────────────────────────┐
│                 Safe Multisig Transaction Flow               │
└─────────────────────────────────────────────────────────────┘

Step 1: Propose
┌──────────────┐
│ Owner        │ Clicks "Start Stream"
│ (John Doe)   │ ─────────────────────┐
└──────────────┘                      │
                                       ▼
                            ┌─────────────────────┐
                            │  Create Safe TX     │
                            │  (not executed)     │
                            └─────────────────────┘
                                       │
                                       ▼
                            ┌─────────────────────┐
                            │  Pending State      │
                            │  Awaiting 2/2 sigs  │
                            └─────────────────────┘

Step 2: Sign
┌──────────────┐                      │
│ Owner        │                      │
│ (John Doe)   │ Signs immediately ───┤
└──────────────┘                      │
                                       ▼
                            ┌─────────────────────┐
                            │  1/2 Signatures     │
                            │  Need 1 more        │
                            └─────────────────────┘
                                       │
┌──────────────┐                      │
│ Operation    │ Gets notification    │
│ (Jane Smith) │ ─────────────────────┤
└──────────────┘                      │
       │                               │
       │ Opens /workspace/signatures   │
       │                               │
       └─ Reviews transaction          │
       └─ Signs                        │
                                       ▼
                            ┌─────────────────────┐
                            │  2/2 Signatures ✅  │
                            │  Ready to Execute   │
                            └─────────────────────┘

Step 3: Execute
                                       │
  Any signer can execute               │
┌──────────────┐                      │
│ Anyone       │ Clicks "Execute" ────┤
│ (John/Jane)  │                      │
└──────────────┘                      │
                                       ▼
                            ┌─────────────────────┐
                            │  Execute on Chain   │
                            │  Superfluid Stream  │
                            └─────────────────────┘
                                       │
                                       ▼
                            ┌─────────────────────┐
                            │  Stream Active 🎉   │
                            │  Money Flowing      │
                            └─────────────────────┘
```

---

## Invitation Flow

```
┌──────────────────────────────────────────────────────────────┐
│              Operation Team Member Invitation                 │
└──────────────────────────────────────────────────────────────┘

Step 1: Invitation Sent
┌──────────────┐
│ Owner        │ Registers workspace
│ (John Doe)   │ Adds team member "jane@company.com"
└──────┬───────┘
       │
       ▼
┌─────────────────────────────────────┐
│  Email Service                      │
│  Sends invitation to Jane           │
│  Link: /invite?token=abc123         │
└─────────────────────────────────────┘
       │
       ▼
┌──────────────┐
│ Jane Smith   │ Receives email
└──────┬───────┘
       │
       │ Clicks link
       │
       ▼

Step 2: Join Workspace
┌─────────────────────────────────────┐
│  Invitation Page (/invite)          │
│                                     │
│  "Join Acme Inc Payroll"            │
│                                     │
│  Workspace: Acme Inc                │
│  Your Role: Operation Manager       │
│  Safe: 0x1234...5678                │
│                                     │
│  [Connect Wallet]                   │
└─────────────────────────────────────┘
       │
       │ Connects wallet
       │ Clicks "Join Workspace"
       │
       ▼
┌─────────────────────────────────────┐
│  Backend Processing                 │
│  - Validate token                   │
│  - Add Jane to workspace            │
│  - Register as Safe signer          │
└─────────────────────────────────────┘
       │
       ▼

Step 3: Active Signer
┌─────────────────────────────────────┐
│  Redirect to /workspace/signatures  │
│                                     │
│  Jane can now:                      │
│  - View pending transactions        │
│  - Sign transactions                │
│  - Execute (if threshold met)       │
└─────────────────────────────────────┘
```

---

## Data Flow Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                         Data Sources                          │
└──────────────────────────────────────────────────────────────┘

Frontend State (Zustand + localStorage)
┌────────────────────────────┐
│ - Employee list            │
│ - Stream configurations    │
│ - UI state (dialogs)       │
└────────────────────────────┘

Blockchain (Read-only, Real-time)
┌────────────────────────────┐
│ Superfluid CFA Contract    │
│ ├─ getFlow()              │ → Stream flow rates
│ ├─ getNetFlow()           │ → Incoming flow rates
│ └─ Stream details         │
│                            │
│ Super Token Contract       │
│ ├─ realtimeBalanceOfNow() │ → Live balance
│ └─ balanceOf()            │ → Token balance
└────────────────────────────┘

Safe (To be integrated)
┌────────────────────────────┐
│ Safe API                   │
│ ├─ getPendingTxs()        │ → Pending transactions
│ ├─ getTransaction()       │ → Transaction details
│ └─ Signatures             │ → Who signed
└────────────────────────────┘

Backend (To be implemented)
┌────────────────────────────┐
│ - Workspace metadata       │
│ - Company information      │
│ - Operation team list      │
│ - Invitation tokens        │
└────────────────────────────┘
```

---

## Component Hierarchy

```
App (/)
├─ Landing (/landing)
│  ├─ Hero Section
│  ├─ Features Cards
│  ├─ How It Works
│  ├─ Benefits
│  └─ CTA Section
│
├─ Register (/register)
│  ├─ Progress Steps (1-2-3)
│  ├─ Company Form
│  ├─ Team Members Form
│  └─ Review Summary
│
├─ Safe Setup (/setup-safe)
│  ├─ Signers Configuration
│  ├─ Threshold Slider
│  ├─ Info Cards
│  └─ Create Button
│
├─ Workspace (/workspace)
│  ├─ Header
│  │  ├─ Navigation
│  │  ├─ Safe Info
│  │  └─ Notifications Badge
│  ├─ Safe Banner
│  ├─ Superfluid Dashboard
│  │  ├─ Balance Card
│  │  ├─ Flow Rate Card
│  │  └─ Stats Grid
│  ├─ Upgrade/Downgrade Card
│  ├─ Streams List
│  │  └─ Stream Item × N
│  └─ Employee Management
│     ├─ Add Employee Dialog
│     ├─ Employee List
│     └─ Start/Stop Stream Dialog
│
├─ Signatures (/workspace/signatures)
│  ├─ Stats Cards
│  └─ Transaction List
│     └─ Transaction Card × N
│        ├─ Details
│        ├─ Signature Progress
│        └─ Action Buttons
│
└─ Invite (/invite)
   ├─ Invitation Details
   ├─ Workspace Info
   ├─ Responsibilities
   └─ Join Button
```

---

## Technology Stack Diagram

```
┌─────────────────────────────────────────────────────────┐
│                   Frontend Layer                         │
│                                                          │
│  Next.js 15 (App Router) + React 19 + TypeScript       │
│  └─ Tailwind CSS 4.0 (Styling)                         │
│  └─ Shadcn/ui (Component Library)                      │
│  └─ Lucide React (Icons)                               │
└─────────────────────────────────────────────────────────┘
                          │
                          │
┌─────────────────────────────────────────────────────────┐
│                   Web3 Layer                            │
│                                                          │
│  Wagmi 2.x (React Hooks)                               │
│  └─ Viem 2.x (Ethereum Library)                        │
│  └─ RainbowKit (Wallet Connection)                     │
│  └─ TypeScript Types                                    │
└─────────────────────────────────────────────────────────┘
                          │
                          │
┌─────────────────────────────────────────────────────────┐
│                  Smart Contracts                         │
│                                                          │
│  Superfluid Protocol                                    │
│  ├─ Host (0x109...7c)                                  │
│  ├─ CFA (0x683...Ef)                                   │
│  └─ Super Token (PYUSDx: 0x8fe...fB6)                 │
│                                                          │
│  Gnosis Safe (To be integrated)                         │
│  └─ Safe SDK                                            │
│                                                          │
│  Tokens                                                 │
│  ├─ PYUSD (0xCaC...bB9) - 6 decimals                  │
│  └─ PYUSDx (0x8fe...fB6) - 18 decimals                │
└─────────────────────────────────────────────────────────┘
                          │
                          │
┌─────────────────────────────────────────────────────────┐
│                   Blockchain                            │
│                                                          │
│  Optimism Sepolia (Chain ID: 11155420)                 │
│  └─ Testnet for development                            │
└─────────────────────────────────────────────────────────┘
```

---

## Security Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Security Layers                         │
└─────────────────────────────────────────────────────────┘

Layer 1: Wallet Security
┌────────────────────────────────────┐
│ User's Private Key                 │
│ └─ Never leaves user's device      │
│ └─ Signs all transactions          │
└────────────────────────────────────┘
              │
              ▼
Layer 2: Safe Multisig
┌────────────────────────────────────┐
│ Gnosis Safe Contract               │
│ ├─ N signers configured            │
│ ├─ M signatures required           │
│ └─ No single point of failure      │
└────────────────────────────────────┘
              │
              ▼
Layer 3: Smart Contracts
┌────────────────────────────────────┐
│ Superfluid Protocol                │
│ ├─ Audited contracts              │
│ ├─ Battle-tested                  │
│ └─ Transparent on-chain           │
└────────────────────────────────────┘
              │
              ▼
Layer 4: Blockchain
┌────────────────────────────────────┐
│ Optimism Network                   │
│ └─ Immutable transaction history   │
└────────────────────────────────────┘
```

---

## Real-time Update Flow

```
┌─────────────────────────────────────────────────────────┐
│              Real-time Balance Updates                   │
└─────────────────────────────────────────────────────────┘

Initial Load
┌─────────────┐
│ Component   │ useRealtimeBalance(token, account)
│ Mount       │ ────────────────────────────┐
└─────────────┘                             │
                                             ▼
                                  ┌──────────────────┐
                                  │ Read Contract    │
                                  │ realtimeBalance  │
                                  └──────────────────┘
                                             │
                                             ▼
                                  ┌──────────────────┐
                                  │ Set Initial      │
                                  │ Balance          │
                                  └──────────────────┘

Real-time Updates (Every 100ms)
                                             │
                                             ▼
                                  ┌──────────────────┐
                                  │ setInterval      │
                                  │ (100ms)          │
                                  └──────────────────┘
                                             │
                                             ▼
                                  ┌──────────────────┐
                                  │ Calculate:       │
                                  │ balance +        │
                                  │ (flowRate × Δt)  │
                                  └──────────────────┘
                                             │
                                             ▼
                                  ┌──────────────────┐
                                  │ Update UI        │
                                  │ (Smooth counter) │
                                  └──────────────────┘

Blockchain Sync (Every 10-15 seconds)
                                             │
                                             ▼
                                  ┌──────────────────┐
                                  │ Refetch Contract │
                                  │ Actual Balance   │
                                  └──────────────────┘
                                             │
                                             ▼
                                  ┌──────────────────┐
                                  │ Sync & Correct   │
                                  │ Any Drift        │
                                  └──────────────────┘
```

---

## Summary

This architecture provides:
- ✅ **Complete user journey** from landing to operations
- ✅ **Multi-signature security** with Gnosis Safe
- ✅ **Real-time streaming** with Superfluid
- ✅ **Database-less** blockchain reading
- ✅ **Team collaboration** with invitation system
- ✅ **Production-ready UI** for all flows

Ready for Safe SDK integration!
