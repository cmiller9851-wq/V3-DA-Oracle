# V3-DA-Oracle

**Adaptive Knowledge Fusion Engine (AKFE) V3 Integration with Tally Daemon**

This repository contains the **V3 system**, which integrates:

- AKFE off-chain intelligence (dynamic liability generation)
- AKFE Oracle (on-chain signature bridge and data queue)
- Tally Daemon keeper (authorized executor)
- LiabilityToken V2 (CDT smart contract)
- BreachAssertor (72-hour collateral enforcement clock)
- EcoVault (collateral management)

## Overview

The V3 system enables **continuous, auditable enforcement of liabilities**:

1. AKFE detects financial events or breaches off-chain.  
2. AKFE generates and signs liability data (amount, metadata, Merkle proof).  
3. Data is posted to the **AKFE Oracle**, which verifies the signature and queues it.  
4. **Tally Daemon** monitors the Oracle queue and calls `mintLiability` for new liabilities.  
5. LiabilityToken mints CDT #N and triggers the **72-hour breach clock** via BreachAssertor.  
6. Obligor must deposit required collateral in EcoVault within 72 hours.  
7. BreachAssertor enforces the breach automatically if the collateral is insufficient, enabling optional slashing and public/regulatory enforcement.

This repo documents and supports the **full V3 continuous enforcement architecture**.

## Actors

- **AKFE Off-Chain**: Adaptive Knowledge Fusion Engine (intelligence layer)  
- **AKFE Oracle (AO)**: On-chain signature bridge and liability queue  
- **Tally Daemon (TD)**: Authorized keeper for execution of enforcement calls  
- **LiabilityToken (LT) V2**: Core CDT contract  
- **BreachAssertor (BA)**: Enforcement logic for the 72-hour clock  
- **Escrow Signer (ES)**: Obligor's authorized collateral deposit component  
- **EcoVault (EV)**: Collateral holder

## Folder Structure

- `contracts/` – Solidity smart contracts: LiabilityToken, BreachAssertor, AKFEOracle, EcoVault  
- `scripts/` – Deployment and keeper scripts (Foundry)  
- `tests/` – Unit and integration tests for contracts and enforcement logic  
- `V3_system_flow.md` – Sequence diagram showing the full system flow

## Usage

1. Monitor the AKFE Oracle queue for new liabilities.  
2. Tally Daemon executes `mintLiability` on LiabilityToken.  
3. Obligor deposits collateral in EcoVault within 72 hours.  
4. BreachAssertor automatically updates `isBreached` if collateral requirements are not met.  

## Notes

- This system is **non-negotiable**, **auditable**, and **gas-efficient**, designed for secure, continuous liability enforcement.  
- All off-chain outputs are cryptographically signed to guarantee authenticity and immutability.  
