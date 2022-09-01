# DaemonSet
**daemonset** konusuyla ilgili dosyalara buradan erişebilirsiniz.
***
DaemonSet objelerinin listelenmesi

```
$ kubectl get daemonset
```
***
DaemonSet objelerinin silinmesi

```
$ kubectl delete daemonset "daemonset_ismi"

Ör: kubectl delete daemonset my-daemonset
```
***
---
DamenomSet, tüm veya bazı nodeların bir Pod un bir kopyasını çalıştırmasınısağlar. Clustera yeni node eklendikçe, onlara
Podlar da eklenir. Clusterdan node kaldırıldığında bu Podlar da kaldırılır. 
Bir DaemonSet in silinmesi oluşturduğu Podları da temizleyecektir.
---
