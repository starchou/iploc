此IP文件，融合了 **Taobao Ip** 和 **GeoIP** 。使用 IANA ip 段。有 国家，区域，城市，运营商，时区，经度，维度 字段。

字节序使用 **little-endian**，使用其他字节序的主机需要自行处理。

[Wikipedia 操作系统和架构对应的字节序](https://en.wikipedia.org/wiki/Endianness#Endianness_and_operating_systems_on_architectures)

# 文件结构图示：

![文件图示](https://github.com/slene/iploc/raw/master/format.png)

# 文件结构依次为：

## 文件头

头12个字节，每4个是一个 uint32 数字。分别是：

|索引起始地址|数据起始地址 |索引IP数量  |
|------------|-------------|------------|
|6a 5f 2a 00 |c8 10 02 00  |58 eb 01 00 |

ps: 上面的12个字节示例，当文件里IP数据改变时会有变化。

## 文本区
每一段文本 string 都以字节 0x00 结束，

## 数据区
每 21 个字节为一段，其顺序为：

| flag      | 国家代码   | 区域       | 城市       | 运营商     | 时区       | 经度        | 维度        |
| :-------- | :--------- | :--------- | :--------- | :--------- | :--------- | :---------- | :---------- |
| uint8 - 1 | string - 2 | uint16 - 2 | uint32 - 4 | uint16 - 2 | uint16 - 2 | float32 - 4 | float32 - 4 |

* flag == 1 IANA保留地址
* flag == 2 已分配地址
* flag == 4 IANA未分配地址

当 country_code 全等于字节 0x00 时，或者，flag标识为保留地址或未分配地址。那么其后面的数据都是不存在的。

country_code 为国家代码 eg: CN

region, city, isp, timezone 都是连接到文本区的地址，此地址为绝对地址，从文件开始处起始。

longitude, latitude 则是 float32 的经纬度

## 索引区
每 4 个字节为一个ip范围起始数字地址，uint32，总共数量为 **索引IP数量**

0.0.0.0 ～ 255.255.255.255 之间的IP地址，索引区已经覆盖。

意味着，在索引里，匹配IP地址时，当一个IP地址大于等于IP A，小于他的下一个IP B时，那他就是匹配A的，从而计算出数据地址。

关于数据地址计算:

数据区里的分段数和索引区的是相同的，所以当前索引的地址对应的数据地址是：

> 数据地址 = 索引地址 / 4 * 21 + **数据起始地址**










