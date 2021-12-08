# Do a K8s benchmark

```
wget https://github.com/aquasecurity/kube-bench/releases/download/v0.4.0/kube-bench_0.4.0_linux_amd64.tar.gz && \
tar zxvf kube-bench_0.4.0_linux_amd64.tar.gz && \
sudo ./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml > /tmp/kk
```
