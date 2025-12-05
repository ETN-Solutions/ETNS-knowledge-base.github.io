---
layout: home
parent: Oracle
title: DB Maintainance
---

Good question. Yes — there *is* official/documentation-level guidance (and widely accepted practice) that you should **check which container (CDB or PDB) you are connected to** before running operations that change tablespace or datafiles. Here’s a **mini-“documentation snippet”** you can use (or adapt) as internal policy — using publicly documented behavior and queries from Oracle Database.

---

## 📄 Documentation / Recommended Pre-Check Before Expanding Tablespaces

**### Why check container first**

* In a multitenant database environment (CDB + PDB), each container (root or pluggable) maintains its own data dictionary and its own set of tablespaces/datafiles. ([Oracle Documentation][1])
* Running a tablespace or datafile operation against the wrong container can lead to unintended modifications (e.g. altering CDB SYSTEM when you intended PDB, or vice versa), which may corrupt data isolation or break the PDB setup.

**### How to check current container (CDB or PDB)**

If using SQL*Plus or similar:

```sql
-- this returns name of current container
SELECT SYS_CONTEXT('USERENV','CON_NAME') AS CURRENT_CONTAINER FROM DUAL;

-- optionally also check CON_ID
SELECT SYS_CONTEXT('USERENV','CON_ID') AS CURRENT_CONTAINER_ID FROM DUAL;
```

([Oracle Documentation][1])

Or in SQL*Plus you can use shortcuts:

```sql
SHOW CON_NAME;   -- shows container name (e.g. CDB$ROOT or PDB name)
SHOW CON_ID;     -- shows container ID (1 = root, 2 = PDB$SEED, ≥3 = other PDBs)
```

([DBASolved][2])

**### How to check that you are on the right container before modifying tablespace**

1. Check current container name/ID (as above).
2. If you intend to modify the CDB (root), ensure `CON_NAME = 'CDB$ROOT'` (or `CON_ID = 1`).
3. If you intend to modify a PDB, first run:

   ```sql
   ALTER SESSION SET CONTAINER = <your_pdb_name>;
   SHOW CON_NAME;
   ```

   to confirm you are inside the correct PDB. ([Oracledbwr | by HariPrasath Rajaram][3])

**### Example “Pre-Expansion Check” Template**

```sql
-- 1. Identify container
SELECT SYS_CONTEXT('USERENV','CON_NAME') AS CONTAINER,
       SYS_CONTEXT('USERENV','CON_ID') AS CON_ID
FROM   DUAL;

-- 2. (Optional) List datafiles for SYSTEM (or relevant tablespace)
SELECT FILE_NAME, BYTES/1024/1024 AS MB
FROM   DBA_DATA_FILES
WHERE  TABLESPACE_NAME = 'SYSTEM';
```

If the `CONTAINER` is not the intended one (root vs PDB), then **switch session or reconnect** appropriately before applying any ALTER DATAFILE / ALTER TABLESPACE commands.

**### Use Official Views to Discover All Containers / PDBs**

When connected to root (CDB$ROOT), you can use:

```sql
SELECT NAME, CON_ID, OPEN_MODE
FROM   V$PDBS;
```

or

```sql
SELECT NAME, CON_ID, DBID
FROM   V$CONTAINERS;
```

to list all pluggable databases (PDBs) under the CDB. ([Oracle Documentation][4])

---

## ✅ Use This as Internal SOP / Policy

When performing tablespace changes (add datafile, resize, autoextend) in a multitenant setup:

1. Always **check container name/ID**.
2. Ensure you are in the correct container — root (CDB) for global/system-wide changes, or the correct PDB for PDB-specific changes.
3. Log the container name and datafile paths before executing — serves as audit and rollback reference.
4. Only after verification proceed with `ALTER TABLESPACE ...` or `ALTER DATABASE ...`

---

If you like, I can draft a **full standard operating procedure (SOP)** for you — formatted as a document (Markdown) listing all steps, checks, and safety precautions for tablespace expansion in a multitenant Oracle environment.

[1]: https://docs.oracle.com/database/122/ADMIN/administering-a-cdb-with-sql-plus.htm?utm_source=chatgpt.com "Administering a CDB with SQL*Plus"
[2]: https://www.dbasolved.com/2013/07/where-am-i-at-cdb-vs-pdb-12-edition/?utm_source=chatgpt.com "Where am I at: CDB vs. PDB? - 12 edition - DBASolved"
[3]: https://oracledbwr.com/connecting-to-container-databases-cdb-and-pluggable-databases-pdb-in-oracle-database-12c-release-2-12-2/?utm_source=chatgpt.com "Connecting to Container Databases (CDB) and Pluggable ..."
[4]: https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/viewing-information-about-cdbs-and-pdbs-with-sql-plus.html?utm_source=chatgpt.com "Viewing Information About CDBs and PDBs with SQL*Plus"
