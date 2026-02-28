## ビルド方法

```sh
docker build -t mysql-toydb-engine .
```

```sh
docker run -d --name mysql-toydb \
  -e MYSQL_ROOT_PASSWORD=root \
  -p 3306:3306 \
  mysql-toydb-engine
```

## 動作チェック

```sh
mysql -u root -p -h 127.0.0.1 -P 3306
```

```sql
-- 自作のStorage Engineが正しく認識されているか確認
SHOW PLUGINS;

CREATE DATABASE test;

CREATE TABLE test.t1 (id INT, name VARCHAR(100)) ENGINE=TOYDB;
INSERT INTO test.t1 VALUES (1, 'hello');
SELECT * FROM test.t1;
```
