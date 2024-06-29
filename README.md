# Sandy CRM 🥐 

> Sandy is the personal assistant for small to medium consulting companies

Built with a modern stack, cloud ready, self hostable

```mermaid
graph TD
  subgraph "User Interface"
    User --> ReactFrontEnd[React Front End]
  end

  subgraph "Middleware"
    ReactFrontEnd --> APIGateway[API Gateway]
    APIGateway --> CloudFunctions[Cloud Functions]
  end

  subgraph "Back End"
    CloudFunctions --> CloudStorage[Cloud Storage]
  end
```

## Storage Model

deployed on cloud storage (`s3`, `gcs`, `adls`, etc), implements the event sourcing pattern, storing the current state along with a change log for each entity, along with indexes to facilitate look ups and joins.  Folder structure is here:

```
org-name/
├── current/
│   ├── customers/
│   │   ├── customer_1.json.gz
│   │   ├── customer_2.json.gz
│   │   └── ...
│   ├── contacts/
│   │   ├── contact_1.json.gz
│   │   ├── contact_2.json.gz
│   │   └── ...
│   ├── opportunities/
│   │   ├── opportunity_1.json.gz
│   │   ├── opportunity_2.json.gz
│   │   └── ...
│   └── ...
├── eventlog/
│   ├── customers/
│   │   ├── customer_1/
│   │   │   ├── event_20220101T123456.json.gz
│   │   │   ├── event_20220102T123456.json.gz
│   │   │   └── ...
│   │   ├── customer_2/
│   │   │   ├── event_20220101T123456.json.gz
│   │   │   ├── event_20220102T123456.json.gz
│   │   │   └── ...
│   │   └── ...
│   ├── contacts/
│   │   ├── contact_1/
│   │   │   ├── event_20220101T123456.json.gz
│   │   │   ├── event_20220102T123456.json.gz
│   │   │   └── ...
│   │   └── ...
│   ├── opportunities/
│   │   ├── opportunity_1/
│   │   │   ├── event_20220101T123456.json.gz
│   │   │   ├── event_20220102T123456.json.gz
│   │   │   └── ...
│   │   └── ...
│   └── ...
├── indexes/
│   ├── customers.json.gz
│   ├── contacts.json.gz
│   ├── opportunities.json.gz
│   └── ...
```

### `insert` operations

The following diagram describes inserts into the model:

```mermaid
graph TD
  subgraph "Insert Operation"
    Insert[Insert Operation]
  end

  subgraph "Storage"
    CurrentState["Current State Storage"]
    EventLog["Event Log Storage"]
    Indexes["Indexes Storage"]
  end

  Insert -->|Add New Event| EventLog
  EventLog -->|Acknowledge Insert| Insert
  Insert -->|Update Index| Indexes
  Indexes -->|Acknowledge Update| Insert
  Insert -->|Update Current State| CurrentState
  CurrentState -->|Acknowledge Update| Insert
```

### `update` operations

The following diagram describes updates to entities in the model:

```mermaid
graph TD
  subgraph "Update Operation"
    Update[Update Operation]
  end

  subgraph "Storage"
    CurrentState["Current State Storage"]
    EventLog["Event Log Storage"]
    Indexes["Indexes Storage"]
  end

  Update -->|Add Update Event| EventLog
  EventLog -->|Acknowledge Update| Update
  Update -->|Update Index| Indexes
  Indexes -->|Acknowledge Update| Update
  Update -->|Update Current State| CurrentState
  CurrentState -->|Acknowledge Update| Update
```

### `select` operations

The following diagram describes read operations:

```mermaid
graph TD
  subgraph "Read Operation"
    Read[Read Operation]
  end

  subgraph "Storage"
    CurrentState["Current State Storage"]
    EventLog["Event Log Storage"]
    Indexes["Indexes Storage"]
  end

  Read -->|Read Index| Indexes
  Indexes -->|Document ID| Read
  Read -->|Fetch Current Document| CurrentState
  Read -->|Fetch Event Stream| EventLog
  Read -->|Reconstruct Document| Read
```

### `delete` operations

The following diagram describes delete operations to entities in the model:

```mermaid
graph TD
  subgraph "Delete Operation"
    Delete[Delete Operation]
  end

  subgraph "Storage"
    CurrentState["Current State Storage"]
    EventLog["Event Log Storage"]
    Indexes["Indexes Storage"]
  end

  Delete -->|Add Delete Event| EventLog
  EventLog -->|Acknowledge Delete| Delete
  Delete -->|Update Index| Indexes
  Indexes -->|Acknowledge Update| Delete
  Delete -->|Update Current State| CurrentState
  CurrentState -->|Acknowledge Update| Delete
```
