#### 1.如何快速找出当前目录下最晚被修改过的文件？

答：个人使用ls -all查看当前目录下文件详情，题目要求可配合grep之类的命令达到要求，具体命令不清楚，find应该也可以达到要求

#### 2.已知某个进程的pid是6666,如何找到它当前打开了哪些文件？

答：lsof -i:6666查到进程，再根据具体服务查看服务详情

#### 3.发现端口8001被占用，如何找出是哪个进程占用了该端口？

答：lsof -i:8001查到进程

#### 4.假设有进程P持续向文件F写入数据，此时把文件F删除，进程P的写入会失败吗？磁盘占用是否会持续增加？为什么？

答：不会失败，会增加，常见情况是日志写入，原因个人不清楚

#### 5.有如下分行的数据：

|      32742 |
|      39013 |
|     481472 |
|     481658 |
|     481885 |
把格式转换为：32742,39013,481337,481472,481658,481885
写出你的办法
答：分行遍历，每行遍历到数字保存，不是数字不保存，每行遍历完加","，最后去掉最后一个逗号

#### 6.数据处理

答：开发中原始sql语句未涉及，常用的是库封装好的轮子

#### 7.为runner.py实现一个函数，检测是否有其他的runner.py进程在正在执行

答：函数中插入执行时写日志，查看日志文件

#### 8.如果有如下文件，记录了用户购买会员权益的订单记录...

答：
from datetime import datetime


def str2int(a, b):
​    a_date = datetime.datetime.strptime(a, '%Y%m%d')
​    b_date = datetime.datetime.strptime(b, '%Y%m%d')
​    c_date = a_date - b_date
​    return c_date.days


def f(date):
​    date = datetime.strptime(date, '%Y%m%d')
​    money = 0
​    with open('a.txt', 'r')as f:
​        line = f.readline()
​        while line:
​            user_info = line.strip().split(',')
​            cost = int(user_info[1])
​            start_day = datetime.strptime(user_info[2], '%Y%m%d')
​            end_day = datetime.strptime(user_info[3], '%Y%m%d')
​            if start_day <= date <= end_day:
​                money = money + cost / ((end_day - start_day).days + 1)
​            line = f.readline()
​    return money
补充：考虑这种情况，计算每天金额平均值向下取整a，
最后一天是cost-a*(end_day - start_day)值为b，
判断输入日期是否为最后一天来确定取a or b
