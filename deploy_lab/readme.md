



# Create lab

```
vagrant up
```


# Destroy lab

```
vagrant destroy
```


# Accesion nodes

## Accessing master

```
$ vagrant ssh k8s-master
vagrant@k8s-master:~$ kubectl get nodes
NAME         STATUS   ROLES    AGE     VERSION
k8s-master   Ready    master   18m     v1.13.3
node-1       Ready    <none>   12m     v1.13.3
node-2       Ready    <none>   6m22s   v1.13.3
```
## Accessing nodes

```
$ vagrant ssh node-1
$ vagrant ssh node-2
```


# Links

[1] https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/
