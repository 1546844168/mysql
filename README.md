    不求甚解的读书和不读有何区别
                                ———— 江峰
第1章 MySQL架构与历史
    
    1.2 并发控制
        1.读写锁（又称共享锁和排他锁）：都是行级锁。
            读锁是共享的，相互不阻塞。多个客户在同一时刻可以读取同一个资源,而互补干扰。读的时候不能修改数据。
            写锁是排他的，也就是说一个写锁会阻塞其他的写锁和读锁。在同一时刻，只有一个用户能进行写入，并防止其他用户读取正在写入的统一资源。
        2.锁粒度：尽量只锁定需要修改的部分数据，而不是所有的资源。锁定的数据量越小，则系统的并发程度越高。加锁本身也需要消耗资源。
            锁策略：就是在锁的开销和数据的安全性之间寻求平衡，这种平衡也会影响到性能。锁策略有表锁和行级锁等。
        3.表锁：是最基本的锁策略，也是开销最小的策略，它会锁定整张表，对表进行写操作（插入，删除，修改），都需要先获取写锁，并阻塞其他用户
                对该表的读写操作。只有没有写锁时，其他用户才能获取读锁，读锁之间是不会相互阻塞的。
                注意：写锁具有比读锁更高的优先级（写锁可以插入到锁队列中读锁的前面，反之读锁则不能插入到写锁的前面）。
        4.行级锁：行级锁可以最大程度的支持并发处理(同时也带来了最大的锁开销)，行级锁只在存储引擎层实现，而MySQL服务器层没有实现。服务器从完全不了解存储引擎中的锁实现。
    1.3 事务（ACID）：事务就是一组原子性的SQL查询，或者说一个独立的工作单元。事务内的语句，要不全部执行成功，要么全部执行失败。
                        一个实现了ACID的数据库，通常需要更高的性能。所以用户需要根据是否需要事务处理，来选择合适的存储引擎。
            原子性：事务是不可分割的最小的执行单元，一个事务中的所有操作要么全部执行成功，要么全部执行失败。
            一致性：数据库总是从一个一致性的状态转换到另外一个一致性的状态。
            隔离性：通常来说，一个事务所做的修改在最终提交以前，对其他事务来说是不可见的。
            持久性：一旦事务提交，则其所做的修改就会永久的被保存到数据库中。
        1.隔离级别：
            读未提交：事务中修改，即使没有提交，对其他事务也都是可见的。
            读已提交：一个事务从开始到提交之前，对其他事务是不可见的。它是一个事务，要等另一个事务提交后才能读取数据。它可能出现不可重复读。（大多数数据库的隔离级别，如SqlServer,Oracle等）
            可重复读：解决了脏读的问题，并保证了在同一个事务中中多次读取同样的记录的结果是一致的。（MySQL默认的隔离级别）
            串行化：强制事务串行执行，避免了幻读的问题。SERIALIZABLE会在读取的每一行数据上都加锁。（性能消耗很高）
        2.死锁：是指两个或多个事务对同一资源相互占用，并请求锁定对方占用的资源，从而导致恶性循环的现象。
                解决办法：死锁检测和死锁超时机制
                InnoDB处理死锁的办法是：将持有最少行级排它锁的事务进行回滚。
                锁的行为和顺序是和执行引擎有关系的，死锁发生以后，只有部分或完全回滚其中一个事务，才能打破事务。
        3.事务日志：事务日志可以帮助提高使用事务的效率。使用事务日志，存储引擎在修改表数据的时候只需要修改其内存拷贝，再将该修改行为持久化到硬盘中的事务日志中，
                    而不是每次都将数据本身持久化到磁盘。事务持久以后，内存中被修改的数据在后台可以慢慢的刷会磁盘。所以修改数据需要写两次磁盘。
        4.MySQL中的事务： 
            1.自动提交：MySQL默认采用自动提交（AUTOCOMMIT）模式，也就是说，如果不是显式的开启一个事务，则每个sql语句都被当做一个事务执行提交动作。
                        当set autocommit = 0时，所有的sql语句都是在一个事务当中，直到执行commit或rollback。该事务结束，同时又开启了另一个事务。
            2.在事务中混合使用存储引擎：MySQL服务器层是不管理事务的，事务是由下层的存储引擎实现的。所以在同一个事务中，使用多种存储引擎是不可靠的。
            3.隐式和显式锁定：InnoDB采用的是两阶段锁定协议。  
                            select ... lock in share mode; 加共享锁
                            select ... for update; 加排它锁
    1.4 多版本并发控制（MVCC）
    1.5 MySQL中的存储引擎:show table status查看表的状态信息;
        1.InnoDB存储引擎：InnoDB采用MVCC来支持高并发，并且实现了四个标准的隔离级别。
                        InnoDB表是基于聚族索引建立的，聚族索引对主键查询有很高的性能。
        2.MyISAM存储引擎:不支持事务，不支持行级锁，且崩溃后无法安全恢复
            MyISAM特性：
                加锁与并发：MyISAM对整张表加锁，而不是针对行。读取时对对需要读取的所有表加共享锁，写入时则对表加排它锁。
                           并在读取的时候也支持插入（称为并发插入）
                修复：MySQL可以手工或者自动执行检查和修复操作，执行表的修复可能会导致部分数据丢失，并且修复操作是非常慢的。
                索引特性：对于MyISAM表，即使是BLOB和TEXT等长字段，也可以基于前500个字符创建索引，MyISAM也支持全文索引，支持复杂查询。
                延迟更新索引键：如果开启了DELY_KEY_WRITE选项，在每次修改的时候，不会立即将修改的索引数据更新到磁盘，而是会写到内存中的键缓冲区。
                            清理缓冲区或关闭表的时候才会写入到磁盘，提高了写入的性能。
                MyISAM压缩表：支持索引，压缩表不能修改（除非先解压缩，在修改，再压缩）。可以减少磁盘IO,提升查询性能。
                MyISAM性能：最典型的性能问题是表锁的问题，如果发现所有的查询都长期处于 “locked” 的状态，那么毫无疑问表锁就是罪魁祸首。 
        3.MySQL内建的其他存储引擎
        4.第三方存储引擎
        5.选择合适的存储引擎：除非用到某些InnoDB不具备的特性，并且没有其他办法替代，否则都应该优先考虑使用InnoDB引擎。
                事务：如果需要事务，选择InnoDB，不需要事务，并且主要是INSERT 和 SELECT操作，那么MyISAM是不错的选择。
                备份：如果可以定期的关闭服务器进行备份，那么备份的因素可以忽略。反之，如果需要热备份，那么选择InnoDB引擎。
                崩溃恢复：MyISAM崩溃恢复后发生损坏的概率比InnoDB高的多，而且恢复速度也很慢，所以即时不需要支持事务，很多人也选择InnoDB，这是一个很重要的因素。
            日志型应用：对插入速度有很高的要求，可以考虑使用MyISAM，开销低，插入快。
            只读或者大部分情况下只读的表：读多写少的业务，如果不介意MyISAM的崩溃恢复，选用MyISAM是合适的。
                                        不要低估崩溃后恢复问题的重要性（MySIAM引擎是只将数据写到内存中，然后操作系统定期将数据刷到磁盘中）。
            订单处理：涉及到订单处理，那么支持事务就是必须选项。InnoDB是支持订单处理的最佳选择。
            电子公告牌和主题讨论论坛：如select count(*) from table;对MyISAM是比较快的，但对于其他的存储引擎可能都不行。
            大数据量：几个TB的数据量，需要合理的选择硬件，做好物理涉及，并未服务器的I/O瓶颈做好规划。
                    在这样的数据量下，如果选用MyISAM，如果崩溃了，那么进行数据恢复基本就是凉凉。
        6.转换表的存储引擎：
            1.ALTER TABLE：比如将myTable表的存储引擎换成InnoDB,
                        CREATE TABLE innodb_table like myTable;
                        ALTER TABLE innodb_table ENGINE = INNODB;
                        INSERT INTO innodb_table SELECT * FROM myTable;
                        该操作是按行将数据从这张表复制到另一张表中，需要执行很长的时间，在复制期间可能会消耗系统所有的I/O能力，
                        同时会在原表加上读锁。所以在繁忙的表上执行此操作需要小心。   
            2.导出与导入：使用mysqldump工具将数据导出到文件等操作。
            3.创建与查询(CREATE 和 SELECT)：创建一个新的存储引擎的表，然后将数据导入到新表。数据量大的话就需要分批处理。
                                            这样操作以后，新表就是原表的一个复制。如果有必要，可以在执行的过程中对原表加锁。
    1.8 总结：MySQL拥有分层的架构。上层是服务器层的服务和查询执行引擎，下层则是执行引擎。等等。。。（对于InnoDB来说，所有的提交都是事务）

