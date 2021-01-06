# 说明

## 数据库授权

登陆管理员账号,创建 exporter 用户

```mysql
mysql>
Query OK, 0 rows affected (0.05 sec)

mysql> GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'exporter'@'%';
Query OK, 0 rows affected (0.01 sec)

mysql>
mysql>
Query OK, 0 rows affected (0.00 sec)

CREATE USER 'exporter'@'%' IDENTIFIED BY 'Auth@12!34';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%' WITH MAX_USER_CONNECTIONS 3;
GRANT SELECT ON performance_schema.* TO 'exporter'@'%';
commit;
FLUSH PRIVILEGES;
```

## 部署

进入到 `overlays` 目录下, 修改 secret.yaml 对应的 MySQL 地址 和 密码

```bash
kustomize build . | kubectl apply -f -
```

prometheus 采集到指标需要稍等一会

## grafana

模版id为7362
