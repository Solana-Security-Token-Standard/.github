# Solana Security Token Standard (SSTS)

SSTS is a foundation-led initiative focused on open standards for regulated tokenized securities on Solana. The goal is to provide a practical baseline for issuance, transfer controls, and long-term lifecycle operations that institutions can use in production.

## Foundation Mission

The foundation’s role is to steward a neutral, reusable standard for compliant digital securities. Rather than forcing each issuer to design policy infrastructure from scratch, SSTS defines shared patterns that can be adopted, audited, and extended across markets.

## What The Standard Defines

SSTS builds on SPL Token-2022 and introduces a configurable verification model so compliance behavior can adapt to local regulatory requirements. The core protocol stays stable, while verification logic can evolve for KYC and AML controls, investor qualification policies, transfer approvals, and related operational rules. The standard also supports broader lifecycle flows, including issuance, policy-aware transfer, and corporate actions such as splits, conversions, and distributions.

The diagrams below separate architecture from runtime behavior and show the two transfer paths the implementation supports.

```mermaid
flowchart TB
  SPL["SPL Standards"] --> T22["Token-2022 Program"]
  T22 --> Mint["Issuer Security Mints and Token Accounts"]

  Core["SSTS Core Program"] -->|initialize Token-2022 extensions| T22
  Core -->|create or update Transfer VerificationConfig| VCfg["Transfer VerificationConfig PDA"]
  Core -->|initialize or update ExtraAccountMetaList| Hook["SSTS Transfer Hook Program"]

  Hook -->|read transfer config and program list| VCfg
  Hook -->|CPI on direct token transfers| VProg["Verification Programs"]
  Core -->|CPI when cpi_mode=true| VProg
  VProg -.->|called before core when cpi_mode=false| Core
```

Forced transfer via SSTS core instruction (`SecurityTokenInstruction::Transfer`):

```mermaid
sequenceDiagram
  participant User
  participant Core as SSTS Core Program
  participant Verify as Verification Programs
  participant T22 as Token-2022
  participant Hook as SSTS Transfer Hook

  User->>Core: Transfer instruction
  alt cpi_mode = true
    Core->>Verify: CPI verification calls
    Verify-->>Core: Approve or reject
  else cpi_mode = false
    User->>Verify: Pre-call verification programs
    Verify-->>User: Approve
    User->>Core: Transfer instruction in same transaction
  end
  Core->>T22: transfer_checked (authority = permanent_delegate PDA)
  T22->>Hook: Execute transfer hook
  Hook-->>T22: Bypass path (permanent_delegate and no extra accounts)
  T22-->>Core: Transfer result
  Core-->>User: Success or failure
```

Direct Token-2022 wallet transfer:

```mermaid
sequenceDiagram
  participant Holder
  participant T22 as Token-2022
  participant Hook as SSTS Transfer Hook
  participant Verify as Verification Programs

  Holder->>T22: transfer_checked
  T22->>Hook: Execute transfer hook
  Hook->>Hook: Load VerificationConfig from extra accounts
  Hook->>Verify: CPI verification program calls
  Verify-->>Hook: Approve or reject
  Hook-->>T22: Return decision
  T22-->>Holder: Settle transfer or fail atomically
```

## Why This Matters

Tokenized securities require more than token movement. They require policy enforcement, auditability, and repeatable operational workflows. SSTS is designed to provide that shared infrastructure so institutions can focus on product and market design, instead of rebuilding core compliance and servicing rails each time.

## Stewardship

SSTS is developed with contributions from Halborn, Hoodies, and a panel of industry experts, with support from the Solana Foundation.

This organization is at an early stage. Today, it is focused on the core standard and its reference implementation. As the initiative matures, this org will gradually expand to include clearer implementation guidance, practical examples, and selected documentation artifacts, including material that may be synchronized from the GitBook docs when appropriate.

## Learn More

Visit [ssts.org](https://ssts.org) for broader context and current initiative updates.
