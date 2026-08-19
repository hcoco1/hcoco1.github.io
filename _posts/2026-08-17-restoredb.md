
---
title: Restore a database in Postgres
author: hcoco1
date: 2026-07-11 10:00:00 +0200
categories: [GIS, Tutorial]
tags: [sql, postgres]
render_with_liquid: false
pin: true
toc: true
---
# PostgreSQL Restore with `pg_restore`: A Practical Quick Guide

I recently restored a PostgreSQL database from a `.backup` file on an Ubuntu server. The process exposed several common problems that are useful to understand because the commands alone are easy to forget.

The important lesson is to understand **where the backup is, which PostgreSQL user is being used, and which backup format you have**.

---

## 1. First: identify the backup format

PostgreSQL backups are commonly restored in two different ways.

### Plain SQL backup

If the backup is a text `.sql` file:

```bash
psql -U postgres -d database_name -f backup.sql
```

### Custom/TAR backup

If the backup was created with PostgreSQL's custom format, use:

```bash
pg_restore -U postgres -d database_name backup.backup
```

For example:

```bash
pg_restore -U postgres -d tompkins tompkins.backup
```

A custom backup can also have extensions such as:

```text
.backup
.dump
.tar
```

The extension alone does not guarantee the format, but `pg_restore` is the appropriate tool for PostgreSQL custom/TAR archives.

---

# 2. The basic restore workflow

The general process is:

```text
Backup file
    ↓
Make sure PostgreSQL can access the file
    ↓
Create target database
    ↓
Run pg_restore
    ↓
Check tables
```

For example:

```bash
sudo -u postgres createdb tompkins
sudo -u postgres pg_restore -d tompkins /path/to/tompkins.backup
```

Then verify:

```bash
sudo -u postgres psql -d tompkins
```

Inside `psql`:

```sql
\dt
```

---

# 3. My specific situation

The backup was originally on Windows:

```text
C:\Users\ivana\Downloads\python\tompkins.backup
```

But PostgreSQL was running on an Ubuntu laptop.

That means the Ubuntu PostgreSQL server cannot simply use the Windows path.

The correct architecture was:

```text
Windows
C:\Users\ivana\Downloads\python\tompkins.backup
            │
            │ scp
            ↓
Ubuntu
/home/admin/gis/python/tompkins.backup
            │
            ↓
PostgreSQL
tompkins database
```

I copied the file to Ubuntu with:

```bash
scp "/mnt/c/Users/ivana/Downloads/python/tompkins.backup" \
    admin@192.168.1.65:/home/admin/gis/python/
```

---

# 4. PostgreSQL authentication: the first error

Initially I tried:

```bash
pg_restore -U postgres -d tompkins /home/admin/gis/python/tompkins.backup
```

PostgreSQL returned:

```text
FATAL: Peer authentication failed for user "postgres"
```

The reason is that Ubuntu was using **peer authentication**.

I was logged into Linux as:

```text
admin
```

but trying to connect to PostgreSQL as:

```text
postgres
```

With peer authentication, PostgreSQL checks that the operating-system user matches the PostgreSQL user.

So this:

```text
Linux user:       admin
PostgreSQL user:  postgres
```

does not match.

The correct command was:

```bash
sudo -u postgres pg_restore -d tompkins /path/to/tompkins.backup
```

The important part is:

```bash
sudo -u postgres
```

It means:

> Run this command as the Linux `postgres` user.

This is different from:

```bash
sudo pg_restore
```

which runs the command as `root`.

---

# 5. File permissions: the second error

After fixing authentication, I got:

```text
could not open input file
"/home/admin/gis/python/tompkins.backup":
Permission denied
```

This was a different problem.

The `postgres` user could authenticate to PostgreSQL, but it could not access the backup file inside my `/home/admin/...` directory.

The simple solution was to copy the backup to `/tmp`:

```bash
sudo cp /home/admin/gis/python/tompkins.backup /tmp/tompkins.backup
```

Then make it readable:

```bash
sudo chmod 644 /tmp/tompkins.backup
```

And restore it:

```bash
sudo -u postgres pg_restore \
    -d tompkins \
    /tmp/tompkins.backup
```

This avoids changing permissions on the whole `/home/admin/gis/python` directory.

---

# 6. The final working command

The final restore command was:

```bash
sudo -u postgres pg_restore -d tompkins /tmp/tompkins.backup
```

Then verify the result:

```bash
sudo -u postgres psql -d tompkins
```

