0. 配置环境
```bash
python3 -m venv ~/Documents/code/cmdbvenv
source ~/Documents/code/cmdbvenv/bin/activate
```

1. 动 mysql 服务, redis 服务,此处以 docker 为例
```bash
# mkdir ~/cmdb_db
# docker run -d  -p 3306:3306  --name mysql-cmdb -e MYSQL_ROOT_PASSWORD=Root_321  -v ~/cmdb_db:/var/lib/mysql mysql
docker rm -f mysql-cmdb redis
docker volume rm vecmdb_db

docker run -d  -p 3306:3306  --name mysql-cmdb -e MYSQL_ROOT_PASSWORD=Root_321  -v vecmdb_db:/var/lib/mysql mysql
docker run -d --name redis -p 6379:6379 redis
```

2. 配置
```bash
cp cmdb-api/settings.example.py cmdb-api/settings.py
```

3. 设置mysql
- 登录mysql
```bash
docker exec -it mysql-cmdb mysql -uroot -pRoot_321
```

- 设置sql_mode、创建数据库和用户
```sql
set global sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
set session sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';


create database cmdb;
create user cmdb@'%' identified by '123456';
grant all privileges on cmdb.* to 'cmdb'@'%';
flush privileges;
```

4. 导入基础数据
```bash
docker cp ./docs/cmdb.sql mysql-cmdb:/root/

# docker exec -it mysql-cmdb bash

# mysql -uroot -pRoot_321 cmdb < /root/cmdb.sql

docker exec -it mysql-cmdb sh -c "mysql -uroot -pRoot_321 cmdb < /root/cmdb.sql"
```

- 创建表
```bash
cd cmdb-api

source ~/Documents/code/cmdbvenv/bin/activate

flask db-setup 
flask common-check-new-columns 
flask cmdb-init-cache
```

5. 运行
- 后端服务
```bash
cd cmdb-api

source ~/Documents/code/cmdbvenv/bin/activate

flask run -h 0.0.0.0
```

- 前端服务
```bash
cd cmdb-ui
nvm use 14

npm install  --registry=https://registry.npmmirror.com
npm run serve
```

- worker
```bash
cd cmdb-api

source ~/Documents/code/cmdbvenv/bin/activate

celery -A celery_worker.celery worker -E -Q one_cmdb_async --autoscale=4,1 --logfile=one_cmdb_async.log -D

celery -A celery_worker.celery worker -E -Q acl_async --autoscale=2,1 --logfile=one_acl_async.log -D
```

- 关闭worker
```bash
ps auxww | grep 'celery' | awk '{print $2}' | xargs kill -9
```