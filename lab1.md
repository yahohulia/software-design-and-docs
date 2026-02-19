## 🔹 Variant 2 — Message Status Tracking
**Focus:** state machine and lifecycle

**Additional requirements:**
- Message statuses: `sent`, `delivered`, `read`
- Client acknowledgements

**Key questions:**
- Who updates message status?
- What happens if acknowledgements are missing?

## Component Diagram

```mermaid
graph LR
  ClientSender[Client A - Sender] --> API
  ClientReceiver[Client B - Receiver] --> API
  
  subgraph "Backend"
    API[Backend API] --> MsgService[Message Service]
    API --> StatusService[Status Tracker]
    MsgService --> DB[(Message & Status DB)]
    StatusService --> DB
    MsgService --> WS[WebSocket/Push Service]
    StatusService --> WS
  end
  
  WS --> ClientReceiver
  WS --> ClientSender
```

## Sequence Diagram

```mermaid
sequenceDiagram
    participant A as User A (Sender)
    participant MsgService as Message Service
    participant DB as Database
    participant B as User B (Receiver)

    A->>MsgService: Send message
    MsgService->>DB: Save message (status: sent)
    MsgService-->>A: 201 Created (status: sent)
    
    MsgService->>B: Deliver message
    B->>MsgService: Acknowledgement (delivered)
    MsgService->>DB: Update status: delivered
    MsgService->>A: Push: Message delivered
    
    Note over B: User B opens chat
    B->>MsgService: Acknowledgement (read)
    MsgService->>DB: Update status: read
    MsgService->>A: Push: Message read
```

## State Diagram

```mermaid
stateDiagram-v2
    [*] --> Sent: Message received by Server
    
    Sent --> Delivered: [Recipient is Online] <br/> Direct delivery
    Sent --> Pending: [Recipient is Offline] <br/> Wait for connection
    
    Pending --> Delivered: [Recipient comes Online] <br/> Push/Sync
    
    Delivered --> Read: [Recipient opens chat] <br/> Client ACK
    
    Sent --> Failed: [Network Error / Timeout]
    Failed --> Sent: Retry logic
```
