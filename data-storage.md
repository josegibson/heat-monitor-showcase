## BLE Session Data Storage System

A critical feature of the application is its ability to persistently store data from each monitoring session. This is managed by the `SessionDataService`, which uses a `SessionStorageProvider` to abstract the underlying storage mechanism.

### Architecture & Features:

- **Automatic Session Management:** The `SessionDataService` automatically manages the lifecycle of a data collection session.
    - A new session **automatically begins** when a BLE connection is established.
    - The session **automatically ends** when the BLE connection is terminated.
    - The system can gracefully **resume an interrupted session** if the app restarts, ensuring data continuity.
- **Efficient SQLite Backend:** The default implementation, `SqliteSessionStorage`, uses a local SQLite database for fast, reliable, and persistent storage. The database schema is optimized with indexes for quick data retrieval.
- **Abstraction Layer:** The use of the `SessionStorageProvider` interface decouples the session management logic from the specific database implementation. This makes the system more modular and allows for easier testing and future expansion (e.g., adding a different type of database).
- **Stream-Based API:** The `SessionDataService` exposes streams (`currentSessionStream`, `newDataPointStream`) that allow the UI to reactively update as new data comes in or as the session state changes. This is a key part of the app's reactive architecture.
- **Automatic Data Cleanup:** The service includes functionality to automatically purge old sessions to manage storage space effectively.

### Database Schema:

The `SqliteSessionStorage` implementation uses the following database schema:

**`sessions` Table:**

```sql
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  device_id TEXT NOT NULL,
  device_name TEXT,
  start_time INTEGER NOT NULL,
  end_time INTEGER,
  data_point_count INTEGER NOT NULL DEFAULT 0,
  status TEXT NOT NULL DEFAULT 'active',
  created_at INTEGER NOT NULL
);
```

**`data_points` Table:**

```sql
CREATE TABLE data_points (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  session_id TEXT NOT NULL,
  sequence_number INTEGER NOT NULL,
  timestamp INTEGER NOT NULL,
  oil_reservoir_level REAL NOT NULL,
  heating_element_level REAL NOT NULL,
  oil_tank_temperature REAL NOT NULL,
  temp_pump_to_heater REAL NOT NULL,
  temp_heater_to_hx REAL NOT NULL,
  pump_flow_rate REAL NOT NULL,
  temp_heating_element_1 REAL NOT NULL,
  temp_heating_element_2 REAL NOT NULL,
  temp_heating_element_3 REAL NOT NULL,
  temp_heating_element_4 REAL NOT NULL,
  temp_before_hx REAL NOT NULL,
  ambient_temperature REAL NOT NULL,
  system_voltage REAL NOT NULL,
  system_current REAL NOT NULL,
  FOREIGN KEY (session_id) REFERENCES sessions (id) ON DELETE CASCADE
);
```