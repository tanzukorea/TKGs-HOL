## JumpBox Ubuntu VM 설치

TKC의 Node는 내부 IP를 가지고 있어 외부에서 접근이 되지 않습니다.
SupervisorControlPlaneVM은 외부에서 접근되며, SupervisorControlPlaneVM안에서 TKC Node 접속이 가능합니다.
이를 참고하여 JumpBox VM을 설정해 봅니다.

### SupervisorControlPlaneVM 네트워크 정보 확인
  - 연결된 Networks 확인
    * DVPG-~~~ : Workload Management Enable시 지정한 네트워크
      - IP Address(외부): 10.193.109.45
    * vm-domain~~~
      - IP Address(내부): 10.244.0.228
    ![ 그림 ](images/supervisor-vm-networks.png)

### JumpBox VM 생성시 아래 네트워크 정보로 기반으로 VM 생성
  - 연결된 Networks : SupervisorControlVM과 동일하게 셋팅
    * DVPG-~~~
      - IP Address(외부): 외부에서 접속가능한 SupervisorControlVM이 사용하는 대역에서 남는 거 지정
    * vm-domain~~~
      - IP Address(내부): SupervisorControlVM이 쓰는 내부용을 참조해서 남는 거 사용

### SupervisorControlPlaneVM 라우팅 테이블 확인
  - routetable 조회결과
```
root@423fec13114cfdad68a006c03993d295 [ /etc/systemd/network ]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.193.109.1    0.0.0.0         UG    0      0        0 eth0
10.96.0.0       10.244.0.225    255.255.254.0   UG    0      0        0 eth1
10.193.109.0    0.0.0.0         255.255.255.0   U     0      0        0 eth0
10.193.109.64   10.244.0.225    255.255.255.192 UG    0      0        0 eth1
10.244.0.0      10.244.0.225    255.255.240.0   UG    0      0        0 eth1
10.244.0.224    0.0.0.0         255.255.255.240 U     0      0        0 eth1
10.244.0.228    10.244.0.225    255.255.255.255 UGH   0      0        0 eth1
100.64.0.0      10.244.0.225    255.255.0.0     UG    0      0        0 eth1
root@423fec13114cfdad68a006c03993d295 [ /etc/systemd/network ]#
```

  - routetable 설정 정보
```
root@423fec13114cfdad68a006c03993d295 [ /etc/systemd/network ]# cat 10-eth1.network 
[Match]
Name = eth1

[Network]
Address = 10.244.0.228/28
DNS = 10.244.0.228
Domains = ~cluster.local

[Route]
Gateway = 10.244.0.225
Table = 200

[Route]
Gateway = 10.244.0.225
Destination = 10.244.0.228

[Route]
Gateway = 10.244.0.225
Destination = 10.96.0.0/23

[Route]
Gateway = 10.244.0.225
Destination = 10.244.0.0/20

[Route]
Gateway = 10.244.0.225
Destination = 10.193.109.64/26

[Route]
Gateway = 10.244.0.225
Destination = 100.64.0.0/16

root@423fec13114cfdad68a006c03993d295 [ /etc/systemd/network ]#
```

### JumpBox VM 라우팅 테이블 설정
VM 생성 직후 네트워크 접속이 안될수 있으니, vSphere Client UI에서 접근해서 아래 정보를 설정합니다.

  - SupervisorControlPlaneVM의 라우팅 테이블 정보를 참고하여 아래 포맷으로 ens19(내부용)의 설정을 수정합니다.
```
root@jumpbox:/etc/netplan# cat 50-cloud-init.yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        ens160:
            addresses:
            - 10.193.109.5/24
            gateway4: 10.193.109.1
            nameservers:
                addresses:
                - 10.192.2.10
                - 10.192.2.11
        ens192:
            addresses:
            - 10.244.0.235/28
            routes:
            - to: 10.244.0.235
            via: 10.244.0.225
            - to: 10.96.0.0/23
            via: 10.244.0.225
            - to: 10.244.0.0/20
            via: 10.244.0.225
            - to: 10.193.109.64/26
            via: 10.244.0.225
            - to: 100.64.0.0/16
            via: 10.244.0.225
    version: 2
root@jumpbox:/etc/netplan#
```

  - 설정 결과 확인
```
root@jumpbox:/etc/netplan# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.193.109.1    0.0.0.0         UG    0      0        0 ens160
10.96.0.0       10.244.0.225    255.255.254.0   UG    0      0        0 ens192
10.193.109.0    0.0.0.0         255.255.255.0   U     0      0        0 ens160
10.193.109.64   10.244.0.225    255.255.255.192 UG    0      0        0 ens192
10.244.0.0      10.244.0.225    255.255.240.0   UG    0      0        0 ens192
10.244.0.224    0.0.0.0         255.255.255.240 U     0      0        0 ens192
10.244.0.235    10.244.0.225    255.255.255.255 UGH   0      0        0 ens192
100.64.0.0      10.244.0.225    255.255.0.0     UG    0      0        0 ens192
root@jumpbox:/etc/netplan#
```

### 접속확인
  - 외부에서 접속
```
dhlee@dhlees-MacBook-Pro:~$ssh ubuntu@10.193.109.5
ubuntu@10.193.109.5's password:
```

  - JumpBox에서 tkc cluster 접근
```
ubuntu@jumpbox:~$ ssh vmware-system-user@10.244.1.34
Welcome to Photon 3.0 (\m) - Kernel \r (\l)
vmware-system-user@10.244.1.34's password: 
```