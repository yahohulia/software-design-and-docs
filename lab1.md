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
