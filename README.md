# fence_ssh
Fencing agent that uses SSH, should only be used for testing.

The branch alternate_version contains a version of the agent that works using the nodename parameter instead (read the readme on that branch).

# How to Use
Place in /usr/sbin with execute permissions

Create stonith resources in Pacemaker

Can see usage with either fence_ssh --help or by running pcs stonith describe fence_ssh

Example usage with PCS:

_Ensure that pcmk_host_list is defined so that Pacemaker knows which resource can fence which node_

pcs stonith create stonith-one fence_ssh user=centos hostname=host1 password=Pa55w0rd sudo=true pcmk_host_list="host1"
pcs stonith create stonith-two fence_ssh user=centos hostname=host2 password=Pa55w0rd sudo=true pcmk_host_list="host2"
pcs constraint location add stonith-one-host-one stonith-one host1 INFINITY
pcs constraint location add stonith-two-host-two stonith-two host2 INFINITY

# memo

* 動作確認
```
/usr/sbin/fence_ssh -o metadata
```

* pacemakerと一緒に動かすときの注意点
  * pacemakerはリソースが停止するまで待ち続けるsystemd設定になっており、fenceが実行されるまで待機し続け、結局両系とも停止することになる
  * fence_sshと一緒に動かすときは、pacemakerのリソース停止を一定時間(以下の例だと5秒)だけ待ち、その後はsigkillさせることで、片系停止だけ実現できる
  * そもそも試験用なので、本番ではだめ  
```
$ vim /usr/lib/systemd/system/pacemaker.service
TimeoutStopSec=5s
SendSIGKILL=yes
```
