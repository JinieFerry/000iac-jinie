```
   kubectl config set-context --current --namespace=wb_prod
   systemctl status nginx
   kubectl config set-context --current --namespace=wb_prod
   kubectl config view --minify | grep namespace:
   kubectl config set-context --current --namespace=wb_dev
   kubectl config view --minify | grep namespace:
   kubectl config set-context --current --namespace=wb_prod
   kubectl config view --minify | grep namespace:
   systemctl status nginx
   kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
   kubectl get pods -n local-path-storage
   kubectl get sc
   cd k8s_lab/
   ls
   mkdir 03_loosepj
   cd 03_loosepj/
   ls
   vi local-pvc.yml
   ansible -playbook local
   k apply
   kubectl apply -f local-pvc.yml 
   history
   kubectl get namespace
   kubectl config set-context --current --namespace=wb-pord
   ls
   kubectl get pvc
   kubectl get pv
   kubectl apply -f local-pvc.yml 
   kubectl apply -f local-pvc.yml  -n wb-pr
   kubectl apply -f local-pvc.yml --namespace=web-prod
   vi local-pvc.yml 
   kubectl apply -f local-pvc.yml --namespace=web-prod
   vi ng-web.yml
   kubectl apply -f ng-web.yml
   kubectl get deployments.apps  ng-web
   kubectl get svc
   kubectl get svc -A
   kubectl config set-context --current --namespace=loosepj
   kubectl get svc
   kubectl apply -f service.yaml -n loosepj
   history
   cd /opt
   ls
   find . -maxdepth 2 -type f \( -name "*.yaml" -o -name "*.yml" \)
   kubectl apply -f ng-web.yml
   cd ~/03_loosepj
   cd ..
   cd
   ls
   cd k8s_lab/
   ls
   cd 03_loosepj/
   ls
   kubectl apply -f ng-web.yml
   kubectl create namespace loosepj
   kubectl get ns
   kubectl apply -f ng-web.yml -n loosepj
#에러났음 끊지않고 되야하는데 안됨
```

```
#013 ningx - local con
vi ng-pvc.yaml
vi ng-web-vol.yaml

```
nginx 

====

delete history : 모두 지움

====

