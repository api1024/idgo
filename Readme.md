## 1. Overview [中文主页](Readme_zh.md)
[![Build Status](https://travis-ci.org/flike/idgo.svg?branch=master)](https://travis-ci.org/flike/idgo)

Idgo is a sequential id generator which can generate batch ids through MySQL transcation way. Its features as follows:

- The id generated by idgo is by order.
-  generate batch ids through MySQL transcation way, dose not increase the load of MySQL.
-  When idgo process crashed, admin can restart idgo, does not worry about generating repetitive ids.
-  Idgo talks with clients using redis protocol, developer can connect idgo using redis sdk.

Someone knows the resolution of generating id with MySQL:

```
REPLACE INTO Tickets64 (stub) VALUES ('a');
SELECT LAST_INSERT_ID();

```

The disadvantage of this resolution is that generates one id need query MySQL once. When generating id too quickly, leading MySQL overload. This is why I build this project to generating ids.

## 2. The architecture of idgo

Idgo talks with clients using redis protocol, developer can connect to idgo using redis sdk. Every Key will mapped to a table storing in MySQL, and the table only has one row data to record the id.

![idgo_arch](http://ww2.sinaimg.cn/large/6e5705a5gw1f2nz3bot3tj20qo0k0mxe.jpg)

Idgo only supports four commands of redis as follows:

- `SET key value`, set the initial value of id generator in idgo.
- `GET key`, get the value of key.
- `EXISTS key`, check the key if exist.
- `DEL key`, delete the key in idgo.

## 3. Install and use idgo

Install idgo following these steps:

```
	1. Install Go environment, the version of Go   
is greater than 1.3.
	2. Install godep. `go get github.com/tools/godep`. 
	3. git clone https://github.com/flike/idgo src/github.com/flike/idgo
	4. cd src/github.com/flike/idgo
	5. source ./dev.sh
	6. make
	7. set the config file.
 	8. run idgo. `./bin/idgo -config=etc/idgo.toml`

```
Set the config file(etc/idgo.toml):

```
#the address of idgo
addr="127.0.0.1:6389"
#log_path: /Users/flike/src 
log_level="debug"

[storage_db]
mysql_host="127.0.0.1"
mysql_port=3306
db_name="idgo_test"
user="root"
password=""
max_idle_conns=64

```

Examples:

```

#start idgo
➜  idgo git:(master) ✗ ./bin/idgo -config=etc/idgo.toml
2016/04/07 11:51:20 - INFO - server.go:[62] - [server] "NewServer" "Server running" "netProto=tcp|address=127.0.0.1:6389" req_id=0
2016/04/07 11:51:20 - INFO - main.go:[80] - [main] "main" "Idgo start!" "" req_id=0

#start a redis client to connecting idgo, and set/get the key.
➜  ~  redis-cli -p 6389
redis 127.0.0.1:6389> get abc
(integer) 0
redis 127.0.0.1:6389> set abc 100
OK
redis 127.0.0.1:6389> get abc
(integer) 101
redis 127.0.0.1:6389> get abc
(integer) 102
redis 127.0.0.1:6389> get abc
(integer) 103
redis 127.0.0.1:6389> get abc
(integer) 104

```

## 4. HA

When the idgo crashed, you can restart idgo and reset the key by increasing a fixed offset.

## 5. License

MIT 
