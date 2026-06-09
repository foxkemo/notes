在 Spring 框架中，`TransactionTemplate` 是**编程式事务管理**的核心工具。

与常用的 `@Transactional` 注解（声明式事务）不同，`TransactionTemplate` 允许你在代码中**精确地控制事务的边界**。在处理高并发、大批量数据的优惠券分发系统时，它往往比注解更合适。

以下是深度解析及实战应用：

---

### 一、 为什么要用 `TransactionTemplate`？（对比 `@Transactional`）

在你的优惠券分发场景（处理 5000 条数据）中，`@Transactional` 有几个明显的痛点：

1.  **事务粒度过大（长事务问题）**：
    `@Transactional` 只能作用于整个方法。如果在方法里先解析了半天 JSON、调了多次 Redis，最后才写数据库，那么数据库连接会被占用很长时间。这在高并发下会迅速耗尽连接池。
2.  **无法精细化控制**：
    如果你想在一个方法里，部分代码走事务，部分代码不走事务（比如发 MQ 就不应该在事务里），注解做不到。
3.  **AOP 失效问题**：
    同一个类中方法互相调用时，`@Transactional` 会失效。

**`TransactionTemplate` 的优势：**
*   **粒度极细**：你可以只给那几行 `INSERT` 语句加事务。
*   **无 AOP 陷阱**：它是普通的对象调用，不存在失效问题。
*   **灵活性**：可以根据逻辑判断手动回滚。

---

### 二、 核心用法示例

#### 1. 注入与初始化
```java
@Service
public class CouponService {
    // 注入事务管理器
    @Autowired
    private TransactionTemplate transactionTemplate;

    public void distribute(List<UserCouponDO> list) {
        // 非数据库操作：逻辑检查、数据组装（不占用事务）
        checkData(list);

        // 编程式事务开始
        transactionTemplate.execute(status -> {
            try {
                // 数据库操作 1
                userCouponMapper.insertBatch(list);
                // 数据库操作 2
                couponTemplateMapper.updateStock(list.size());
                return true;
            } catch (Exception e) {
                // 手动标记回滚
                status.setRollbackOnly();
                log.error("分发失败，执行回滚", e);
                return false;
            }
        });
        
        // 事务完成后操作：发送通知 MQ（不占用事务）
        sendNotice();
    }
}
```

---

### 三、 在你的“优惠券分发”项目中的实战演练

结合你之前的批量插入逻辑，使用 `TransactionTemplate` 优化**长事务**：

```java
@Component
public class CouponExecuteDistributionConsumer {

    private final TransactionTemplate transactionTemplate;
    private final UserCouponMapper userCouponMapper;
    private final CouponTemplateMapper couponTemplateMapper;

    // ... 构造函数 ...

    public void onMessage(CouponTemplateDistributionEvent event) {
        // 1. 准备数据 (非事务执行)
        List<UserCouponDO> userCouponDOList = prepareData(event);

        // 2. 数据库批量操作 (编程式事务，快速提交)
        Boolean success = transactionTemplate.execute(status -> {
            try {
                // 批量插入
                userCouponMapper.insertBatch(userCouponDOList);
                // 扣减数据库库存
                int rows = couponTemplateMapper.decrementStock(event.getTemplateId(), userCouponDOList.size());
                if (rows <= 0) {
                    status.setRollbackOnly(); // 库存不足，手动回滚
                    return false;
                }
                return true;
            } catch (Exception e) {
                status.setRollbackOnly(); // 异常回滚
                return false;
            }
        });

        // 3. 事务完成后处理 (非事务执行)
        if (Boolean.TRUE.equals(success)) {
            // 只有数据库成功了，才去写 Redis 缓存或发成功通知
            syncToRedis(event);
        } else {
            // 记录失败日志
            saveFailLog(event);
        }
    }
}
```

---

### 四、 避坑与进阶建议

1.  **返回值处理**：
    `transactionTemplate.execute` 有返回值。如果操作不需要返回，可以使用 `TransactionCallbackWithoutResult`（已废弃）或直接用 lambda 返回 `null`。
2.  **传播行为与隔离级别**：
    `TransactionTemplate` 也可以配置。如果需要特殊的隔离级别：
    ```java
    TransactionTemplate tt = new TransactionTemplate(transactionManager);
    tt.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
    tt.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
    ```
3.  **异常捕获**：
    在 `execute` 内部，一定要 `try-catch` 并在 `catch` 里显式调用 `status.setRollbackOnly()`，否则事务可能不会按预期回滚。

### 总结

*   **`@Transactional`**：适合逻辑简单、全方法都要么成功要么失败的场景。
*   **`TransactionTemplate`**：适合**高并发**、**大批量数据**、**逻辑复杂**且需要严格控制事务耗时的场景。

在优惠券秒杀和分发这种**性能敏感型**系统中，推荐优先使用 `TransactionTemplate` 来缩短事务持有的时间（缩短长事务），从而提升系统整体的吞吐量。