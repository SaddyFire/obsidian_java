#####  MyBatis编程步骤是什么样的？

1. 创建SqlSessionFactory

2. 通过SqlSessionFactory创建SqlSession

3. 通过sqlsession执行数据库操作

4. 调用session.commit()提交事务

5. 调用session.close()关闭会话

