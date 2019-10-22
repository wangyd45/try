#geo-Replication(gluster version 3.9+)

## 创建两块盘：gv3,gv4
A集群：
```
gluster volume create gv3 replica 2 transport tcp 10.252.205.78:/opt/wangyd/gv3/bk1 10.252.205.80:/opt/wangyd/gv3/bk2
gluster volume gv3
```
B集群：
```
gluster volume create gv4 replica 2 transport tcp 10.252.205.81:/opt/wangyd/gv4/bk1 10.252.205.82:/opt/wangyd/gv4/bk2
gluster volume gv4
```


## 所有从集群,创建用户和组
```
groupadd geogroup
useradd -g geogroup geoaccount
```

## 任意从集群执行命令,创建备份目录
```
gluster-mountbroker setup  /opt/wangyd/mountbroker-root geogroup
```

## 任意从集群执行命令,指定备份卷和用户(配置新的备份卷须重新配置)
```
gluster-mountbroker add gv4 geoaccount
gluster-mountbroker status
```

## 所有从集群重启glusterd服务(配置新的备份卷须重新执行)
```
service glusterd restart
```

## 配置主集群一台主机免密ssh到从集群所有主机
```
ssh-copy-id geoaccount@10.252.205.81
```

## 在主机群，创建gluster类型证书和秘钥
```
gluster-georep-sshkey generate
```

## 在主机群,创建session(配置新的备份卷须重新配置)
```
gluster volume geo-replication gv3 geoaccount@10.252.205.81::gv4 create push-pem
```

## 在刚才session中从集群主机(10.252.205.81)设置主从关系
```
/usr/libexec/glusterfs/set_geo_rep_pem_keys.sh geoaccount gv3 gv4
```

## mount主集群和从集群卷测试
```
mount -t glusterfs 10.252.205.78:gv3 /opt/wangyd/mountpath1/
mount -t glusterfs 10.252.205.81:gv4 /opt/wangyd/mountpath2/
```

## 在主集群上开启同步
```
gluster volume geo-replication gv3 geoaccount@10.252.205.81::gv4 start
gluster volume geo-replication gv3 geoaccount@10.252.205.81::gv4 status
#停止
gluster volume geo-replication gv3 geoaccount@10.252.205.81::gv4 stop
```






root      8187  4639 79 13:47 ?        00:00:39 rsync -aR0 --inplace --files-from=- --super --stats --numeric-ids --no-implied-dirs --existing --xattrs --acls --ignore-missing-args . -e ssh -oPasswordAuthentication=no -oStrictHostKeyChecking=no -i /var/lib/glusterd/geo-replication/secret.pem -p 22 -oControlMaster=auto -S /tmp/gsyncd-aux-ssh-5Ys1sy/9c7c00b27f29a6246783a0ae58c36bd4.sock --compress geoaccount@10.252.205.82:/proc/36535/cwd
root      8188  8187  0 13:47 ?        00:00:00 ssh -oPasswordAuthentication=no -oStrictHostKeyChecking=no -i /var/lib/glusterd/geo-replication/secret.pem -p 22 -oControlMaster=auto -S /tmp/gsyncd-aux-ssh-5Ys1sy/9c7c00b27f29a6246783a0ae58c36bd4.sock -l geoaccount 10.252.205.82 rsync --server -logDtpAXRze.LsfxC --super --stats --numeric-ids --existing --inplace --no-implied-dirs . /proc/36535/cwd
