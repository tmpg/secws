# Lab 3: Transparent Data Encryption

In this lab, we will address the problem of protecting against data theft or loss of disks and backups by encrypting all database files.
Encryption of some sensitive data is a requirement in certain regulated environments. 

## Requirements

- [Lab 02: "Network Encryption"](../lab200/lab200_network_encryption.md) completed. 
- Session open to **secdb** with user **oracle**
- session open to **dbclient** with user **oracle**   

## Review findings from DBSAT

In [Lab 01, DBSAT](../lab100/lab100_dbsat.md) we had the following finding:

![](./images/dbsat_tde.png)

**Transparent Data Encryption** automatically encrypts data as it is stored and decrypts it upon retrieval. This protects sensitive data from attacks that bypass the database to read data files directly. Encryption keys may be stored in wallets on the database server itself, or stored remotely in Oracle Key Vault for improved security. The ENCRYPT_NEW_TABLESPACES parameter ensures that TDE tablespace encryption is applied to all newly created tablespaces.

## Configuring Transparent Data Encryption

A software keystore is a container that stores the Transparent Data Encryption master encryption key. 
Before you can configure the keystore, you first must define a location for it in the sqlnet.ora file. There is one keystore per database, and the database locates this keystore by checking the keystore location that you define in the sqlnet.ora file. You can create other keystores, such as copies of the keystore and export files that contain keys, depending on your needs. However, you must never remove or delete the keystore that you configured in the sqlnet.ora location, nor replace it with a different keystore. 

### Set a location for the Wallet (TDE keystore)

Run the following script from a terminal window to the secdb server:

```
[oracle@secdb lab03_tde]$ <copy>cd ~/HOL/lab03_tde/</copy>
```
```
[oracle@secdb lab03_tde]$ <copy>./tde10_wallet_loc.sh</copy>
Adding the following lines to sqlnet.ora:
ENCRYPTION_WALLET_LOCATION =
  (SOURCE =
    (METHOD = FILE)
    (METHOD_DATA =
      (DIRECTORY=$ORACLE_BASE/admin/$ORACLE_SID/tde_wallet)
    )
  )
```

### Creating the Keystore

Run the following script from a terminal window to the **secdb** server:

```
[oracle@secdb lab03_tde]$ <copy>./tde20_wallet_create.sh</copy>

SQL*Plus: Release 19.0.0.0.0 - Production on Sat May 9 01:18:27 2020
Version 19.6.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Connected.
SQL> --
SQL> -- create the keystore
SQL> --
SQL> administer key management
  2    create keystore '$ORACLE_BASE/admin/$ORACLE_SID/tde_wallet'
  3    identified by "MyWalletPwd#1";

keystore altered.

SQL>
SQL> --
SQL> -- open the keystore
SQL> --
SQL> administer key management
  2    set keystore open identified by "MyWalletPwd#1"
  3    container=all;

keystore altered.

SQL>
SQL> --
SQL> -- view keystore status
SQL> --
SQL> select * from gv$encryption_wallet;

   INST_ID WRL_TYPE
---------- --------------------
WRL_PARAMETER
--------------------------------------------------------------------------------
STATUS                         WALLET_TYPE          WALLET_OR KEYSTORE FULLY_BAC
------------------------------ -------------------- --------- -------- ---------
    CON_ID
----------
         1 FILE
/u01/oracle/db/admin/CONT/tde_wallet/
OPEN_NO_MASTER_KEY             PASSWORD             SINGLE    NONE     UNDEFINED
         1


SQL>
SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.6.0.0.0
```
As shown above, the wallet is created as password protected, but is still empty.

### Create a Master Key for pdb1

Run the following script from a terminal window to the **secdb** server:

```
[oracle@secdb lab03_tde]$ <copy>./tde30_create_keys.sh</copy>
SQL*Plus: Release 19.0.0.0.0 - Production on Sat May 9 01:22:59 2020
Version 19.6.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Connected.
SQL> --
SQL> -- create master encryption keys for CDB/PDBs and activate them
SQL> --
SQL> administer key management
  2    set key identified by "MyWalletPwd#1"
  3    with backup using 'backup'
  4    container=all;

keystore altered.

SQL>
SQL> --
SQL> -- view keystore status
SQL> --
SQL> select * from gv$encryption_wallet;

   INST_ID WRL_TYPE
---------- --------------------
WRL_PARAMETER
--------------------------------------------------------------------------------
STATUS                         WALLET_TYPE          WALLET_OR KEYSTORE FULLY_BAC
------------------------------ -------------------- --------- -------- ---------
    CON_ID
----------
         1 FILE
/u01/oracle/db/admin/CONT/tde_wallet/
OPEN                           PASSWORD             SINGLE    NONE     NO
         1


SQL>
SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.6.0.0.0
```