第2章 MySQL基准测试

    2.1 为什么要有基准测试：对系统的性能做出一个大概的评估， TPS(每秒事务数)
    2.2 基准测试的策略：主要有两种，一种是对整个系统进行测试（集成式基准测试），二是单独测试MySQL(单组件式基准测试)。
        1.测试何种指标：
            1.吞吐量：指的是单位时间内的事务处理数。
            2.响应时间或延迟：这个指标用于测试任务所需的整体时间。计算出平均响应时间、最小响应时间、最大响应时间和所占百分比等。
            3.并发性：Web服务器的并发性也不等同于数据库的并发性，而仅仅表示回话存储机制能够处理多少数据的能力。
                      Web服务器的并发性更正确的度量指标，应该是在任意时间有多少同时发生的并发请求。Web服务器的高并发，一般也会导致数据库的高并发。
            4.可拓展性：
            归根结底，应该测试那些对用户来说最重要的指标。
     2.3 基准测试方法：
                测试中一些常见的错误，见书82页
            
第3章 服务器性能分析

    3.1 性能优化简介：性能即响应时间。数据库服务器的目的是执行SQL语句，所以它关注的应该是查询或者语句，如SELECT,UPDATE ,DELETE等。
                        数据库的性能用查询的响应时间来度量，单位是每个查询花费的时间。
        1.通过性能剖析进行优化：
            两种类型的性能剖析：基于时间的分析和基于等待的分析。
        2.理解性能剖析：
    3.2 对应用程序进行进行性能剖析：
    3.3 剖析MySQL查询：
        1.剖析服务器负载
        2.剖析单条查询：
            set profiling = 1; 打开测量服务器中运行的所有的语句的运行时间，默认关闭。
            show profiles; 显示SQL语句的运行时间。（会发现每条语句都会创建一个id）
            show profile for query [id]; 详细显示这条语句（id）的运行时间。
            
            show status本身也会创建一条临时表
        3.使用性能剖析
    3.4 诊断间歇性问题：比如系统偶尔停顿或者慢查询。
        1.使用 SHOW GLOBAL STATUS
        
