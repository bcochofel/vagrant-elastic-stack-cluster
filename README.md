# Provision a Elastic Stack Cluster

## Introduction

Provision a Elastic Stack cluster composed of master nodes, data nodes, kibana and logstash.

This project uses ansible roles from:

- https://galaxy.ansible.com/elastic/elasticsearch
- https://galaxy.ansible.com/cloudalchemy/grafana
- https://galaxy.ansible.com/cloudalchemy/prometheus
- https://galaxy.ansible.com/dj-wasabi/telegraf

## Considerations

The default Vagrantfile deploys the following VMs:

| name | ip | type |
| ---- | -- | ---- |
| master-1 | 192.168.77.30 | master node |
| data-1 | 192.168.77.40 | data node |
| data-2 | 192.168.77.41 | data node |
| data-3 | 192.168.77.42 | data node |
| logstash-1 | 192.168.77.50 | logstash node |
| kibana-1 | 192.168.77.60 | kibana + coordinating node + prometheus server |

## Installation

To create the cluster execute:

```bash
sudo pip install jmespath
ansible-galaxy install --roles-path ./ansible/roles -r requirements.yml
vagrant plugin install vagrant-hostmanager
vagrant up
```

After running the commands go to:

| application | url |
| ----------- | --- |
| Prometheus | http://kibana-1:9090 |
| Grafana (1) | http://kibana-1:3000 |
| Kibana (2) | http://kibana-1:5601 |

* (1) Default credentials for Grafana are: admin:password
* (2) Needs Kibana installation