```
 1574  together
 1575  k get svc
 1576  k get namespace
 1577  k delete namespaces wb-prod
 1578  k get pod
 1579  kubectl create namespace wb-prod
 1580  kubectl create namespace wb-dev
 1581  kubectl config set-context --current --namespace=wb-prod
 1582  kubectl config view --minify | grep namespace:
 1583  kubectl config set-context --current --namespace=wb-prod
 1584  k get sc
 1585  ls
 1586  vi ng-web.yaml
 1587  kubectl delete deployment nginx-deployment
 1588  vi ng-web.yaml
 1589  kubectl delete service nginx-service
 1590  kubectl apply -f ng-web.yaml
 1591  kubectl get deployment ng-web
 1592  kubectl get service fr-end
 1593  kubectl get deployment ng-web
 1594  kubectl get service fr-end
 1595  kubectl apply -f ng-web.yaml
 1596  vi ng-web.yaml
 1597  ls
 1598  kubectl apply -f ng-web.yaml
 1599  kubectl delete deployment nginx-deployment
 1600  kubectl delete service nginx-service
 1601  ls
 1602  kubectl apply -f ng-web.yaml
 1603  vi ng-web.yaml
 1604  kubectl get deployment ng-web
 1605  kubectl get service fr-end
 1606  history
 1607  vi ng-web.yaml
 1608  ls
 1609  vi ng-web.yaml
 1610  ls
 1611  kubectl delete deployment nginx-deployment
 1612  kubectl delete service nginx-service
 1613  k get deployment
 1614  k delete deployment ng-web
 1615  k get deploymetn
 1616  k get deployment
 1617  k apply -f ng-web.yaml
 1618  k get deployments.apps ng-web
 1619  vi ng-web.yaml
 1620  ip a
 1621  k get nodes
 1622  sudo ufw status
 1623  k get svc
 1624  k apply -f ng-web.yaml -n wb-prod
 1625  kubectl config view --minify | grep namespace:
 1626  kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
 1627  kubectl get pods -n local-path-storage
 1628  k get sc
 1629  vi wb-prod
 1630  k get deployments.apps 
 1631  k get pv
 1632  lsof -i :8080
 1633  lsof -i :80
 1634  lsof -i :30080
 1635  k get svc
 1636  k get svc all
 1637  k get svc All
 1638  k get svc A
 1639  k get svc --all
 1640  k get svc -all
 1641  k get svc -a
 1642  k get --help
 1643  k get svc -A
 1644  vi wb-prod
 1645  k get pv
 1646  vi ng-web.yaml
 1647  k delete svc -A
 1648  k delete svc --all
 1649  k delete svc --all-namespaces
 1650  k get svc -A
 1651  k get svc --all -A
 1652  k delete svc --all -A
 1653  k get svc -A
 1654  kubectl delete deployment nginx-deployment
 1655  kubectl delete service nginx-service
 1656  kubectl apply -f ng-web.yaml  -n wb-prod
 1657  kubectl get deployment ng-web
 1658  kubectl get service fr-end
 1659  ls
 1660  rm ng-web.yml
 1661  ls
 1662  k config view --minify | grep namespace:
 1663  k get svc --all -A
 1664  k get svc -A
 1665  kubectl config view --minify | grep namespace:
 1666  master@vmmaster1:~/k8s_lab/03_loosepj$ kubectl config view --minify | grep namespace:
 1667  kubectl get deploy,po,svc,ep
 1668  k get deployments.apps 
 1669  k get svc
 1670  k get namespaces 
 1671  k delete loosepj
 1672  k delete namespace loosepj
 1673  k get svc -A
 1674  lsof -i :80
 1675  lsof -i :8080
 1676  sudo ufw status
 1677  cd /opt
 1678  ls
 1679  cd
 1680  k get nodes
 1681  k get pods
 1682  k get local-path
 1683  k get pvc
 1684  k get pvc -A
 1685  systemctl status nfs
 1686  systemctl status nfs-server
 1687  systemctl stop nfs-server.service 
 1688  systemctl disabled nfs-server.service
 1689  systemctl disable nfs-server.service
 1690  systemctl status
 1691  systemctl status nfs-server
 1692  history
 1693  k get pvc -A
 1694  k delete pvc -A
 1695  k delete pvc --all
 1696  k delete pvc --all -A
 1697  k get pvc -A
 1698  k delete pods  --all -A
 1699  k get pods
 1700  k get deployments.apps -A
 1701  k delete deployments.apps --all -A
 1702  k get deployments.apps -A
 1703  k get namespace
 1704  k get svc -A
 1705  k get deployments.apps 
 1706  k get deployment ng-web
 1707  ls
 1708  cd k8s_lab/
 1709  ls
 1710  cd 03_loosepj/
 1711  ls
 1712  k apply -f ng-web.yaml -n wb-prod
 1713  k get deployments.apps ng-web 
 1714  k get service fr-end 
 1715  k get pod -n wb-prod -o wide
 1716  k get svc
 1717  k delete svc
 1718  k delete svc -A
 1719  k delete svc --all -A
 1720  k get deployments.apps 
 1721  k get svc
 1722  k get svc -A
 1723  k get deployments.apps 
 1724  k delete deployments.apps -A
 1725  k delete deployments.apps --all -A
 1726  ls
 1727  vi local-pvc.yml 
 1728  k get namespace
 1729  k get sc
 1730  k get pod -n local-path-storage -o wide
 1731  k logs -n local-path-storage deploy/local-path-provisioner --tail=200
 1732  k get deploy -n local-path-storage
 1733  k delete namespace local-path-storage 
 1734  k get namespace
 1735  clear
 1736  kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
 1737  k get pods
 1738  k get pods -n local-path-storage 
 1739  k get sc
 1740  ls
 1741  vi local-pvc.yml 
 1742  vi ng-pvc.yaml
 1743  kubectl get pvc ng-static-pvc -n wb-prod -o yaml | grep "volume.kubernetes.io/selected-node"
 1744  cd ~/.ssh
 1745  ls
 1746  vi authorized_keys
 1747  history

```
테인트 풀기
