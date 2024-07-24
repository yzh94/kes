#### 支持系统

| 系统        | kubernetes版本 |
| ----------- | -------------- |
| ubuntu22.04 | v1.28.2        |
| ----------- | -------------- |
| centos7.9   | v1.28.2        |
#### ubuntu安装依赖

```
apt install ansible  sshpass -y

```
#### centos安装依赖

```
只在ansible机器

sudo yum install epel-release -y

yum install python3 python3-pip sshpass python2-pyyaml gcc libffi-devel python-devel openssl-devel -y
pip3 install --upgrade pip -i  https://mirrors.aliyun.com/pypi/simple
pip3 install ansible -i  https://mirrors.aliyun.com/pypi/simple

#####设置完hosts文件后执行  给所有机器安装依赖python
yum install python3 python3-pip  -y
```
#### 上传安装包（上传路径/root）

略

#### 解压并进入项目路径

```
unzip kes.zip && cd kes
```

#### 配置hosts文件

```
# 'etcd' cluster should have odd member(s) (1,3,5,...)
[etcd]
192.168.106.100 ansible_ssh_user=root      ansible_ssh_pass=yzh994927
# master node(s), set unique 'k8s_nodename' for each node
# CAUTION: 'k8s_nodename' must consist of lower case alphanumeric characters, '-' or '.',
# and must start and end with an alphanumeric character
[kube_master]
192.168.106.100 k8s_nodename='master-100' ansible_ssh_user=root      ansible_ssh_pass=yzh994927

# work node(s), set unique 'k8s_nodename' for each node
# CAUTION: 'k8s_nodename' must consist of lower case alphanumeric characters, '-' or '.',
# and must start and end with an alphanumeric character
[kube_node]
# 192.168.106.101 k8s_nodename='node-101' ansible_ssh_user=root      ansible_ssh_pass=yzh994927
# and must set in cluster node
[registry_node]
192.168.106.100

# [optional] ntp server for the cluster
[chrony]
192.168.106.100




# Deploy Directory ( workspace) 需要修改到hosts所在文件路径
base_dir="/root/kes"

```

#### 执行安装

```
ansible-playbook -i hosts -e @config.yml playbooks/setup.yml
```


#### 增加etcd节点

```
在hosts文件中增加etcd节点
[etcd]
192.168.106.100 ansible_ssh_user=root      ansible_ssh_pass=yzh994927
192.168.106.101 ansible_ssh_user=root      ansible_ssh_pass=yzh994927




ansible-playbook -i hosts -e @config.yml -e "ETCD_TO_ADD=192.168.106.101"   playbooks/addetcd.yml   
会重启master组件及etcd节点 慎重
ansible-playbook -i hosts -e @config.yml -t restart   playbooks/rerenderetcd.yml

```


#### 删除etcd节点

```
ansible-playbook -i hosts -e @config.yml -e "ETCD_TO_DEL=192.168.106.101"   playbooks/deleteetcd.yml
会重启master组件及etcd节点 慎重
ansible-playbook -i hosts -e @config.yml -t restart   playbooks/rerenderetcd.yml
```


#### 增加node节点
```
在hosts文件中增加node节点
[kube_node]
192.168.106.101 k8s_nodename='node-101' ansible_ssh_user=root      ansible_ssh_pass=yzh994927


ansible-playbook -i hosts -e @config.yml -e "NODE_TO_ADD=192.168.106.101" playbooks/addnode.yml
```

#### 删除node节点

```
ansible-playbook -i hosts -e @config.yml -e "NODE_TO_DEL=192.168.106.101" playbooks/deletenode.yml
```

#### 增加master节点

```
添加master节点
[kube_master]
192.168.106.100 k8s_nodename='master-100' ansible_ssh_user=root      ansible_ssh_pass=yzh994927


ansible-playbook -i hosts -e @config.yml -e MASTER_TO_ADD=192.168.106.101  playbooks/addmaster.yml

ansible-playbook -i hosts -e @config.yml -t restart_kube-lb  playbooks/setup.yml

```

#### 删除master节点
```
ansible-playbook -i hosts -e @config.yml -e MASTER_TO_DEL=192.168.106.101   playbooks/deletemaster.yml

ansible-playbook -i hosts -e @config.yml  -t create_kctl_cfg  roles/deploy/deploy.yml

ansible-playbook -i hosts -e @config.yml -t restart_kube-lb  playbooks/setup.yml

```