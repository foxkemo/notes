SQL（Structured Query Language）根据功能的不同，通常分为五大核心类别。作为 Java 后端开发人员，理解这些分类有助于你在设计数据库架构、编写 DAO 层代码以及配置数据库权限时更加得心应手。

---

## 1. DDL (Data Definition Language) - 数据定义语言

用于定义或修改数据库的**结构（Schema）**。它操作的是表、索引、视图等“容器”本身。

- **核心指令：** `CREATE`, `ALTER`, `DROP`, `TRUNCATE`。
    
- **特点：** * 通常具有**隐式提交**（执行后自动 Commit，无法回滚）。
    
    - 在 Java 中，常用于数据库迁移工具（如 Flyway 或 Liquibase）。
        
- **示例：** `CREATE TABLE users (...);`
    

---

## 2. DML (Data Manipulation Language) - 数据操作语言

用于对表中的**数据**进行增、删、改。

- **核心指令：** `INSERT`, `UPDATE`, `DELETE`。
    
- **特点：** * 这是业务代码（Service 层）操作最频繁的部分。
    
    - **支持事务控制**，可以回滚。
        
- **示例：** `UPDATE orders SET status = 1 WHERE id = 100;`
    

---

## 3. DQL (Data Query Language) - 数据查询语言

专门用于从数据库中**检索**数据。虽然有时被归类为 DML，但由于其复杂性，通常独立出来。

- **核心指令：** `SELECT`。
    
- **常见子句：** `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT`。
    
- **特点：** 不改变数据库内容，只负责读取。
    

---

## 4. DCL (Data Control Language) - 数据控制语言

用于定义数据库的**访问权限和安全级别**。

- **核心指令：** `GRANT` (授权), `REVOKE` (回收权限)。
    
- **业务场景：** DBA 给你的 Java 应用账号分配权限（例如只允许 `SELECT` 和 `UPDATE`，禁止 `DROP`）。
    
- **示例：** `GRANT SELECT ON mydb.* TO 'user'@'localhost';`
    

---

## 5. TCL (Transaction Control Language) - 事务控制语言

用于管理数据库事务，确保 ACID 特性。

- **核心指令：** `COMMIT`, `ROLLBACK`, `SAVEPOINT`。
    
- **特点：** * 在使用 `FOR UPDATE` 或 `FOR SHARE` 时，TCL 决定了锁何时释放。
    
    - 在 Spring 框架中，通常由 `@Transactional` 注解底层自动处理这些指令。
        

---

## 总结对比表

|**缩写**|**全称**|**操作对象**|**常用命令**|**事务支持**|
|---|---|---|---|---|
|**DDL**|Data Definition|结构 (表/库)|`CREATE`, `DROP`, `ALTER`|否 (自动提交)|
|**DML**|Data Manipulation|行数据|`INSERT`, `UPDATE`, `DELETE`|是|
|**DQL**|Data Query|行数据|`SELECT`|仅锁定读相关|
|**DCL**|Data Control|权限|`GRANT`, `REVOKE`|否|
|**TCL**|Transaction Control|事务状态|`COMMIT`, `ROLLBACK`|-|

在你的短链接系统中，设计 `link_mapping` 表结构属于 **DDL**，用户点击时更新访问计数属于 **DML**，而你通过管理后台查看分析报表则属于 **DQL**。

你目前是在整理面试题库，还是在为项目编写技术方案文档？