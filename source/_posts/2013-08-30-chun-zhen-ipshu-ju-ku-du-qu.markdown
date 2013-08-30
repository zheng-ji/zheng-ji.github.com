---
layout: post
title: "纯真IP数据库读取"
date: 2013-06-18 19:14
comments: true
categories: Programe
---
纯真IP数据库

参考链接：

[这个](http://lumaqq.linuxsir.org/article/qqwry_format_detail.html),[这个](http://www.pythonclub.org/python-flies/chunzhen-ip-database),还有[这个](http://demon.tw/programming/python-qqwry-dat.html)

以上文章写的很好了，我只是把我学习的过程记录下来，做知识积累。

做IP地址查询服务时，一定会听到QQ纯真数据库。当然你需要下载的。名为QQWry.Dat

结构
QWry.dat文件在结构上分为3块：
+ 文件头
+ 记录区
+ 索引区。

一般我们要查找IP时，先在索引区查找记录偏移，然后再到记录区读出信息。由于记录区的记录是不定长的，所以直接在记录区中搜索是不可能的。由于记录数比较多，如果我们遍历索引区也会是有点慢的，一般来说，我们可以用二分查找法搜索索引区，其速度比遍历索引区快若干数量级。
纯真数据库内容结构

+ 文件头：4字节开始索引偏移值+4字节结尾索引偏移值
+ 记录区： 每条IP记录格式 ==> IP地址[国家信息][地区信息]

对于国家记录，可以有三种表示方式：
* 字符串形式(IP记录第5字节不等于0×01和0×02的情况)，
* 重定向模式1(第5字节为0×01),则接下来3字节为国家信息存储地的偏移值
* 重定向模式(第5字节为0×02),

对于地区记录，可以有两种表示方式： 字符串形式和重定向

最后一条规则：重定向模式1的国家记录后不能跟地区记录

索引区： 每条索引记录格式 ==> 4字节起始IP地址 + 3字节指向IP记录的偏移值
索引区的IP和它指向的记录区一条记录中的IP构成一个IP范围。查询信息是这个
范围内IP的信息

代码不是我写的，网上有很多版本。这里纯粹是为了学习。
```
#!/usr/bin/python

from struct import *
import string

def ip2string( ip ):
    a = (ip & 0xff000000) >> 24
    b = (ip & 0x00ff0000) >> 16
    c = (ip & 0x0000ff00) >> 8
    d = ip & 0x000000ff
    return "%d.%d.%d.%d" % (a,b,c,d)

def string2ip( str ):
    ss = string.split(str, '.');
    ip = 0L
    for s in ss:
        ip = (ip << 8) + string.atoi(s)
    return ip;

class IpLocater :
    def __init__( self, ipdb_file ):
        self.ipdb = open( ipdb_file, "rb" )
        # get index address
        str = self.ipdb.read( 8 )
        (self.first_index,self.last_index) = unpack('II',str)
        self.index_count = (self.last_index - self.first_index) / 7 + 1

    def getString(self,offset = 0):
        if offset :
             self.ipdb.seek( offset )
        str = ""
        ch = self.ipdb.read( 1 )
        (byte,) = unpack('B',ch)
        while byte != 0:
            str = str + ch
            ch = self.ipdb.read( 1 )
            (byte,) = unpack('B',ch)
        return str

    def getLong3(self,offset = 0):
        if offset :
            self.ipdb.seek( offset )
        str = self.ipdb.read(3)
        (a,b) = unpack('HB',str)
        return (b << 16) + a

    def getAreaAddr(self,offset=0):
        if offset :
            self.ipdb.seek( offset )
        str = self.ipdb.read( 1 )
        (byte,) = unpack('B',str)
        if byte == 0x01 or byte == 0x02:
            p = self.getLong3()
            if p:
                return self.getString( p )
            else:
                return ""
        else:
             return self.getString( offset )

     def getAddr(self,offset ,ip = 0):
        self.ipdb.seek( offset + 4) 
        countryAddr = ""
        areaAddr = ""
        str = self.ipdb.read( 1 )
        (byte,) = unpack('B',str)
        if byte == 0x01:
            countryOffset = self.getLong3()
            self.ipdb.seek(countryOffset )
            str = self.ipdb.read( 1 )
            (b,) = unpack('B',str)
            if b == 0x02:
                countryAddr = self.getString( self.getLong3() )
                self.ipdb.seek( countryOffset + 4 )
            else:
                countryAddr = self.getString( countryOffset )
            areaAddr = self.getAreaAddr()
        elif byte == 0x02:
            countryAddr = self.getString( self.getLong3() )
            areaAddr = self.getAreaAddr( offset + 8 )
        else:
            countryAddr = self.getString( offset + 4 )
            areaAddr = self.getAreaAddr( )

        return countryAddr + "/" + areaAddr

    def output(self, first ,last ):
        if last > self.index_count :
            last = self.index_count
        for index in range(first,last):
            offset = self.first_index + index * 7
            self.ipdb.seek( offset )
            buf = self.ipdb.read( 7 )
            (ip,of1,of2) = unpack("IHB",buf)
            print "%s - %s" % (ip, self.getAddr( of1 + (of2 << 16) ) )

    def find(self,ip,left,right):
        if right-left == 1:
            return left
        else:
            middle = ( left + right ) / 2
            offset = self.first_index + middle * 7
            self.ipdb.seek( offset )
            buf = self.ipdb.read( 4 )
            (new_ip,) = unpack("I",buf)
            if ip <= new_ip :
                return self.find( ip, left, middle )
            else:
                return self.find( ip, middle, right )

     def getIpAddr(self,ip):
         index = self.find( ip,0,self.index_count - 1 )
         ioffset = self.first_index + index * 7
         aoffset = self.getLong3( ioffset + 4)
         address = self.getAddr( aoffset )
         return address                

if __name__ == "__main__" :
    ip_locater = IpLocater( "QQWry.Dat" )
    ip_locater.output( 100, 120 )
    ip = '59.64.234.174'
    ip = '58.38.139.229'
    address = ip_locater.getIpAddr( string2ip( ip ) )
    print "the ip %s come from %s" % (ip,address)
```
        
