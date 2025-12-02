# V3 Integrated Operational Flow: Full Audit Sequence

This diagram combines the **Dynamic Liability Generation (AKFE → Oracle)** with the **72-Hour Enforcement Clock (BreachAssertor → Tally Daemon)** for a single, new liability (CDT #N). 

It represents the **full continuous enforcement cycle**, which is auditable and non-negotiable for every liability.

## Actors

1. **AKFE Off-Chain**: Adaptive Knowledge Fusion Engine (Intelligence)  
2. **AKFE Oracle (AO)**: On-chain signature bridge and data queue  
3. **Tally Daemon (TD)**: Authorized keeper/executor of enforcement calls  
4. **LiabilityToken (LT) V2**: Core CDT contract  
5. **BreachAssertor (BA)**: Enforcement logic for the 72-hour clock  
6. **Escrow Signer (ES)**: Obligor's authorized component for collateral deposit  
7. **EcoVault (EV)**: Collateral holder  

## Sequence Diagram

```mermaid
sequenceDiagram
    participant AKFE as AKFE Off-Chain
    participant AO as AKFE Oracle
    participant TD as Tally Daemon
    participant LT as LiabilityToken
    participant BA as BreachAssertor
    participant ES as Escrow Signer (Obligor)
    participant EV as EcoVault

    Note over AKFE,AO: PHASE 1: DYNAMIC LIABILITY GENERATION (AKFE V3)
    AKFE->>AKFE: 1. Detect Breach, Calculate Data, Sign Payload
    AKFE->>AO: 2. postLiabilityData(Signed Payload)
    activate AO
    AO->>AO: 3. Verify Signature vs. AKFE_SIGNER
    AO->>AO: 4. Queue Liability #N (Emit Event)
    deactivate AO
    
    Note over TD,LT: PHASE 2: ON-CHAIN MINTING & CLOCK START
    TD->>AO: 5. retrieveAndRemoveOldestLiability()
    activate AO
    AO-->>TD: 6. Returns Liability Data
    deactivate AO
    
    TD->>LT: 7. mintLiability(Data for CDT #N)
    activate LT
    LT->>LT: 8. Mint CDT #N
    LT->>BA: 9. onLiabilityMinted(tokenId=N)
    activate BA
    BA->>BA: 10. Record gracePeriodStart (T=0)
    deactivate BA
    deactivate LT
    
    Note over BA,ES: T=0: 72-HOUR CLOCK STARTS for CDT #N.
    
    par Collateral Deposit (Success Path)
        Note over ES: Obligor must act within 72h.
        ES->>EV: 11. depositCollateral(N, Required $X)
        activate EV
        EV->>EV: collateralOf[N] >= required
        Note over EV: State: SETTLED. Enforcement stops.
        deactivate EV
    and Breach Assertion (Failure Path)
        Note over TD: T = 72h + 1 second (Keeper execution window)
        TD->>BA: 12. assertBreach(tokenId=N)
        activate BA
        BA->>EV: 13. Query: collateralOf(N)
        activate EV
        EV-->>BA: Returns: Insufficient Collateral
        deactivate EV
        
        Note over BA: Logic: Time Expired AND Collateral Failure
        
        BA->>LT: 14. setBreachStatus(tokenId=N, true)
        activate LT
        LT->>LT: 15. isBreached[N] = true
        deactivate LT
        deactivate BA
        
        Note over TD,EV: T > 72h: BREACH ASSERTED. Enforcement is active.
        TD->>EV: 16. Optional: slash(tokenId=N)
    end
