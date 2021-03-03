# 用户

## crud用户

1. 创建用户

   ```mysql
   CREATE USER 'username'@'host' IDENTIFIED BY 'password';
   ```

   ``` mysql
   # host='%' 从任意主机登录
   CREATE USER 'username'@'%' IDENTIFIED BY 'password';
   ```

   

2. 删除用户

   ```mysql
   DROP USER 'username'@'host';
   ```

## 密码

1. 设置用户密码

   ``` mysql
   SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
   ```

   设置当前登录用户密码

   ``` mysql
   SET PASSWORD = PASSWORD("newpassword");
   ```

# 权限

## 授权

``` mysql
GRANT privileges ON databasename.tablename TO 'username'@'host'
```

取值说明：

- `privileges`：`SELECT`、`INSERT`、`UPDATE`、`ALL` 等等
- `databasename` 和 `databasename`：如果要授予所有数据库或/和表则可用 `*` 来表示

``` mysql
GRANT SELECT ON test.user TO 'pig'@'%';
GRANT SELECT ON test.* TO 'pig'@'%';
GRANT ALL ON *.* TO 'pig'@'%';
```

注意:

用以上命令授权的用户不能给其它用户授权，如果**想让该用户可以授权**，用以下命令:

```mysql
GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
```

## 撤权

``` mysql
REVOKE privilege ON databasename.tablename FROM 'username'@'host';
```

- <u>*todo: 注意撤销用户权限的语句需要和授予权限的语句相对应？不对应不能撤销？*</u>

