%%{init: {
"flowchart": {
"curve": "stepAfter"
}
}}%%

flowchart TB

    USER[User]

    LB[Load Balancer]

    AG[API Gateway]

    USER --> LB
    LB --> AG

    %% Services Layer
    subgraph SERVICES
        direction TB

        US[User Service]
        PS[Post Service]
        CS[Comment Service]
        LS[Like Service]
        FS[Feed Service]
        MS[Media Service]
        NS[Notification Service]
    end

    AG --> US
    AG --> PS
    AG --> CS
    AG --> LS
    AG --> FS
    AG --> MS

    %% Databases Layer
    UDB[(User DB)]
    PDB[(Post DB)]
    CDB[(Comment DB)]
    LDB[(Like DB)]
    REDIS[(Redis Cache)]

    US --> UDB
    PS --> PDB
    CS --> CDB
    LS --> LDB

    FS --> REDIS
    REDIS --> PDB

    %% Media Layer
    OS[(Object Storage)]
    CDN[CDN]

    MS --> OS
    OS --> CDN
    CDN --> USER

    %% Async Layer
    MQ[Message Queue]

    PS --> MQ
    MQ --> NS

    %% Service Discovery
    SR[Service Registry]

    US -.-> SR
    PS -.-> SR
    CS -.-> SR
    LS -.-> SR
    FS -.-> SR
    MS -.-> SR
    NS -.-> SR
