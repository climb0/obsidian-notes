## 前期准备
### Mysql库初始化
这里创建2张表来：task1、task2
```sql
CREATE TABLE `task1`  
(  
    `id`       INT(11)      NOT NULL AUTO_INCREMENT,  
    `biz_type` VARCHAR(64)  NOT NULL COMMENT '业务类型',  
    `biz_code` VARCHAR(256) NOT NULL COMMENT '业务码',  
    PRIMARY KEY (`id`)  
) ENGINE = INNODB  
  AUTO_INCREMENT = 1  
  DEFAULT CHARSET = utf8mb4;

CREATE TABLE `task2`  
(  
    `id`       INT(11)      NOT NULL AUTO_INCREMENT,  
    `biz_type` VARCHAR(64)  NOT NULL COMMENT '业务类型',  
    `biz_code` VARCHAR(256) NOT NULL COMMENT '业务码',  
    PRIMARY KEY (`id`)  
) ENGINE = INNODB  
  AUTO_INCREMENT = 1  
  DEFAULT CHARSET = utf8mb4;
```

