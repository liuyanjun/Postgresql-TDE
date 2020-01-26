# Postgresql-TDE
#### Postgresql transparent data encryption is not support officially by PG community. But serveral commercial or open source third party build TDE on Postgresql old version, my target is to build transparent data encryption feature on PG latest version (12.1).
### I do not re-invent the wheel since Cybertech, other company already implement this feature in PG 9.6, I plan to port this implementation into PG last version. (https://www.cybertec-postgresql.com/en/products/postgresql-transparent-data-encryption/)

### <img src="https://www.cybertec-postgresql.com/wp-content/uploads/2017/11/PostgreSQL-instance-level-encryption2.jpg"/>

# Development Plan
### To-Do
#### 2020.01.26
* TDE概要设计和实现设计
* AES实现算法
* PG TDE 配置
* TDE测试和验证

# High Level Introduction
### Transparent Database Encryption
### <img src="/"/>
### Advanced Encryption Standard
### <img src="https://raw.githubusercontent.com/oYo-Byte/img_libs/master/blog/165259_mERh_2910723.png"/>



# Known Issues
### This is whole database encryption implementation, user has no choice to choose which database is encrypted, which one is not encrypted. this is known limitation.

# References
### Website
* https://www.postgresql.org/
* https://www.cybertec-postgresql.com/
* https://oyo-byte.github.io/2018/10/25/postgresql_fde_encryption/
* http://momjian.us/
* https://billtian.github.io/digoal.blog/
### Books
* 《PostgreSQL数据库内核分析》
* 《PostgreSQL技术内幕》
