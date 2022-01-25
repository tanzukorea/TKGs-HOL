---
title: "4. TKC - NodePort 접근"
lastmod: 2020-10-05
draft: true
weight: 400
---

### NodePort
서비스 유형중에서 외부에서 접근하기 위한 방법으로 LoadBalancer, NodePort를 사용할 수 있습니다. TKC&NSX-T를 사용하는 경우 NodePort가 외부에서 접근이 되지 않습니다. NodePort를 사용해야 하는 경우 NSX-T에서 추가 설정을 통해 접근하게 설정할 수 있습니다.


### 그룹 설정
- NSX-T Manager에 로그인합니다.
- [Inventory] > [Groups]로 이동합니다.
  * [ADD GROUP] 클릭

- 생성할 그룹이름을 입력합니다. 이름은 자동으로 생성된 것들과 구별하기 쉽게 입력합니다. 그리고 [Set Members] 클릭


- 멤버를 구성하는 방식이 여러가지가 있습니다. 여기서는 TKC 클러스터의 Worker Nodes로 LB를 구성할 예정이라, 이름 규칙을 통해 멤버를 찾도록 하겠습니다. *Computer Name* 항목으로 Worker Node의 이름으로 설정합니다. 예, tkc-cluster-worker


- 설정한 그룹을 [Save] 합니다.

- 상태가 [Success]로 된 것을 확인하고 멤버 구성을 확인하기 위해 [View Members] 클릭

- 그림처럼 원하는 멤버가 맞는지 확인합니다.


### Server Pool 설정