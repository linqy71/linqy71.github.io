---
title: 安装python-rocksdb
date: 2023-10-12 13:48:39
tags: python; rocksdb;
categories: coding
copyright: 
---

# 安装rocksdb (twmht不需要这一步)

```
# latest version 8.6.7
git clone https://github.com/facebook/rocksdb.git
cd rocksdb
mkdir build
cd build
cmake ..
make -j
cd ..
export CPLUS_INCLUDE_PATH=${CPLUS_INCLUDE_PATH}${CPLUS_INCLUDE_PATH:+:}`pwd`/include/
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}${LD_LIBRARY_PATH:+:}`pwd`/build/
export LIBRARY_PATH=${LIBRARY_PATH}${LIBRARY_PATH:+:}`pwd`/build/
```

# 从源码安装python-rocksdb

因为用pip install python-rocksdb一直失败，用源码安装成功

```
git clone https://github.com/twmht/python-rocksdb.git --recursive -b pybind11
cd python-rocksdb
python setup.py install
```

如果import出现报错
```
undefined symbol _ZN7rocksdb22GetLengthPrefixedSliceEPKc 
```

原本使用gcc4.8.5，改为gcc7.5.0后就没问题了

faust-streaming-rocksdb 亲测不行。
```
pip install git+https://github.com/faust-streaming/python-rocksdb.git#egg=faust-streaming-rocksdb
```

# twmht python-rocksdb 文档 

[](https://twmht.github.io/python-rocksdb/)

# 简单使用

```
import pyrocksdb
db = pyrocksdb.DB()
opts = pyrocksdb.Options()
# for multi-thread
opts.IncreaseParallelism()
opts.OptimizeLevelStyleCompaction()
opts.create_if_missing = True
s = db.open(opts, '/path/to/db')
assert(s.ok())
# put
opts = pyrocksdb.WriteOptions()
s = db.put(opts, b"key1", b"value1")
assert (s.ok())
# get
opts = pyrocksdb.ReadOptions()
blob = db.get(opts, b"key1")
print (blob.data) # b"value1"
print (blob.status.ok()) # true
#delete
opts = pyrocksdb.WriteOptions()
s = db.delete(opts, b"key1")
assert(s.ok())
db.close()

```

# 列族的使用

```
import rocksdb

# 打开或创建一个RocksDB数据库
db_options = rocksdb.Options(create_if_missing=True)
column_options = rocksdb.Options()

# 创建一个RocksDB数据库，包含两个列族
db = rocksdb.DB("mydb_with_column_families", db_options, column_families=["cf1", "cf2"])

# 列族名称必须与上面的列表匹配
cf1 = db.column_family_handles["cf1"]
cf2 = db.column_family_handles["cf2"]

# 写入数据到列族 cf1
db.put(b"key1", b"value1", column_family=cf1)
db.put(b"key2", b"value2", column_family=cf1)

# 写入数据到列族 cf2
db.put(b"key1", b"valueA", column_family=cf2)
db.put(b"key2", b"valueB", column_family=cf2)

# 从列族 cf1 读取数据
value1_cf1 = db.get(b"key1", column_family=cf1)
value2_cf1 = db.get(b"key2", column_family=cf1)

# 从列族 cf2 读取数据
value1_cf2 = db.get(b"key1", column_family=cf2)
value2_cf2 = db.get(b"key2", column_family=cf2)

print("cf1 key1:", value1_cf1.decode("utf-8"))
print("cf1 key2:", value2_cf1.decode("utf-8"))

print("cf2 key1:", value1_cf2.decode("utf-8"))
print("cf2 key2:", value2_cf2.decode("utf-8")

# 关闭数据库
db.close()

```