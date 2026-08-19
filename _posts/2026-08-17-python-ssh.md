---
title: PostgreSQL Through SSH
description: Quick Guide: Python → PostgreSQL Through SSH
categories: [Python]
tags: [ssh, sql, postgres]
---

# Quick Guide: Python → PostgreSQL Through SSH

## 1. Architecture

Your setup is:

```text
Windows PC
    │
    ▼
WSL2
    │
    │ SSH tunnel
    │ localhost:15432
    ▼
Ubuntu laptop
    │
    │ localhost:5432
    ▼
PostgreSQL 18.6
    │
    ▼
tompkins database
```

The important point:

* **PostgreSQL port:** `5432`
* **SSH tunnel port in WSL2:** `15432`
* `15432` is only the local endpoint of the tunnel.

---

## 2. Verify PostgreSQL on Ubuntu

On Ubuntu:

```bash
sudo -u postgres psql -c "SHOW port;"
```

Expected:

```text
5432
```

PostgreSQL is listening on:

```text
127.0.0.1:5432
```

---

## 3. Create the SSH tunnel

From **WSL2**:

```bash
ssh -L 15432:127.0.0.1:5432 admin@192.168.1.65
```

Keep this terminal open.

The syntax is:

```text
-L LOCAL_PORT:REMOTE_HOST:REMOTE_PORT
```

So:

```text
15432:127.0.0.1:5432
  │        │          │
  │        │          └── PostgreSQL on Ubuntu
  │        └───────────── Ubuntu's localhost
  └────────────────────── WSL2 local port
```

---

## 4. Test the tunnel with `psql`

Open a **second WSL2 terminal**:

```bash
psql -h 127.0.0.1 -p 15432 -U postgres -d tompkins
```

Then:

```sql
SELECT version();
```

You successfully tested this with:

```text
PostgreSQL 18.6
```

Exit:

```sql
\q
```

---

## 5. Set up Python

In WSL2:

```bash
mkdir -p ~/python-postgres
cd ~/python-postgres

python3 -m venv .venv
source .venv/bin/activate

pip install psycopg2-binary
```

---

## 6. Python connection

`test_connection.py`:

```python
import psycopg2

conn = psycopg2.connect(
    host="127.0.0.1",
    port=15432,
    dbname="tompkins",
    user="postgres",
    password="YOUR_PASSWORD"
)

print("Connected to PostgreSQL!")

cur = conn.cursor()

cur.execute("SELECT version();")

result = cur.fetchone()

print(result[0])

cur.close()
conn.close()
```

Run:

```bash
python3 test_connection.py
```

You successfully obtained:

```text
Connected to PostgreSQL!
PostgreSQL 18.6 ...
```

---

## 7. What is happening?

Python connects to:

```text
127.0.0.1:15432
```

SSH forwards that connection to:

```text
Ubuntu 127.0.0.1:5432
```

PostgreSQL receives it normally.

So Python does **not** directly connect to:

```text
192.168.1.65:5432
```

and PostgreSQL does **not** need to be exposed to the network.

---

## 8. Do you need to create the tunnel every time?

With the manual approach, **yes**.

Every time you need the connection:

```bash
ssh -L 15432:127.0.0.1:5432 admin@192.168.1.65
```

Then run Python in another terminal.

You can later simplify this with SSH configuration or background tunnels.

### Current workflow

```text
Terminal 1 — WSL2
ssh -L 15432:127.0.0.1:5432 admin@192.168.1.65


Terminal 2 — WSL2
cd ~/python-postgres
source .venv/bin/activate
python3 test_connection.py
```

This is the fundamental **Python → SSH tunnel → remote PostgreSQL** pattern.
