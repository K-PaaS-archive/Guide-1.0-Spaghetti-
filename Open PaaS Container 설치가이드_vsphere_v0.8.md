## Table of Contents
1. [����](#����)
     * [���� ����](#����-����)
     * [����](#����)
     * [���� �ڷ�](#����-�ڷ�)
2. [Prerequisites](#prerequisites)
    * [OpenPaas Controller ��ġȮ��](#openpaas-controller-��ġȮ��)
3. [Open Paas Container ��ġ](#open-paas-container-��ġ)
   * [Release Upload](#release-upload)
4. [Deployment Manifest ���� �����ϱ�](#deployment-manifest-����-�����ϱ�). 
   * [Name & Release](#name-release)
   * [Compilation](#compilation)
   * [Resource Pools](#resource-pools)
   * [Update](#update)
   * [Jobs](#jobs)
   * [Properties](#properties)
5. [Deployment Manifest ����](#deployment-manifest-����)
6. [Bosh Deploy](#bosh-deploy)
7. [��ġ���� Ȯ��](#��ġ����-Ȯ��)
7. [��ġ ����](#��ġ-����)
   * [CF Login](#cf-login)
   * [Application Deploy](#application-deploy)
   * [Application Access](#application-access)
   
## ����
----

#### ���� ����

 �� ����(��ġ���̵�)��, �� �������� �����Ǵ� IaaS(Infrastructure as a Service) �� �ϳ��� VSphere ȯ�濡�� ������Ŭ�����÷���(OpenPaas Container) �� ��ġ�ϱ� ���� ���̵带 �����ϴµ� �� ������ �ִ�.

#### ����

 �� ������ ������ ������Ŭ�����÷����� VSphere ��ݿ� ��ġ�ϱ� ���� �������� �����Ǿ� �ִ�. Openstack/AWS�� ���� �ٸ� IaaS ȯ�濡���� ��ġ�� �׿� �´� ���̵� ������ �����ؾ� �ϸ�, Bosh/CF release ��ġ ���� �ش� ���̵� ������ ������ �����ؾ� �Ѵ�.

#### ���� �ڷ�

 [***https://github.com/cloudfoundry-incubator/diego-release***](https://github.com/cloudfoundry-incubator/diego-release)


## Prerequisites

#### OpenPaas Controller ��ġȮ��

 ������Ŭ�����÷��� (OpenPaas Container) �� ��ġ�ϱ� ���ؼ��� ������ OpenPaas Controller�� ��ġ�Ǿ� �־�� �Ѵ�.

 Ȯ���ϴ� ����� bosh deployments�� ���� ������ ����Ʈ ������� Ȯ���Ѵ�.
 
 ![cf push](https://github.com/OpenPaaSRnD/Documents/blob/master/images/openpaas-container/bosh_deployments_vsphere.png)

## Open Paas Container ��ġ

#### Release Upload

 ������ ��ġ ��Ű���� OpenPaaS-Controller ������ �ִ� Open PaaS Container Bosh Release�� Bosh Server�� �Ʒ��� ���� ������� Beta-1.0 ������ Upload �Ѵ�.

| bosh upload release $INSTALL\_PACKAGE/OpenPaaS-Container/ openpaas-container-beta-1.0.tgz |
|-------------------------------------------------------------------------------------------|

 Release Upload�� ��Ȳ�� ���� �ټ� ���̴� ������ ���� 20-30�� ���� �ҿ䰡 �Ǹ�, ���� Upload�� �Ǹ� �Ʒ��� �׸��� ���� �޽����� ��µȴ�.
 
 ![OpenPaas Container Upload Release](https://github.com/OpenPaaSRnD/Documents/blob/master/images/openpaas-container/bosh_upload_release.png)

 \[����\] Release Upload �������� �۾������ ��/tmp�� ������ ����� ���� ��� ���������� Ǯ�ų� ���� �� ������ �߻��� �� �����Ƿ�, 10GB �̻� Free Size�� �ִ����� Ȯ���ؾ� �Ѵ�.

 Bosh Sever�� Release�� ���������� Upload �Ǿ������� ��bosh releases�� ������� Ȯ���Ѵ�.


| bosh releases |
|---------------|

 ![OpenPaas Container Releases](https://github.com/OpenPaaSRnD/Documents/blob/master/images/openpaas-container/bosh_releases.png)

## Deployment Manifest ���� �����ϱ�

 ������ ��ġ ��Ű���� ���Ե� Sample Deployment Manifest File($INSTALL\_PACKAGE/OpenPaaS-Deployment/openpaas-container-vsphere-beta-1.0.yml)�� �Ʒ��� ������� ��ġȯ�濡 �����ϰ� �����Ѵ�.

#### Name & Release

```
 name: openpaas-container-vsphere-beta-1.0                    # Deployment Name
 director\_uuid: 6e0f7c41-2415-4319-98aa-38109597aff4         # Bosh Director UUID
 releases:
   - name: openpaas-container         # container Release Name
     version: latest                  # container Release Version
   - name: openpaas                   # controller Release Name
     version: latest                  # controller Release Version 

```
 Deployment Name�� ��ġ�ڰ� ���Ƿ� �ο��ϴµ�, IaaS�� Version�� ǥ���� ���� �����Ѵ�. Bosh Director UUID�� ��bosh status�� ����� �����ϸ� ��µǴ� UUID ���� �ִ´�.
 
 �� container & controller Release Name�� Version�� ��bosh releases�� ����� ����� ������ ������ �Է��ϵ��� �Ѵ�.

#### Networks

```
networks:
- name: openpaas-container-network          # Platform�� ��ġ�� Network Name
  subnets:
  - cloud_properties:
      name: Internal                        # VSphere Network �̸�
    dns:
    - 10.30.20.24                           # DNS Server
    - 8.8.8.8
    gateway: 10.30.20.23                    # Gateway IP Address
    range: 10.30.0.0/16                     # Network CIDR
    reserved:
    - 10.30.0.1 - 10.30.10.254
    - 10.30.40.1 - 10.30.40.9
    - 10.30.50.101 - 10.30.254.254
    static:
    - 10.30.40.150 - 10.30.40.200           # VM�� �Ҵ�� Static IP �ּ� �뿪
  type: manual

```

 Network Name�� ��ġ�ڰ� ���Ƿ� �ο� �����ϴ�. Network ID, Security_groups, Gateway, DNS Server, Network CIDR�� Vsphere ������ ���� Ȯ���ϰų� ������ ����ڿ��� �����Ͽ� ������ �򵵷� �Ѵ�. Static IP �ּҴ� Platform�� ��ġ�� �� ���� VM�� �Ҵ�� IP�� �ּ� �뿪���� ���������� ������ ����ڿ��� �Ҵ��� �޾ƾ� �Ѵ�.

#### compilation:

```
compilation:
  cloud_properties:
    cpu: 3                                 # Compile�� ����� CPU ����
    disk: 8192                             # Compile�� ����� Disk ũ�� 
    ram: 4096                              # Compile�� ����� Memory ũ�� 
  network: openpaas-container-network      # ��Ʈ��ũ ������ ���� �Ͱ� ������ �̸� 
  reuse_compilation_vms: true              # compilation VMs ���� ����
  workers: 6                               # ���� �����ϴ� VM ��

```

Network Name�� 3.3.2���� ������ �Ͱ� ������ �̸��� ��� �Ѵ�. Workers�� ���ÿ� Compile�� �����ϴ� VM�� ������ ���ٸ� ȯ���� Ư���� ���ٸ� Default ���� ������ �Ѵ�.

#### resource_pools

```
resource_pools:
- name: access                                            # Resource Name
  cloud_properties:
    cpu: 1                                                # Resource ���� ��  CPU ����
    disk: 1024                                            # Resource ���� ��  Disk ũ�� 
    ram: 1024                                             # Resource ���� ��  Memory ũ�� 
  env:
    bosh:
      password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0
  network: openpaas-container-network                     # Network Name
  size: 1 
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent        # Stemcell Name
    version: latest                                       # Stemcell Version 
    
- name: etcd
  cloud_properties:
    cpu: 1
    disk: 1024
    ram: 1024
  env:
    bosh:
      password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0
  network: openpaas-container-network
  size: 1
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
    version: latest
    
- name: cc_bridge
  cloud_properties:
    cpu: 1
    disk: 1024
    ram: 1024
  env:
    bosh:
      password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0
  network: openpaas-container-network
  size: 1
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
    version: latest
    
- name: route_emitter
  cloud_properties:
    cpu: 1
    disk: 1024
    ram: 1024
  env:
    bosh:
      password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0
  network: openpaas-container-network
  size: 1
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
    version: latest

- name: brain
  cloud_properties:
    cpu: 1
    disk: 1024
    ram: 1024
  env:
    bosh:
      password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0
  network: openpaas-container-network
  size: 1
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
    version: latest
                         
- name: cell
  cloud_properties:
    cpu: 2
    disk: 20480                                # Cell ������ VM�� �⺻������ 5G Disk �� �ʿ���ϱ� ������ App ���� ������ ����Ͽ� �ּ� 10G �̻��� ũ�⸦ �����Ѵ�.
    ram: 4096
  env:
    bosh:
      password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0
  network: openpaas-container-network
  size: 1
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
    version: latest

```

Resource pool ������ Jobs �������� �� VM���� ����ϱ� ���� Resource�� ���� ������ ��������, �� VM ������ �̸����� ���Ǿ� ������, �ʿ� ũ�⿡ ���� cpu, disk, ram ������ �����Ѵ�.. Stemcell Name�� Version�� ��bosh stemcells�� ��ɾ� ����� ��µǴ� ������ �Է��ϵ��� �Ѵ�.


#### update

```
update:
  canaries: 1                              # Canary instance ����
canary_watch_time: 30000-600000            # Canary instance �� healthy ���� ������� ����ϴ� �ð� 
  max_in_flight: 1                         # update instance���� �ִ� ����ó�� ���� 
  serial: true	                           # VM�� ������ Update
update_watch_time: 5000-600000             # canary instance �׽�Ʈ �� ���� instance update �ϸ鼭 healthy ���� ������� ����ϴ� �ð�

```
  Default ������ ���� ���� ����Ѵ�.

#### Jobs
  �Ʒ� Sample Jobs�� �����Ͽ� ��ġ ȯ�濡 �°� �����Ѵ�.

```
jobs:
- instances: 1                                      # VM Instance ����
  name: etcd                                        # Job Name
  networks:
  - name: openpaas-container-network                # Network Name
    static_ips:
    - 10.30.40.152                                  # Job(etcd) VM�� �Ҵ��� IP �ּ�
  persistent_disk: 1024
  resource_pool: etcd                               # Resource Name
  templates:
  - name: etcd                                      # etcd VM�� ����� ������Ʈ (job template����)
    release: openpaas-container                     # etcd job template�� �����ϴ� release ����
  update:
    max_in_flight: 1                                # update instance���� �ִ� ����ó�� ����
    serial: true                                    # ������ ���࿩��
- instances: 1
  name: brain
  networks:
  - name: openpaas-container-network
    static_ips:
    - 10.30.40.153                                  # Job(brain) VM�� �Ҵ��� IP �ּ�
  properties:
    metron_agent:
      zone: z1                                      # Zone ����(��ġ�Ǵ� VM �׷�)
  resource_pool: brain
  templates:
  - name: consul_agent
    release: openpaas
  - name: auctioneer
    release: openpaas-container
  - name: converger
    release: openpaas-container
  - name: runtime_metrics_server
    release: openpaas-container
  - name: metron_agent
    release: openpaas
  update:
    max_in_flight: 1
    serial: true
- instances: 1
  name: cell
  networks:
  - name: openpaas-container-network
    static_ips:
    - 10.30.40.154                                  # Job(cell) VM�� �Ҵ��� IP �ּ�
  properties:
    diego:
      rep:
        zone: z1
    metron_agent:
      zone: z1
  resource_pool: cell
  templates:
  - name: rep
    release: openpaas-container
  - name: consul_agent
    release: openpaas
  - name: garden-linux
    release: openpaas-container
  - name: metron_agent
    release: openpaas
  update:
    max_in_flight: 1
    serial: false
- instances: 1
  name: cc_bridge
  networks:
  - name: openpaas-container-network
    static_ips:
    - 10.30.40.155                                  # Job(cc_bridge) VM�� �Ҵ��� IP �ּ�
  properties:
    consul:
      agent:
        services:
        - file_server
        - nsync
        - stager
        - tps
    metron_agent:
      zone: z1
  resource_pool: cc_bridge
  templates:
  - name: stager
    release: openpaas-container
  - name: nsync
    release: openpaas-container
  - name: tps
    release: openpaas-container
  - name: file_server
    release: openpaas-container
  - name: consul_agent
    release: openpaas
  - name: metron_agent
    release: openpaas
  update:
    max_in_flight: 1
    serial: false
- instances: 1
  name: route_emitter
  networks:
  - name: openpaas-container-network
    static_ips:
    - 10.30.40.156                                  # Job(route_emitter) VM�� �Ҵ��� IP �ּ�
  properties:
    metron_agent:
      zone: z1
  resource_pool: route_emitter
  templates:
  - name: route_emitter
    release: openpaas-container
  - name: consul_agent
    release: openpaas
  - name: metron_agent
    release: openpaas
  update:
    max_in_flight: 1
    serial: false
- instances: 1
  name: access
  networks:
  - name: openpaas-container-network
    static_ips:
    - 10.30.40.157                                  # Job(access) VM�� �Ҵ��� IP �ּ�
  properties:
    consul:
      agent:
        services:
        - receptor
        - ssh_proxy
    metron_agent:
      zone: z1
  resource_pool: access
  templates:
  - name: ssh_proxy
    release: openpaas-container
  - name: consul_agent
    release: openpaas
  - name: metron_agent
    release: openpaas
  - name: receptor
    release: openpaas-container
  update:
    max_in_flight: 1
    serial: false

```

#### Properties

�Ʒ� Sample Manifest�� �����Ͽ� ��ġ ȯ�濡 �°� ���� �����Ѵ�.

```
properties:
  consul:                                   # consul �Ӽ� ����
    agent:
      log_level: null
      servers:
        lan:
        - 10.30.40.150                      # openpaas controller�� ������ consul server IP
  diego:
    auctioneer:                             # auctioneer �Ӽ� ����
      etcd:
        machines:
        - 10.30.40.152                        # etcd ���� ����
      log_level: null
    converger:                              # converger �Ӽ� ����
      etcd:
        machines:
        - 10.30.40.152         
      log_level: debug
    etcd:                                   # etcd �Ӽ� ����
      machines:
      - 10.30.40.152
    executor:                               # executor �Ӽ� ����
      allow_privileged: null                # privilege �Ӽ�
      drain_timeout_in_seconds: 0           # deprecated property 
      garden:
        address: 127.0.0.1:7777             # garden server ����
        network: tcp
      log_level: debug
    file_server:
      cc:
        base_url: http://api.controller.open-paas.com      # cloud controller url ����
        basic_auth_password: admin                         # cloud controller ���� �н����� 
        external_port: 9022                                # cloud controller ���� ��Ʈ 
        staging_upload_password: admin                     # staging upload �� ���� �н�����
        staging_upload_user: staging_upload_user           # staging upload �� ���� ����
      log_level: null
    garden-linux:
      allow_networks: null                                 # ���� ����� CIDR ���
      disk_quota_enabled: false                            # container�� ���� disk �ѵ� ���뿩��
      insecure_docker_registry_list: null                  # private docker registry url ����
      kernel_network_tuning_enabled: false          
      listen_address: 0.0.0.0:7777                         # garden ���� ����
      listen_network: tcp
    nsync:
      cc:
        base_url: http://api.controller.open-paas.com      # file_server �Ӽ� ����
        basic_auth_password: admin
        external_port: 9022
        staging_upload_password: admin
        staging_upload_user: staging_upload_user
      diego_api_url: http://:@receptor.service.consul:8888 # receptor ���� ����
      etcd:
        machines:
        - 10.30.40.152
      log_level: debug
    receptor:
      cors_enabled: null
      domain_names:
      - receptor.cf.open-pass.com.xip.io                   # receptor ������ ����
      etcd:
        machines:
        - 10.30.40.152                                       # etcd ���� ����
      log_level: debug
      nats:
        machines:
        - 10.30.40.111                                     # openpaas controller�� ������ nats vm ip
        password: admin                                    # nats ���� ���� �н�����
        port: 4222                                         # nats ���� ���� ��Ʈ
        username: nats                                     # nats ���� ���� ����
      password: ""                                         # receptor ���� ���� �н�����
      register_with_router: true                           # receptor ���� router�� ��Ͽ���
      username: ""                                         # receptor ���� ���� ����
    rep:
      etcd:
        machines:
        - 10.30.40.152
      log_level: debug
    route_emitter:
      diego_api_url: http://:@receptor.service.consul:8888
      log_level: debug
      nats:
        machines:
        - 10.30.40.111                                       # openpaas controller�� ������ nats vm ip
        password: admin
        port: 4222
        username: nats
    runtime_metrics_server:
      diego_api_url: http://:@receptor.service.consul:8888
      etcd:
        machines:
        - 10.30.40.152
      log_level: null
      nats:
        machines:
        - 10.30.40.111                                       # openpaas controller�� ������ nats vm ip
        password: admin
        port: 4222
        username: nats
    ssh_proxy:
      cc:
        external_port: 9022
      diego_api_url: http://:@receptor.service.consul:8888
      enable_cf_auth: true                                 # openpaas controller application�� ssh ���� ��� ����
      enable_diego_auth: true                              # openpaas container application�� ssh ���� ��� ����
      host_key: |+                                         # ssh_proxy host key
        -----BEGIN RSA PRIVATE KEY-----
        MIIEhgIBAAKB/DMF5qOW+fh608KhX7qBLNHHmfzCfOONd176Oaf8rGht5KdnoNge
        TYSGqBFuYB1r1RbYEVhWAkH/8mW14XRVNmQ4C9eQDFqeWmmaOoSBG5GdP5GUfhI/
        z5vprQw+rnV4gt4InCA7QaR86pLj5sMiUij5OE/CW0dw29+z5E0p5WnQX5+utRmw
        ioQJD8jUDvzFrvzKIdE0HVOEl0agbeXq8U2e9E1de4iR+NiDc1zeiQmDNCIhFJb4
        FL7WqqokL+49SwSWGmOFKAlpj4Dlhx5dDwJWpcDe0XBXCkfcXn8xXNOT+4YBxJUG
        idNMPpLKpDUphZRj8CNBSMkjehIKVwIDAQABAoH8MiCAAQQYvXfeh36HT/IMmGSi
        8mIY1G5tclAfSNzCfS5Jz/XNXcYXnjW09LsdjoocJX9NOx30xeawvCA+SU5WS4uM
        htEscfLVHJ67EubMsPhuNZZPbZpnWuPucPM77ojg+UY4LKpKyVE4G+vvEJKtaTe/
        jQyDJOLKATL4/p5DtbDH7hVZcJVHU94csiE9a9OtyAvSwZLmNxGIBHshFntjcI+/
        hmQSFl3d1iduYGx7oeq3wX0sQ1mk/QksUTHRrlLfSQhLi5ZmH9Hnn/Qw2WeXKVdk
        BvXAUBiHG7Y0qGHXl5FOkB1BSlmk/EOkBk6gWl1a1Kx4A6oyNL4+HsuBAn572PqW
        IDutj4shf8ysI5fLJnvGCygZmk8LPZIlZZqLpDGo+l4iF3VCsd8CU2jKfWqel8+Q
        axdmu/BrQ7xyuWpxoHtKICv+CitI1ivzeYQwRCmjIN84jeGP9Pty4AJzhySegf/h
        n3irIp07wEzdedoj4A3RWWObX+AeubyUqfcCfml3scNb2oBK24RDVGYaUSWkSHBe
        OEU0QlOaJXZ2kCK2rIK/IVI7cD12WpkWTGY782VBmipEXwtMTprQzMrnK25shS+z
        AjCDGXtqr0GjxJh73WRurs1dVk6sqslSp1M/R9fmjGU4vdYL2JfMczEH4+57aOpR
        sW+H0FEYDayKoQJ+Eo8gdjDcYJT7N4jsRfuLesEImVQArV2HbNrMNNh2AWkYnAbw
        5lD3nIgFMFcJhBapTJzZWP4DYrzVOW3MJrEMd3yiHSiXDxm9BMw7h9/05DrCtpRt
        fw8b9zOyHrPdCiz9WteGXexE6/hi8ZpOqn3hJ7EiwPWRTK5gappQ3UJfAn4Tr0t2
        cwZtO4uNPCPcirzqkacTkgJeqEpY4ERtv+NXF1FLdfD6MC3ayuRN/mN0EWx0UbI8
        gVZb/XoOWzpeBJeOnKKfLIIUG+P9rQPY9IAVFclUnXPy0KDzPjcCLHMejokSOu2p
        VtXXxY4/huFZHWflcxM56NV9Q5QWDq8+rQECfjQTbNbd4ehbC/Q5EZ1SIzeaSLrn
        0ICmiRajnISbje5vPntqPXjBkbiVGx31qOaZ+DlGGLOyzW/GP5X4NOUwza2bYh3q
        nnzwBhoGLZfvoes5Nw06leOdVqcvIjLIDhb+XbiiEeAnONUp+BAKzDYOIp7K+LPe
        1rHeshh0P/QfCQ==
        -----END RSA PRIVATE KEY-----
      servers:
      - 10.30.40.157                                       # ssh_proxy ���� ����
    ssl:
      skip_cert_verify: true                  
    stager:
      cc:
        base_url: http://api.controller.open-paas.com
        basic_auth_password: admin
        external_port: 9022
        staging_upload_password: admin
        staging_upload_user: staging_upload_user
      diego_api_url: http://:@receptor.service.consul:8888
      insecure_docker_registry: false
      log_level: null
    tps:
      cc:
        base_url: http://api.controller.open-paas.com
        basic_auth_password: admin
        external_port: 9022
        staging_upload_password: admin
        staging_upload_user: staging_upload_user
      diego_api_url: http://:@receptor.service.consul:8888
      log_level: null
      traffic_controller_url: wss://doppler.cf.open-paas.com:443
  etcd:
    machines:
    - 10.30.40.124                                           # openpaas controller�� ������ etcd vm ip
  loggregator_endpoint:
    shared_secret: admin
  metron_agent:
    deployment: openpaas-container-vsphere-beta-1.0
  nats:
    machines:
    - 10.30.40.111                                           # openpaas controller�� ������ nats vm ip
    password: admin
    port: 4222
    user: nats
  syslog_daemon_config:
    address: null
    port: null

```

## Deployment Manifest ����

| bosh deployment openpaas-container-vsphere-beta-1.0.yml |
|-----------------------------------------------------------|

 ��bosh deployment�� ��ɾ�� ������ Deployment Manifest File�� �����ϰ�, �Ʒ��� �׸��� ���� ������ ��ɾ�� ���� ���� �Ǿ������� Ȯ���Ѵ�
 
  ![OpenPaas Container deployment manifest file](https://github.com/OpenPaaSRnD/Documents/blob/master/images/openpaas-container//bosh_deployment_vsphere.png)

## Bosh Deploy

  Diego module�� ���� bosh upload ������ ��������, deploy ������ ���� Diego ���� VM�� �����Ѵ�.

| bosh deploy |
|-------------|

 ![OpenPaas Container deploy](https://github.com/OpenPaaSRnD/Documents/blob/master/images/openpaas-container/bosh_deploy_vsphere.png)

 \[�׸� : bosh deploy ���� ���\]


## ��ġ���� Ȯ��

 ��ġ�� ���������� �Ϸ�� �� ��bosh vms�� ������� ��ġ�� Platform�� ������ Ȯ���Ѵ�.

| bosh vms |
|----------|

 �Ʒ� �׸��� ���� Deployment Name, Virtual Machine, IP �ּ� ���� ������ Ȯ���� �� �ִ�.

 ![OpenPaas Container vms](https://github.com/OpenPaaSRnD/Documents/blob/master/images/openpaas-container/bosh_vms_vsphere.png)

##  ��ġ ����

#### CF Login

```
cf target api.controller.open-paas.com

��

cf login

Email&gt; admin

Password&gt; admin

OK

��

cf create-org open-paas

cf target -o open-paas

cf create-space dev

cf target -o open-paas -s dev

�� 

```


 CF Target�� �����ϰ�, Login�� �����Ѵ�. �� �� ������ admin/admin�� ����Ѵ�.
 Application�� Deploy�� ORG�� Space�� �����ϰ�, �ش��ϴ� ORG/Space�� Targetting �Ѵ�.

 �� admin ������ �н����� ������ �ٲٰ� �ʹٸ�, CF-Release deploy�� manifest ���� ���Ͽ��� �����Ͼ� �Ѵ�.

#### Application Deploy

 ������Ŭ�����÷��� ��Ű���� �Բ� ������ Sample Application�� ��ġ�ϴ� ���丮�� �̵��ϰ� Application�� Deploy �Ѵ�.

```
cd $PACKAGE\_ROOT/apps/hello-java

cf push ��application-name�� -i ��instance\_count�� -m ��memory\_size��

```

 Application ������ Disk ���� �ɼ� (-k)�� �������� ���� ��쿡�� �⺻������ 1G ũ���� ��ũ ��뷮�� �����ȴ�.
 
  ![cf push](https://github.com/OpenPaaSRnD/Documents/blob/master/images/openpaas-container/cf_push_vsphere1.png)
  ![cf push](https://github.com/OpenPaaSRnD/Documents/blob/master/images/openpaas-container/cf_push_vsphere2.png)
  
  ![cf apps](https://github.com/OpenPaaSRnD/Documents/blob/master/images/openpaas-container/cf_apps_vsphere.png)

#### Application Access

 Deploy�� Application URL�� Browser �Ǵ� curl ��ɾ�� Access�Ͽ� ���� ���� �Ǵ����� Ȯ���Ѵ�.
 
 ����) ������ App URL�� hello-spring-test.controller.open-paas.com �� ���

```
curl -L http://hello-spring-test.controller.open-paas.com

```