As shown above, the wallet is now open and a TDE master key has been created for PDB1.

### Configure The Wallet as **autologin** for Ease of Management

A `LOCAL AUTOLOGIN` keystore might be a good tradeoff between security and ease of management. Run the following script from a terminal window to the **secdb** server:

```
[oracle@secdb lab03_tde]$ <copy>./tde40_wallet_autologin.sh</copy>

SQL*Plus: Release 19.0.0.0.0 - Production on Sat May 9 01:26:01 2020
Version 19.6.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Connected.
SQL> --
SQL> -- make keystore LOCAL AUTO_LOGIN
SQL> --
SQL> administer key management
  2    create local auto_login keystore
  3    from keystore '$ORACLE_BASE/admin/$ORACLE_SID/tde_wallet'
  4    identified by "MyWalletPwd#1";

keystore altered.

SQL>
SQL> --
SQL> -- reset wallet from PASSWORD to AUTOLOGIN mode
SQL> --
SQL> administer key management
  2    set keystore close identified by "MyWalletPwd#1"
  3    container=all;

keystore altered.

SQL>
SQL> --
SQL> -- view keystore status
SQL> --
SQL> select * from gv$encryption_wallet;

   INST_ID WRL_TYPE
---------- --------------------
WRL_PARAMETER
--------------------------------------------------------------------------------
STATUS                         WALLET_TYPE          WALLET_OR KEYSTORE FULLY_BAC
------------------------------ -------------------- --------- -------- ---------
    CON_ID
----------
         1 FILE
/u01/oracle/db/admin/CONT/tde_wallet/
OPEN                           LOCAL_AUTOLOGIN      SINGLE    NONE     NO
         1


SQL>
SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.6.0.0.0
```

## Encrypting Existing Data Files

We can now encrypt existing and future tablespaces. 

### Encrypting Existing Data Files

Run the following script from a terminal window to the **secdb** server:

```
[oracle@secdb lab03_tde]$ <copy>./tde50_encrypt_ts.sh</copy>

SQL*Plus: Release 19.0.0.0.0 - Production on Sat May 9 01:29:25 2020
Version 19.6.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Connected.
SQL> alter session set container=pdb1;

Session altered.

SQL> select tablespace_name, encrypted from dba_tablespaces;

TABLESPACE_NAME                ENC
------------------------------ ---
SYSTEM                         NO
SYSAUX                         NO
UNDOTBS1                       NO
TEMP                           NO
USERS                          NO

SQL> -- create a new encrypted tablespace
SQL> create tablespace enc_data encryption using 'AES128' default storage(encrypt);

Tablespace created.

SQL> -- 12cR2 allows online encryption of existing user tablespaces
SQL> alter tablespace users encryption online encrypt;

Tablespace altered.

SQL> -- check which tablespaces have been encrypted
SQL> select tablespace_name, encrypted from dba_tablespaces;

TABLESPACE_NAME                ENC
------------------------------ ---
SYSTEM                         NO
SYSAUX                         NO
UNDOTBS1                       NO
TEMP                           NO
USERS                          YES
ENC_DATA                       YES

6 rows selected.

SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.6.0.0.0
```

### Encrypt Sensitive Credential Data in the Dictionary

It is not recommended to encrypt database internal objects such as the SYSTEM, SYSAUX, UNDO, or TEMP tablespaces using TDE tablespace encryption. You should focus TDE tablespace encryption on tablespaces that hold application data, not on these core components of the Oracle database.

Instead of encrypting the SYSTEM tablespace as a whole, one should encrypt sensitive credentials data in the dictionary. (By default, the credential data in the SYS.LINK$ and SYS.SCHEDULER$_CREDENTIAL system tables is simply obfuscated.) 
To do this, simply execute `ALTER DATABASE DICTIONARY ENCRYPT CREDENTIALS`. 

_No TDE license is required for this, however a keystore must exist, be open and one needs to have the SYSKM privilege._

Let’s do this in our PDB1 database. Run the following script from a terminal window to the **secdb** server:

