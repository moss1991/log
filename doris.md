```
docker pull apache/incubator-doris:build-env-1.3
```
# 1 创建用户
## 1.1 Root 用户登录与密码修改
```
mysql -h FE_HOST -P9030 -uroot
SET PASSWORD FOR 'root' = PASSWORD('your_password');
```

## 1.2 创建用户
```
CREATE USER 'test' IDENTIFIED BY 'test_passwd';
mysql -h FE_HOST -P9030 -utest -ptest_passwd
```
