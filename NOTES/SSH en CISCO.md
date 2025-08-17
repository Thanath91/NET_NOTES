



```
conf t
hostname R-PS
ip domain name mgmt.net
username netadmin privilege 15 secret netadmin
crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 15
transport input ssh
exec-timeout 60 0
logging synchronous
login local
exit
do wr
```