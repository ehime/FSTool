FSTool
======

PHP 5/6 reimplementation of the FS Tool

Requires:

 * php --with-ldap
 * OpenSSL

Project Install:

 * composer install --dev


MYSQL:

 - Creating a generic users and database

```sql
    DROP DATABASE IF EXISTS fstools;
    CREATE DATABASE fstool;
    GRANT ALL ON fstool.* TO 'fstool'@'localhost' IDENTIFIED BY 'fstool';
    FLUSH PRIVILEGES;
```

 - Creating a primary users table

```sql
    DROP TABLE IF EXISTS users; 
    CREATE TABLE users (
        id          BIGINT       NOT NULL AUTO_INCREMENT PRIMARY KEY,
        login       VARCHAR(100) NOT NULL,
        email       VARCHAR(50)  NOT NULL,
        hash        VARCHAR(255) NOT NULL, ## Switching to BCRYPT
        created     TIMESTAMP    DEFAULT NOW(),
        modified    TIMESTAMP    DEFAULT NOW() ON UPDATE NOW()
    ) ENGINE = InnoDB DEFAULT CHARACTER SET = utf8;
```

 - Creating table for Applications

```sql
    DROP TABLE IF EXISTS applications; 
    CREATE TABLE applications (
        id          BIGINT       NOT NULL AUTO_INCREMENT PRIMARY KEY,
        name        VARCHAR(100) NOT NULL,
        created     TIMESTAMP    DEFAULT NOW(),
        modified    TIMESTAMP    DEFAULT NOW() ON UPDATE NOW()
    ) ENGINE = InnoDB DEFAULT CHARACTER SET = utf8;
```

 - Creating table for FSGroups

```sql
    DROP TABLE IF EXISTS fsgroups; 
    CREATE TABLE fsgroups (
        id          BIGINT       NOT NULL AUTO_INCREMENT PRIMARY KEY,
        name        VARCHAR(100) NOT NULL,
        aid         BIGINT       NOT NULL,
        created     TIMESTAMP    DEFAULT NOW(),
        modified    TIMESTAMP    DEFAULT NOW() ON UPDATE NOW()
    ) ENGINE = InnoDB DEFAULT CHARACTER SET = utf8;
```

 - Creating table for Filesystems

```sql
    DROP TABLE IF EXISTS fsgroups; 
    CREATE TABLE fsgroups (
        id          BIGINT       NOT NULL AUTO_INCREMENT PRIMARY KEY,
        name        VARCHAR(100) NOT NULL,
        fid         BIGINT       NOT NULL,  ## FSGroup ID
        oid         BIGINT       NOT NULL,  ## User ID
        gid         BIGINT       NOT NULL,  ## Group ID
        volume      varchar(100) NOT NULL,
        mount       varchar(100) NOT NULL,
        size        varchar(100) NOT NULL,
        created     TIMESTAMP    DEFAULT NOW(),
        modified    TIMESTAMP    DEFAULT NOW() ON UPDATE NOW()
    ) ENGINE = InnoDB DEFAULT CHARACTER SET = utf8;
```

 - Tracking (Don't implement yet plz)
 
```sql
    -- DROP DATABASE tracking;
    CREATE DATABASE tracking;

    USE tracking;

    -- DROP TABLE IF EXISTS login_tracking;
    CREATE TABLE login_tracking (
      user      VARCHAR(16), 
      host      VARCHAR(60), 
      ts        TIMESTAMP, 
      PRIMARY KEY (user, host)
    ) ENGINE = MyISAM;

    -- DROP PROCEDURE IF EXISTS login_trigger;

    DELIMITER //

    CREATE PROCEDURE login_trigger()
    SQL SECURITY DEFINER
    BEGIN
      INSERT INTO login_tracking (user, host, ts)
      VALUES (SUBSTR(USER(), 1, instr(USER(), '@') - 1), substr(USER(), instr(USER(), '@') + 1), NOW())
      ON DUPLICATE KEY UPDATE ts = NOW();
    END;

    //
    DELIMITER;

    -- REVOKE EXECUTE ON PROCEDURE tracking.login_trigger FROM '{dbname}'@'%';
    GRANT EXECUTE ON PROCEDURE tracking.login_trigger TO '{dbname}'@'%';
    
    ## Grant execute to all users who do not have Super
    SELECT CONCAT("GRANT EXECUTE ON PROCEDURE tracking.login_trigger TO '", user, "'@'", host, "';") AS query
      FROM mysql.user
     WHERE Super_priv = 'N';

    ## Activate stored proceedure by hooking to login trigger
    -- SET GLOBAL init_connect="";
    SET GLOBAL init_connect="CALL tracking.login_trigger()";
```