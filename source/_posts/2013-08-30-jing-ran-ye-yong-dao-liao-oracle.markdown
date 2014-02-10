---
layout: post
title: "竟然也用到了Oracle"
date: 2013-08-16 20:25
comments: true
categories: DataBase
---

#### Oracle初印象

曾经阿里的去O化搞得满城风雨，，随后很多企业也开始跟风去除Oracle，让Oracle这样的企业都为之咳嗽了一阵子。有兴趣的朋友可以看看王坚博士的文章。作为一个开源软件热爱者，首先接触到的一定是Mysql， 随后才是MongoDB,Redis,LevelDb。无论如何是怎么也想不到会去接触Oracle，就像我再也没有机会也不会去使用3.5英寸软盘的样子,因为这样会让我感觉到世界在倒退。

先知言：”存在即合理”。也就为以后的遇见埋下伏笔。果然，由于历史原因，庞大的业务体系下，我终于还是遇见了它，带着对伟大数据库敬仰的心探索了一把。说实话，当你习惯了简单的Mysql之类的思维之后，接触Oracle就有点纠结了。我愿意称其为接触另一种新思维方式。没有比这更值得鼓舞人心的动力了 ：）

{% img /images/2013/08/oracle.png %}

我做的事情是在游戏数据操作中做一个抽象的操作类。涉及框架代码的改造，这过程有点DT。我从事的工作大部分是C++开发这过程着实feel到了“生命太短，远离C++” 涉及到多重继承，虚析构函数，同个父类冲突，属性可见性,抽象类可用。Debug 了两天。还好有GDB这种家伙。
Oracle查询流程

简单描述下普通的Oracle建立连接以及查询的顺序,使用的是OCCI （Oracle的C++接口）:

```
//创建enviroment变量
Environment *env = Environment::createEnvironment( Environment::DEFAULT);
//创建连接
Connection *conn = env->createConnection( userName, password, connectString);
//创建 查询描述符
Statement *stmt = conn->createStatement( "SELECT blobcol FROM mytable");
//执行查询，返回结果集
ResultSet *rs = stmt->executeQuery();
//关闭查询，环境变量
stmt->closeResultSet(rs);
conn->terminateStatement(stmt);
env->terminateConnection(conn);
Environment::terminateEnvironment(env);
```

蛋蛋的疼

期间遇到了Oracle 好多奇形怪状的问题。尽管如此，我喜欢这样折腾自己 :) 是不是有点变态。

+ 创建环境变量类指明多线程使用。该任务需要多线程执行的，如果不特别声明多线程执行，如果不特别指明的话会奔溃，我纠结了一天就是因为没有指明这个常量。这在Mysql是没有见到的吧
+ 连接管理。Oracle的连接关闭很重要。查询任务完成后要及时关闭连接。如果不关闭连接和环境句柄，程序就会有很大的潜在bug,程序执行期间也会奔溃。这过程中自己写了连接管理模块。特别处理了多线程处理时的构造与析构。

以下是连接管理类的代码

```            
#include<string.h>
using namespace oracle::occi;
class COraclconn
{
    public :

        COraclconn (const std::string & user, const std:: string& password,
                const std:: string& dbname)
        {
            m_status = false;
            env = NULL;
            conn = NULL;
            stmt = NULL;
            env = Environment::createEnvironment( Environment::THREADED_MUTEXED);
            if (! env)
            {
                env = NULL;
                return;
            }
            conn = env-> createConnection(user, password, dbname);
            if (! conn)
            {
                conn = NULL;
                return;
            }
            m_status = true;
        }

        ~COraclconn ()
        {
            if ( conn && stmt)
                conn-> terminateStatement( stmt);
            if ( env && conn)
                env-> terminateConnection( conn);
            if ( env)
                Environment:: terminateEnvironment( env);
        }
        Statement * getstmt( const string& sql)
        {
            if (! m_status)
                return NULL;
            if ( stmt)
            {
                conn-> terminateStatement( stmt);
                stmt = NULL;

            }
            stmt = conn-> createStatement(sql);
            if (! stmt)
            {
                stmt = NULL;
                m_status = false;
            }
            return stmt;

        }
    protected :
        bool m_status;
        Environment *env ;
        Connection *conn ;
        Statement *stmt ;
};
```

还好我做的是封装SQL，对于其诡异的语法可以不用涉及到。相信每个热爱技术的人儿碰到写相对非业务的代码会感到充实愉快的吧。

Happy Hacking ：）

