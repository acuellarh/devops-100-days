# Day 17: Install and Configure PostgreSQL

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

The Nautilus application development team has shared that they are planning to deploy one newly developed application on Nautilus infra in Stratos DC. The application uses PostgreSQL database, so as a pre-requisite we need to set up PostgreSQL database server as per requirements shared below:

PostgreSQL database server is already installed on the Nautilus database server.

a. Create a database user kodekloud_aim and set its password to B4zNgHA7Ya.

b. Create a database kodekloud_db7 and grant full permissions to user kodekloud_aim on this database.

Note: Please do not try to restart PostgreSQL server service.

## Context

What is PostgreSQL?
PostgreSQL (also called Postgres) is one of the most popular open-source relational database management systems (RDBMS) used in production environments. It stores data in tables with rows and columns, and uses SQL (Structured Query Language) to manage data.

Key Concepts

_Roles & Users in PostgreSQL_ : In PostgreSQL, users and roles are the same thing. A ROLE with login capability is what we call a "user". When you run CREATE USER, PostgreSQL internally creates a ROLE WITH LOGIN.

_psql CLI_ : psql is the command-line interface for PostgreSQL. It lets you run SQL commands and special meta-commands (prefixed with \) to manage the database server.

_The postgres system user_: When PostgreSQL is installed, it creates a special OS-level user called postgres. This user is the superadmin of the database server and is used to perform administrative tasks like creating users and databases.

Why No Restart is Needed
Unlike services such as nginx or sshd that require a restart to reload configuration, PostgreSQL applies user and database changes immediately and in real-time. This is critical in production — no downtime required.

## Solution steps

**Step 1 — Connect to PostgreSQL as the superuser**  

This switches to the `postgres` OS user and opens the `psql` terminal.

```
sudo -u postgres psql
```

**Step 2 — Create the database user with a password**

```
CREATE USER kodekloud_aim WITH PASSWORD 'B4zNgHA7Ya';
```
`CREATE USER` is shorthand for `CREATE ROLE ... WITH LOGIN`. It creates a role that can authenticate.

**Step 3 — Create the database**

```
CREATE DATABASE kodekloud_db7;
```

**Step 4 — Grant full permissions to the user on the database**

```
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db7 TO kodekloud_aim;
```

`ALL PRIVILEGES` on a database grants: CONNECT, CREATE (schemas), and TEMP privileges at the database level.

**Step 5 — Verify everything is set up correctly**

```
-- Check the user was created
\du kodekloud_aim

-- Check the database was created
\l kodekloud_db7

-- Exit psql
\q
```

## Command 1: `sudo -u postgres psql -c "\du kodekloud_aim"`

_The Command Itself_

| Part | Meaning |
|------|---------|
| `sudo` | Run as another user with elevated privileges |
| `-u postgres` | Specifically run as the `postgres` OS system user |
| `psql` | Open the PostgreSQL interactive terminal |
| `-c` | Execute a single command and exit (non-interactive) |
| `"\du kodekloud_aim"` | The psql meta-command to run (`\du` = describe user) |

The output
```
List of roles
   Role name   | Attributes | Member of 
---------------+------------+-----------
 kodekloud_aim |            | {}
```


| Column | Value | Meaning |
|--------|-------|---------|
| `Role name` | `kodekloud_aim` | The user/role you created exists ✅ |
| `Attributes` | *(empty)* | No special powers — not a superuser, can't create DBs, can't create other roles |
| `Member of` | `{}` | Not part of any group roles (empty array) |

What "Attributes" would look like if set
```
Superuser       → full admin power
Create DB       → can create databases
Create role     → can create other users
Replication     → used for DB replication

```

## Command 2: `sudo -u postgres psql -c "\l kodekloud_db7"`

### The Command Itself

Same structure as above, but `\l` means **list databases**, filtered to `kodekloud_db7`.

### The Output

```
      Name      |  Owner   | Encoding | Collate | Ctype  |     Access privileges      
----------------+----------+----------+---------+--------+----------------------------
 kodekloud_db7  | postgres | UTF8     | C.utf8  | C.utf8 | =Tc/postgres              +
                |          |          |         |        | postgres=CTc/postgres     +
                |          |          |         |        | kodekloud_aim=CTc/postgres
```

### Column by Column

| Column | Value | Meaning |
|--------|-------|---------|
| `Name` | `kodekloud_db7` | The database exists ✅ |
| `Owner` | `postgres` | The superuser owns it (you created it while logged in as postgres) |
| `Encoding` | `UTF8` | Character encoding — supports all international characters, emojis, etc. |
| `Collate` & `Ctype` | `C.utf8` | Rules for how text is sorted and classified (uppercase/lowercase detection, alphabetical order, etc.) |

---

### The `Access privileges` Column (most important!)

This uses a special notation: **`who=privileges/granted_by`**

```
=Tc/postgres
postgres=CTc/postgres
kodekloud_aim=CTc/postgres
```

| Entry | Who | Privileges | Granted by |
|-------|-----|-----------|------------|
| `=Tc/postgres` | *(empty = PUBLIC)* | `Tc` | `postgres` |
| `postgres=CTc/postgres` | `postgres` | `CTc` | `postgres` |
| `kodekloud_aim=CTc/postgres` | `kodekloud_aim` | `CTc` | `postgres` |

---

### What each privilege letter means

| Letter | Privilege | What it allows |
|--------|-----------|----------------|
| `C` | CREATE | Create new **schemas** inside the database |
| `T` | TEMP | Create **temporary tables** (exist only during a session) |
| `c` | CONNECT | **Connect/log in** to the database |

> Notice `C` (uppercase) = CREATE schemas, `c` (lowercase) = CONNECT. PostgreSQL is case-sensitive here!

---

### Breaking down each line

**`=Tc/postgres`** — The empty left side means **PUBLIC** (all users by default)
- Every user on the system can use TEMP and CONNECT by default
- This is PostgreSQL's default behavior

**`postgres=CTc/postgres`** — The superuser has all three privileges on its own database (expected)

**`kodekloud_aim=CTc/postgres`** — Your user has CREATE, TEMP, and CONNECT ✅
- This is the result of your `GRANT ALL PRIVILEGES ON DATABASE` command
- Granted by `postgres` (the superuser)

---

### 🧠 One Important Nuance

`GRANT ALL PRIVILEGES ON DATABASE` gives privileges at the **database level** (connect, create schemas, temp tables).
If `kodekloud_aim` also needs to read/write **tables inside** the database, you'd additionally need:

```sql
\c kodekloud_db7   -- connect to that database
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO kodekloud_aim;
```