第4章 Schema与数据类型优化
    
    4.1 选择优化的数据类型
            1.更小的通常更好：一般情况下，应该选择正确存储数据的最小数据类型。（占用更少的磁盘、内存、CPU缓存）
            2.简单就好：简单数据类型的操作通常需要更少的CPU周期,整形比字符型操作代价更低（因为字符集和校对规则使字符比较比整型比较要复杂）
                    例子1：应该使用MySQL内建的类型(date,time,dateTime)，而不是字符串来存储日期和时间。
                    例子2：应该使用整型来存储IP地址。
            3.尽量避免NULL：因为可为NULL的列使得索引，索引统计，和值都比较复杂。导致更难优化，把可为NULL的列改为非NULL能带来性能上的提升。
            
            在为列选择数据类型时：第一步，选择合适的大类型。数字、字符串、时间等。
                                 第二步，选择具体类型。很多MySQL的数据类型可以存储相同的数据，只是存储的范围，精度或占用磁盘和内存空间不同。
        1.整型类型：TINYINT,SMALLINT,MEDIUMINT,INT,BIGINT。分别占用8位、16、24、32、64位存储空间。
                    整数类型有可选的UNSIGNED属性，表示不允许负数。这大致可以使整数的上限大概提升一倍。
                  
                    有符号类型与无符号类型占用相同的存储空间，并具有相同的性能，因此根据实际情况选择合适的数据类型。
                    
                    MySQL可以为整数类型指定宽度：例如int(11),对大多数应用来说，是没有意义的：它不会限制值得合法范围，只是规定
                                                了MySQL的一些（例如MySQ命令行客户端）用来显示字符的个数。对于存储和计算来说，INT(1)和INT(20)来说是一样的。
        2.实数类型：实数是带有小数部分的数字。FLOAT,DOUBLE是浮点类型，DECIMAL类型（需要额外的空间和计算开销，要慎用）。  
        3.字符串类型：
            VARCHAR：存储可变长字符串，是最常见的字符串类型。它比定长类型更节省空间，因为它使用必要的空间（例如，越短的字符串使用越少的空间）。
                    VARCHAR需要1或2个额外字节记录字符串的长度
            CHAR：定长的，MySQL根据定义的字符串长度分配足够的空间。
                    CHAR适合存储很短的字符串或者所有值都接近同一个长度。
                    
                    通常情况下使用varchar(20)和varchar(255)保持'hello'占用的空间都是一样的，但使用长度较短的列却有巨大的优势。
                    较大的列使用更多的内存，因为MySQL通常会分配固定大小的内存块来保存值，这对排序或使用基于内存的临时表尤其不好。
                    同样的事情也会发生在使用文件排序或者基于磁盘的临时表的时候。
            BLOB和TEXT类型：都是为了存储很大的数据而设计的字符串数据类型，分别采用二进制和字符方式存储。
                            如果查询使用了BLOB或TEXT列并且需要使用隐士临时表，将会使用磁盘临时表。这会导致严重的性能开销。    
            使用枚举（ENUM）代替字符串类型：
        4.日期和时间类型
            DATETIME:精度为秒，默认情况下，MySQL以一种可排序的，无歧义的格式显示DATETIME值，与时区无关，使用8个字节的存储空间。例如：" 2018-11-14 10:49:00 "。
            TIMESTAMP：占4个字节的存储空间， 通常情况下，应该尽量使用TIMESTAMP而不是DATETIME，因为它比DATETIME占用空间更小。
        5.位数据类型：从技术来说都是字符串类型。
            1.BIT：最好不要使用
            2.SET：缺点是改变列的代价太高。
            3.在整数列上进行按位操作：
        6.选择标识符：     
            数据库中schema指的是数据库的组织和结构。如ORM自动生成的schema可能不会会存储任意的数据类型，所以要检查清楚。避免性能问题。
        7.特殊类型数据：某些数据的类型并不直接与内置类型一致。
    4.2 MySQL schema 设计中的陷阱
            太多的列：API进行工作时，会在服务器层和存储引擎层之间进行行缓冲格式拷贝数据，然后在服务器层将缓冲内容解码成各个列。字段太多的话，转换代价就会很高。
            太多的关联：一个粗略的经验法则，如果希望查询执行的快速且并发性好，单个查询最好在12个表以内做关联。但阿里巴巴开发手册，严禁超过三张表以上做关联。    
             
    4.3 范式和反范式 
        1.范式的优点很多，缺点是通常需要关联，这不但代价昂贵，且有可能使一些索引策略失效。
        2.反范式的schema因为所有的数据都在一张表里面，可以很好的避免关联。
    4.4 缓存表和汇总表
    4.5 加快ALTER TABLE操作的速度
        ALTER TABLE操作对于大表性能上是个问题。因为ALTER TABLE 是用新的结构创建一个新表，从旧表汇总查出所有数据插入到新表中，然后删除旧表。
  
