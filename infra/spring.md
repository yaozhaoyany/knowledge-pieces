1. @Transactional 是什么？干嘛用的？
没错，这是 Spring 框架提供的声明式本地数据库事务。

作用：它保证了这个方法里的所有关系型数据库（PostgreSQL）操作，要么全部成功（Commit），要么全部失败回滚（Rollback）。这叫 ACID 中的原子性（Atomicity）。

如果不加会怎样？ 假设你的逻辑是：

orderRepository.save(order) （保存订单主体）
orderItemRepository.save(items) （保存订单里面的商品明细）
如果在执行第 2 步时，数据库连接突然断了，或者抛了一个空指针异常。如果没有 @Transactional，你在第 1 步存的订单由于已经执行，就会永远留在数据库里，变成一个孤立的、没有商品明细的“脏数据”或“半残数据”。 加了 @Transactional，Spring 只要捕捉到了运行时异常，就会立刻通知数据库执行 ROLLBACK，把第 1 步存的数据撤销，当作什么都没发生过。