```
[oracle@secdb lab03_tde]$ <copy>./tde60_encrypt_system.sh</copy>

SQL*Plus: Release 19.0.0.0.0 - Production on Sat May 9 01:49:13 2020
Version 19.6.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Connected.
SQL> alter session set container=pdb1;

Session altered.

SQL>
SQL> -- SYSKM is required to encrypt sensitive dictionary data
SQL> -- (and a keystore must exist and be open)
SQL> grant syskm to dba_debra;

Grant succeeded.

SQL>
SQL> connect dba_debra/MyDbPwd#1@pdb1 as syskm
Connected.
SQL> set echo on
SQL>
SQL> -- check encryption status of sensitive data in the dictionary
SQL> select * from DICTIONARY_CREDENTIALS_ENCRYPT;

ENFORCEM
--------
DISABLED

SQL>
SQL> -- encrypt sensitive data in the dctionary
SQL> ALTER DATABASE DICTIONARY ENCRYPT CREDENTIALS;

Database dictionary altered.

SQL>
SQL> -- check status again
SQL> select * from DICTIONARY_CREDENTIALS_ENCRYPT;

ENFORCEM
--------
ENABLED

SQL>
SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.6.0.0.0
```


### Inspect keys in the Wallet

You may want to inspect existing keys in the TDE wallet.
Run the following script from a terminal window to the **secdb** server:

```
[oracle@secdb lab03_tde]$ <copy>./tde70_inspect_keys.sh</copy>

SQL*Plus: Release 19.0.0.0.0 - Production on Sat May 9 01:50:04 2020
Version 19.6.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Connected.
SQL> alter session set container=cdb$root;

Session altered.

SQL> select pdb.name pdb, e.key_id,
  2  to_char(e.creation_time,'DD-MON-YY HH24:MI:SS') created,
  3  to_char(e.activation_time,'DD-MON-YY HH24:MI:SS') activated,
  4  e.creator ,
  5  e.activating_pdbname acted_from
  6  from v$encryption_keys e, v$pdbs pdb
  7  where pdb.con_id=e.con_id and pdb.dbid = e.activating_pdbuid
  8  order by pdb.name desc, created ;

PDB        KEY_ID                                                CREATED            ACTIVATED          CREATOR     ACTED_FROM
---------- ----------------------------------------------------- ------------------ ------------------ ------------ ------------
PDB1       AXjCNiSR+k+Lv5WK+roTwMQAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  08-MAY-20 23:23:00 08-MAY-20 23:23:00 SYSKM       PDB1

SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.6.0.0.0
[oracle@secdb lab03_tde]$

```

### Rotating TDE Master Keys

Occasionally you may need to replace the TDE master encryption key by a new one and re-encrypt all local keys at the tablespace or table levels (with the new key). This operation is called rotating the key. Rotate the master encryption key only if it was compromised or as per the security policies of the organization (for example once a year). This process deactivates the previous TDE master encryption key.

Do not perform a rotation operation of the master key concurrently with an online tablespace rekey operation.

If you want to rotate the TDE master key, run the following script from a terminal window to the **secdb** server:

```
[oracle@secdb lab03_tde]$ <copy>./tde80_rotate_keys.sh</copy>

SQL*Plus: Release 19.0.0.0.0 - Production on Sat May 9 01:56:13 2020
Version 19.6.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Connected.
SQL> administer key management set keystore open force keystore identified by "MyWalletPwd#1" container=all;

keystore altered.

SQL> administer key management set encryption key
  2  using tag 'newkey'
  3  force keystore identified by "MyWalletPwd#1"
  4  with backup using 'pre_newkey'
  5  container=all;

keystore altered.

SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.6.0.0.0
```

The “with backup” clause means that the **previous keystore** is backed up before the new TDE key is added:

```
[oracle@secdb lab03_tde]$ cd $ORACLE_BASE/admin/$ORACLE_SID/tde_wallet
[oracle@secdb tde_wallet]$ ls -l

-rw------- 1 oracle oinstall 8408 Mar  6 12:19 cwallet.sso
-rw------- 1 oracle oinstall 2555 Mar  6 12:09 ewallet_2019030611095013_backup.p12
-rw------- 1 oracle oinstall 5467 Mar  6 12:19 ewallet_2019030611192096_pre_newkey.p12
-rw------- 1 oracle oinstall 8347 Mar  6 12:19 ewallet.p12
```

The presence of cwallet.sso indicates an **autologin wallet**.

This completes the **Transparent Data Encryption** lab. You can continue with [Lab 4: Database Vault](../lab400/lab400_data_redaction.md)


## Acknowledgements ##

- **Authors** - Adrian Galindo, PTS LAD & François Pons, PTS EMEA - Database Product Management - May 2020.
- **Credits** - This lab is based on materials provided by Oracle Database Security Product Management.