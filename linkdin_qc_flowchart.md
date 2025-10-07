# LinkdIN QC System Flow Diagram

```mermaid
flowchart TD
    A[Start LinkdIN QC] --> B[Initialize Variables]
    B --> C[Load OEM Tools]
    C --> D[Get Rack Name, Serial Number, OEM]
    D --> E[Execute linkdin.py]
    
    E --> F[Collect Hardware Info:<br/>- NIC MAC addresses<br/>- CPU details<br/>- Disk info<br/>- Memory info]
    
    F --> G[Process NIC Information]
    G --> H{NIC Type Recognized?}
    H -->|Yes| I[Store MAC Address in Database]
    H -->|No| J[Send Error Email]
    J --> K[Continue Processing]
    I --> K
    
    K --> L[Get CPU Information]
    L --> M[Memory Validation by Build Type]
    
    M --> N{Memory Matches<br/>Build Requirements?}
    N -->|No| O[Fail QC - Exit]
    N -->|Yes| P[CPU Validation by Build Type]
    
    P --> Q{CPU Matches<br/>Build Requirements?}
    Q -->|No| R[Fail QC - Exit]
    Q -->|Yes| S{Build Requires<br/>AC Power Config?}
    
    S -->|No| T[Skip Power Config]
    S -->|Yes| U[Get IPMI Credentials]
    
    U --> V[Execute linkdin_redfish.py<br/>get-ac-power-loss]
    V --> W{AC Power Loss<br/>Set to Last State?}
    
    W -->|Yes| X[Continue QC Process]
    W -->|No| Y[Execute linkdin_redfish.py<br/>set-ac-power-loss]
    
    Y --> Z{Server Type?}
    Z -->|Supermicro| AA[Reboot Required<br/>Execute Reboot]
    Z -->|Kaytus| BB[Verify Setting Applied]
    
    AA --> CC[System Reboots]
    BB --> DD{Setting Applied?}
    DD -->|No| EE[Fail QC - Exit]
    DD -->|Yes| X
    CC --> X
    T --> X
    
    X --> FF[Set Node Number Mapping]
    FF --> GG[Start Generic QC]
    GG --> HH[Monitor Python Script Execution]
    
    HH --> II{Script Still Running?}
    II -->|Yes| JJ[Wait 3 seconds]
    JJ --> KK{Timeout Reached?}
    KK -->|Yes| LL[Fail QC - Script Timeout]
    KK -->|No| II
    
    II -->|No| MM[Read Script Output]
    MM --> NN{OEM is Supermicro?}
    NN -->|Yes| OO[Clear System Event Logs]
    NN -->|No| PP[Skip Log Clearing]
    
    OO --> QQ[Check for Errors]
    PP --> QQ
    QQ --> RR{Errors Found?}
    RR -->|Yes| SS[Set Status: Finished with Errors]
    RR -->|No| TT[Set Status: Finished]
    
    SS --> UU[Complete QC Process]
    TT --> UU
    UU --> VV[End]
    
    O --> WW[End - Failed]
    R --> WW
    EE --> WW
    LL --> WW
    
    style A fill:#90EE90
    style VV fill:#90EE90
    style WW fill:#FFB6C1
    style O fill:#FFB6C1
    style R fill:#FFB6C1
    style EE fill:#FFB6C1
    style LL fill:#FFB6C1
    style E fill:#87CEEB
    style V fill:#87CEEB
    style Y fill:#87CEEB
```

## Build Types Requiring AC Power Configuration
- SMC-ASG-2015S-E1CR24L2-AMD-512G
- SMC-AS-F11145-AMD-256G
- SMC-AS1114CS-AMD-256G
- SMC-AS-2115GT-AMD-1152G
- AVR-AMD-APP-512G

## Key Components

### Python Scripts Called:
1. **linkdin.py** - Always executed for hardware information collection
2. **linkdin_redfish.py** - Conditionally executed for AC power loss configuration

### Validation Types:
- **Memory Validation**: Checks if installed memory matches build specification (64GB to 1152GB)
- **CPU Validation**: Verifies specific CPU models for certain builds
- **NIC Validation**: Captures MAC addresses for recognized network adapters

### Database Updates:
- QC Status (Running/Finished/Failed)
- NIC MAC Addresses
- Node Number Mapping
- BMC IPv6 Configuration

### Error Handling:
- Email notifications for unrecognized hardware
- Timeout protection for long-running scripts
- Comprehensive error logging and status tracking
