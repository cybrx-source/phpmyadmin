# 🔍 MySQL Security & Export Configuration Checker

## 📁 Directory Configuration

| **Variable**         | **Value**                        |
|----------------------|----------------------------------|
| `@@datadir`          | `/var/lib/mysql/`               |
| `@@basedir`          | `/usr/`                         |
| `@@secure_file_priv` | `/var/lib/mysql-files/`         |

> 📌 **Note**: `secure_file_priv` restricts the directories where `LOAD DATA` and `SELECT ... INTO OUTFILE` operations can be performed.

---

## 🧪 Export Path Test

Check whether specific paths are allowed by `secure_file_priv`.

| **Path**           | **Can Export?**        |
|--------------------|------------------------|
| `/var/www/html`    | ❌ Tidak Bisa         |
| `/home/user`       | ❌ Tidak Bisa         |
| `/tmp`             | ❌ Tidak Bisa         |
| `/var/tmp`         | ❌ Tidak Bisa         |
| `/var/lib/mysql-files/` | ✅ Hanya ini yang diizinkan |

---

## 🔐 User & Privilege Info

| **Info**                 | **Value**                         |
|--------------------------|-----------------------------------|
| **Current User**         | `root@localhost`                 |
| **Current Database**     | `NULL`                           |
| **MySQL Version**        | `8.0.33`                         |
| **File Privilege**       | ✅ YES                            |
| **Can INTO OUTFILE**     | ✅ Only to `secure_file_priv` dir |

---

## 🛠️ Recommendations Based on Settings

### ⚠️ Case: `secure_file_priv IS NULL`
- **Behavior**: ❌ You cannot use `INTO OUTFILE` at all.
- **Fix**: Request admin to update `my.cnf`:
    ```ini
    [mysqld]
    secure_file_priv = /tmp/
    ```
    Then restart MySQL service.

---

### ✅ Case: `secure_file_priv = ''` (Empty String)
- **Behavior**: ✅ You can export to **any directory** (⚠️ risky).
- **Recommendation**: Limit to specific folder:
    ```ini
    secure_file_priv = /var/lib/mysql-files/
    ```

---

### 📂 Case: `secure_file_priv` has a value
- **Behavior**: ✅ Export only to specified directory.
- **Usage**:
    ```sql
    SELECT 'test' INTO OUTFILE '/var/lib/mysql-files/test.txt';
    ```

---

## 🧰 Example Export Commands

```sql
-- Export version to secure directory
SELECT @@version INTO OUTFILE '/var/lib/mysql-files/version.txt';

-- Export data to secure directory
SELECT CONCAT('User: ', USER(), ' on DB: ', DATABASE()) INTO OUTFILE '/var/lib/mysql-files/session_info.txt';


-- CEK DOCUMENT ROOT + LUAR DOCUMENT ROOT
SELECT @@datadir;
SELECT @@basedir;

SELECT 
    '/var/www/html' AS path,
    IF(@@secure_file_priv = '' OR @@secure_file_priv LIKE '%/var/www/html%', 'Mungkin Bisa', 'Tidak Bisa') AS export_possible
UNION ALL
SELECT 
    '/home/user' AS path,
    IF(@@secure_file_priv = '' OR @@secure_file_priv LIKE '%/home/user%', 'Mungkin Bisa', 'Tidak Bisa')
UNION ALL
SELECT 
    '/tmp' AS path,
    IF(@@secure_file_priv = '' OR @@secure_file_priv LIKE '%/tmp%', 'Mungkin Bisa', 'Tidak Bisa')
UNION ALL
SELECT 
    '/var/tmp' AS path,
    IF(@@secure_file_priv = '' OR @@secure_file_priv LIKE '%/var/tmp%', 'Mungkin Bisa', 'Tidak Bisa');

-- CEK PRIVILEGE
SELECT * FROM (
    -- Bagian 1: Basic Info
    SELECT 'basic_info' AS section, 'datadir' AS variable, @@datadir AS value
    UNION ALL SELECT 'basic_info', 'basedir', @@basedir
    UNION ALL SELECT 'basic_info', 'secure_file_priv', @@secure_file_priv
    UNION ALL SELECT 'basic_info', 'version', VERSION()
    
    -- Bagian 2: Directory Info
    UNION ALL SELECT 'directories', 'character_sets_dir', @@character_sets_dir
    UNION ALL SELECT 'directories', 'plugin_dir', @@plugin_dir
    UNION ALL SELECT 'directories', 'lc_messages_dir', @@lc_messages_dir
    
    -- Bagian 3: User Info
    UNION ALL SELECT 'user_info', 'current_user', CURRENT_USER()
    UNION ALL SELECT 'user_info', 'database', DATABASE()
    UNION ALL SELECT 'user_info', 'hostname', @@hostname
    
    -- Bagian 4: Status
    UNION ALL SELECT 'status', 'can_export', 
           CASE 
               WHEN @@secure_file_priv IS NULL THEN 'NO'
               ELSE 'YES'
           END
) AS all_info
ORDER BY section, variable;

-- CEK PRIV + IOF
SELECT 
    'CURRENT SETTING' AS info,
    @@secure_file_priv AS value,
    CASE 
        WHEN @@secure_file_priv IS NULL THEN '❌ TIDAK BISA INTO OUTFILE sama sekali'
        WHEN @@secure_file_priv = '' THEN '✅ BISA ke semua direktori'
        ELSE CONCAT('✅ BISA ke: ', @@secure_file_priv)
    END AS status
UNION ALL
SELECT 
    'YOUR PRIVILEGES',
    IF(FILE_PRIV = 'Y', 'HAS FILE PRIV', 'NO FILE PRIV'),
    IF(FILE_PRIV = 'Y', '✅ Bisa export', '❌ Tidak bisa export')
FROM mysql.user 
WHERE User = SUBSTRING_INDEX(CURRENT_USER(), '@', 1)
    AND Host = SUBSTRING_INDEX(CURRENT_USER(), '@', -1)
UNION ALL
SELECT 
    'RECOMMENDATION',
    CASE 
        WHEN @@secure_file_priv IS NULL THEN 'Minta admin ubah config'
        WHEN @@secure_file_priv = '' THEN 'Gunakan /tmp/ atau /home/'
        ELSE @@secure_file_priv
    END,
    CASE 
        WHEN @@secure_file_priv IS NULL THEN 'Butuh akses server'
        ELSE 'Langsung export ke path tersebut'
    END;
