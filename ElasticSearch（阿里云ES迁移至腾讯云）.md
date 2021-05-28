# ElasticSearch（阿里云ES迁移至腾讯云）

###### 源端：阿里云ES（OSS）

###### 目的端：腾讯云ES（COS）

#### 1.使用阿里云OSS中创建一个新的空桶

#### 2.阿里云Kibana上命令行创建ES快照仓库，并挂载至OSS桶

```
PUT _snapshot/my_backup/
{
    "type": "oss",
    "settings": {
        "endpoint": "http://oss-cn-hangzhou-internal.aliyuncs.com",
        "access_key_id": "xxxx",
        "secret_access_key": "xxxxxx",
        "bucket": "xxxxxx",
        "compress": true,
        "chunk_size": "500mb",
        "base_path": "/"
    }
}
```

#### 3.阿里云Kibana上为所有索引创建快照，保存至ES快照仓库，并在OSS上查看之前新建桶里是否有快照信息

```
PUT _snapshot/my_backup/snapshot_1
```

#### 4.使用腾讯云COS创建空桶，并使用MSP工具将阿里OSS中快照数据同步至COS

#### 5.腾讯云Kibana上命令行创建ES快照仓库，并挂载至COS桶

```
PUT _snapshot/my_backup
{
    "type": "cos",
    "settings": {
        "app_id": "xxxxxxx",
        "access_key_id": "xxxxxx",
        "access_key_secret": "xxxxxxx",
        "bucket": "xxxxxx",
        "region": "ap-shanghai",
        "compress": true,
        "chunk_size": "500mb",
        "base_path": "/"
    }
}
```

#### 6.腾讯云Kibana上命令行查询快照状态

```
GET _snapshot/my_backup/snapshot_1
```

#### 7.状态正常的话，执行以下命令进行数据还原，还原需要屏蔽系统默认索引，不然还原会报错

```
POST _snapshot/my_backup/snapshot_1/_restore 
{"indices":"*,-.monitoring*,-.security*,-.kibana*","ignore_unavailable":"true"}
```

#### 8.查看某个索引的数据状态，进行复核

```
GET index/_recovery/
```

查看所有索引的恢复状态

```
GET /_recovery/
```

###### 注：为避免相关报错，桶命名、ES快照仓库命名及“base_path”的路径尽可能保持一致。