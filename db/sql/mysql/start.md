

创建表
----------
```sql
CREATE TABLE IF NOT EXISTS user(
	id int(10) UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
	username VARCHAR(24) NOT NULL UNIQUE,
	password VARCHAR(8) NOT NULL DEFAULT '',
	age tinyint(3) UNSIGNED NOT NULL DEFAULT 0,
	create_at INT(10) UNSIGNED NOT NULL DEFAULT 0
)ENGINE=InnoDB DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```
  
常用命令   
----------  
  