第5章 创建高性能上的索引
            
         索引是存储引擎快速找到记录的一种数据结构，在MySQL中，首先在索引中找到对应值，然后在根据匹配到的索引记录找到对应的数据行。
     5.1 索引的类型
        1.B-Tree索引
            可以使用B-Tree索引查询的类型：
                全值匹配
                匹配最左前缀
                匹配列前缀
                匹配范围值
                精确匹配某一列并范围匹配另外一列
            B-Tree索引的限制
                1.如果不是按照索引的最左列开始查找，则无法使用索引。
                2.不能跳过索引中的列
                3.如果查询中有某个列的范围查询，则其右边所有的列都无法使用索引优化查询。
               
               根据索引查询时只能从第一个索引列开始依次次往后开始查询，所以建立索引是索引的列的顺序是非常重要的。
        2.哈希索引
            基于哈希表实现，只有精确匹配索引所有列的查询才有效。
            对于每一行数据，存储引擎都会对所有的索引列计算一个哈希吗，哈希吗是一个较小的值，并且不同键值的行计算出来的哈希吗是不一样的。
            哈希索引将所有的哈希吗存储在索引中，同时在哈希表中保存指向每个数据行的指针。
            
            因为索引只需存储对应的哈希值，所以索引的结构十分紧凑，这也让哈希索引查找的速度非常快。
          
          哈希索引的限制很多，比如只能进行等值比较查询。
        3.空间数据索引
        4.全文索引：是一种特殊类型的索引，它查找的是文本中的关键字词，而不是直接比较索引中的值。
                    全文索引更类似于搜索引擎做的事情，而不是简单的WHERE条件匹配。
     5.2 索引的优点
            索引可以让服务器快速定位到表的指定位置。B-Tree索引是按照顺序存储数据。               
                    
                    
                    
  一个人有两块手表就永远不知道时间。