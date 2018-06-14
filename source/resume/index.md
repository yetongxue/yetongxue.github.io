mysql 赋给用户权限 grant all privileges on1、grant 权限1,权限2,…权限n on 数据库名称.表名称 to 用户名@用户地址 identified by ‘连接口令’;样例：mysql> GRANT ALL PRIVILEGES ON *.* TO 'webuser'@'192.168.33.50' IDENTIFIED BY 'webuser@yeahka.com';数据库连接        mysql -u root -pa120090024 -h127.0.0.1mysql -h192.168.33.50 -uwebuser -pwebuser@yeahka.com --default-character-set=utf8 -A biz数据导入source /tmp/t_cardbin.sql数据库基本操作1、创建数据库 并制定默认的字符集是utf8create database dbname default charset  utf8 collate utf8_general_ci;JDBC连接说明，举例：jdbc:mysql://192.168.10.107:3306/lepos?useUnicode=true&amp;characterEncoding=utf-8&amp;zeroDateTimeBehavior=convertToNull&autoReconnect=true一、useUnicode=true&amp;characterEncoding=utf-8 作用有如下两个方面：1. 存数据时：     数据库在存放项目数据的时候会先用UTF-8格式将数据解码成字节码，然后再将解码后的字节码重新使用GBK编码存放到数据库中。2.取数据时：     在从数据库中取数据的时候，数据库会先将数据库中的数据按GBK格式解码成字节码，然后再将解码后的字节码重新按UTF-8格式编码数据，最后再将数据返回给客户端。二、zeroDateTimeBehavior=convertToNull表示日期格式不正确是，把日期转换成null代替异常处理。三、jdbc:mysql://[host:port],[host:port].../[database][?参数名1][=参数值1][&参数名2][=参数值2]...几个重要的参数，如下表所示：参数名称参数说明缺省值最低版本要求user数据库用户名（用于连接数据库） 所有版本password用户密码（用于连接数据库） 所有版本useUnicode是否使用Unicode字符集，如果参数characterEncoding设置为gb2312或gbk，本参数值必须设置为truefalse1.1gcharacterEncoding当useUnicode设置为true时，指定字符编码。比如可设置为gb2312或gbkfalse1.1gautoReconnect当数据库连接异常中断时，是否自动重新连接？false1.1autoReconnectForPools是否使用针对数据库连接池的重连策略false3.1.3failOverReadOnly自动重连成功后，连接是否设置为只读？true3.0.12maxReconnectsautoReconnect设置为true时，重试连接的次数31.1initialTimeoutautoReconnect设置为true时，两次重连之间的时间间隔，单位：秒21.1connectTimeout和数据库服务器建立socket连接时的超时，单位：毫秒。 0表示永不超时，适用于JDK 1.4及更高版本03.0.1socketTimeoutsocket操作（读写）超时，单位：毫秒。 0表示永不超时03.0.1存储过程


-- 利用存储过程批量插入数据
create procedure scott(min INT,max INT)
    begin
    declare i int;
    set i=min;
    while i<max do
        insert into t_user_info (F_uid,F_passwd,F_state,F_username,F_real_name,F_create_time) Values (i,MD5('test_user_'+i),1,concat('test_user',i),'模拟用户',now());
        select concat("创建用户成功，uid=",i);
        set i = i+1;
    end while;
    end;
    $
注意:linux环境写存储过程时，因为';'为结束符，故要先修改结束符为'$'，命令：delimiter $