Inside PostgreSQL:

```sql
\dt
```

Other useful commands:

```sql
\l
```

List databases.

```sql
\dn
```

List schemas.

```sql
\dt
```

List tables.

```sql
\d table_name
```

Describe a table.

```sql
\q
```

Exit PostgreSQL.

---

# 7. If the target database already exists

If `tompkins` already exists and you want to replace its contents, you have two common approaches.

### Option A — Drop and recreate

From Ubuntu:

```bash
sudo -u postgres dropdb tompkins
sudo -u postgres createdb tompkins
sudo -u postgres pg_restore -d tompkins /tmp/tompkins.backup
```

This gives you a clean database.

**Be careful:** `dropdb` destroys the existing database.

### Option B — Let `pg_restore` clean objects

For a suitable backup:

```bash
sudo -u postgres pg_restore \
    --clean \
    --if-exists \
    -d tompkins \
    /tmp/tompkins.backup
```

`--clean` tells `pg_restore` to drop database objects before recreating them.

`--if-exists` prevents errors when an object does not already exist.

---

# 8. Useful `pg_restore` commands to remember

### See what's inside the backup

Before restoring:

```bash
pg_restore -l /tmp/tompkins.backup
```

This is extremely useful because it lets you inspect the archive without restoring it.

### Restore normally

```bash
sudo -u postgres pg_restore -d tompkins /tmp/tompkins.backup
```

### Restore and clean existing objects

```bash
sudo -u postgres pg_restore \
    --clean \
    --if-exists \
    -d tompkins \
    /tmp/tompkins.backup
```

### Restore with verbose output

```bash
sudo -u postgres pg_restore \
    --verbose \
    -d tompkins \
    /tmp/tompkins.backup
```

`--verbose` is useful when you want to see what PostgreSQL is doing.

### Restore only the schema

```bash
sudo -u postgres pg_restore \
    --schema-only \
    -d tompkins \
    /tmp/tompkins.backup
```

### Restore only the data

```bash
sudo -u postgres pg_restore \
    --data-only \
    -d tompkins \
    /tmp/tompkins.backup
```

---

# 9. The troubleshooting pattern to remember

When a restore fails, don't immediately change random PostgreSQL settings. Identify **which layer failed**.

### Error 1

```text
Peer authentication failed
```

Think:

```text
Who am I?
Who am I trying to connect as?
```

Check:

```bash
whoami
```

If you are `admin` and need PostgreSQL's `postgres` account:

```bash
sudo -u postgres pg_restore ...
```

---

### Error 2

```text
Permission denied
```

Think:

```text
Can the postgres Linux user read the backup file?
```

A simple solution:

```bash
sudo cp backup /tmp/backup
sudo chmod 644 /tmp/backup
```

Then:

```bash
sudo -u postgres pg_restore -d database /tmp/backup
```

---

### Error 3

```text
No such file or directory
```

Think:

```text
Am I using the correct filesystem?
```

A Windows path such as:

```text
C:\Users\ivana\Downloads\python\tompkins.backup
```

is not automatically the same environment as the Ubuntu server.

If the PostgreSQL server is on another machine, first transfer the backup:

```bash
scp backup admin@server:/path/
```

Then restore it **on the server**.

---

# 10. The mental model

The most useful thing to remember is not a single command. Remember these four questions:

```text
1. What format is my backup?
       ↓
   .sql → psql
   custom/TAR → pg_restore

2. Where is the backup?
       ↓
   PostgreSQL server must be able to access it

3. Which PostgreSQL user am I using?
       ↓
   sudo -u postgres ... when peer authentication requires it

4. Does that Linux user have permission to read the file?
       ↓
   Move/copy the backup to a readable location if necessary
```

For this particular case, the final workflow was:

```bash
# 1. Copy backup from Windows to Ubuntu
scp "/mnt/c/Users/ivana/Downloads/python/tompkins.backup" \
    admin@192.168.1.65:/home/admin/gis/python/

# 2. Make it accessible to postgres
sudo cp /home/admin/gis/python/tompkins.backup /tmp/tompkins.backup
sudo chmod 644 /tmp/tompkins.backup

# 3. Restore
sudo -u postgres pg_restore \
    -d tompkins \
    /tmp/tompkins.backup

# 4. Verify
sudo -u postgres psql -d tompkins
```

That sequence is worth understanding because it covers the three problems encountered here: **remote file location, PostgreSQL authentication, and Linux file permissions**.
