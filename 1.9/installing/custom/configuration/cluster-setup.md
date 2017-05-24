---
post_title: Cluster Setup
menu_order: 100
---

```yaml
agent_list:
- <agent-private-ip-1>
- <agent-private-ip-2>
- <agent-private-ip-3>
bootstrap_url: <path-to-installer>
cluster_docker_credentials:
  auths:
    '<path-to-credentials>':
      auth: <username>
      email: <email>
  cluster_docker_credentials_dcos_owned: <true|false>
    cluster_docker_credentials_write_to_etc: <true|false>
      cluster_docker_credentials_write_to_etc: <true|false>
cluster_docker_registry_url: <url>
cluster_name: '<cluster-name>'
cosmos_config:
  staged_package_storage_uri: <temp-path-to-files>
  package_storage_uri: <permanent-path-to-files>
exhibitor_storage_backend: static
exhibitor_storage_backend: zookeeper
  exhibitor_zk_hosts: `<list-of-ip-port>`
  exhibitor_zk_path: <filepath-to-data>
exhibitor_storage_backend: aws_s3
  aws_access_key_id: <key-id>
  aws_region: <bucket-region>
  aws_secret_access_key: <secret-access-key>
  exhibitor_explicit_keys: <true|false>
  s3_bucket: <s3-bucket>
  s3_prefix: <s3-prefix>
exhibitor_storage_backend: azure
  exhibitor_azure_account_name: <storage-account-name>
  exhibitor_azure_account_key: <storage-account-key>
  exhibitor_azure_prefix: <blob-prefix>
master_discovery: static
  master_list:
  - <master-private-ip-1>
  - <master-private-ip-2>
  - <master-private-ip-3>
master_discovery: master_http_loadbalancer
  exhibitor_address: <loadbalancer-ip>
  num_master: <num-of-masters>
public_agent_list:
- <hostname>
platform: <platform>
```

# agent_list
This parameter specifies a YAML nested list (`-`) of IPv4 addresses to your [private agent](/docs/1.9/overview/concepts/#private) host names.

# bootstrap_url
This required parameter specifies the URI path for the DC/OS installer to store the customized DC/OS build files. If you are using the automated DC/OS installer, you should specify `bootstrap_url: file:///opt/dcos_install_tmp` unless you have moved the installer assets. By default the automated DC/OS installer places the build files in `file:///opt/dcos_install_tmp`.

# cluster_docker_credentials
This parameter specifies a dictionary of Docker credentials to pass. 

- If unset, a default empty credentials file is created at `/etc/mesosphere/docker_credentials` during DC/OS install. A sysadmin can change credentials as needed. A `systemctl restart dcos-mesos-slave` or `systemctl restart dcos-mesos-slave-public` is required for changes to take effect.
- You can also specify by using the `--docker_config` JSON [format](http://mesos.apache.org/documentation/latest/configuration/). You can write as YAML in the `config.yaml` file and it will automatically be mapped to the JSON format for you. This will store the Docker credentials in the same location as the DC/OS internal configuration (`/opt/mesosphere`). If you need to update or change the configuration, you will have to create a new DC/OS internal configuration.

You can use the following options to further configure the Docker credentials:

### cluster_docker_credentials_dcos_owned
This parameter specifies whether to store the credentials file in `/opt/mesosphere` or `/etc/mesosphere/docker_credentials`. A sysadmin cannot edit `/opt/mesosphere` directly.

*  `cluster_docker_credentials_dcos_owned: 'true'` The credentials file is stored in `/opt/mesosphere`.

    *  **cluster_docker_credentials_write_to_etc** This parameter specifies whether to write a cluster credentials file.
    
        *  `cluster_docker_credentials_write_to_etc: 'true'` Write a credentials file. This can be useful if overwriting your credentials file will cause problems (e.g. if it is part of a machine image or AMI). This is the default value.
        *  `cluster_docker_credentials_write_to_etc: 'false'` Do not write a credentials file.
        
*  `cluster_docker_credentials_dcos_owned: 'false'` The credentials file is stored in `/etc/mesosphere/docker_credentials`.

### cluster_docker_credentials_enabled
This parameter specifies whether to pass the Mesos `--docker_config` option to Mesos. 

*  `cluster_docker_credentials_enabled: 'true'` Pass the Mesos `--docker_config` option to Mesos. It will point to a file that contains the provided `cluster_docker_credentials` data.
*  `cluster_docker_credentials_enabled: 'false'` Do not pass the Mesos `--docker_config` option to Mesos. 
    
For more information, see the [examples](#docker-credentials).

# cluster_docker_registry_url
This parameter specifies a custom URL that Mesos uses to pull Docker images from. If set, it will configure the Mesos' `--docker_registry` flag to the specified URL. This changes the default URL Mesos uses for pulling Docker images. By default `https://registry-1.docker.io` is used.

# cluster_name
This parameter specifies the name of your cluster.

# cosmos_config
This parameter specifies a dictionary of packaging configuration to pass to the [DC/OS package manager](https://github.com/dcos/cosmos). If set, the following options must also be specified.

### staged_package_storage_uri
This parameter specifies where to temporarily store DC/OS packages while they are being added. The value must be a file URL, for example, `file:///var/lib/dcos/cosmos/staged-packages`.

### package_storage_uri
This parameter specifies where to permanently store DC/OS packages. The value must be a file URL, for example, `file:///var/lib/dcos/cosmos/packages`.

# exhibitor_storage_backend
This parameter specifies the type of storage backend to use for Exhibitor. You can use internal DC/OS storage (`static`) or specify an external storage system (`zookeeper`, `aws_s3`, and `azure`) for configuring and orchestrating ZooKeeper with Exhibitor on the master nodes. Exhibitor automatically configures your ZooKeeper installation on the master nodes during your DC/OS installation.

*   `exhibitor_storage_backend: static`
    This option specifies that the Exhibitor storage backend is managed internally within your cluster.
*   `exhibitor_storage_backend: zookeeper`
    This option specifies a ZooKeeper instance for shared storage. If you use a ZooKeeper instance to bootstrap Exhibitor, this ZooKeeper instance must be separate from your DC/OS cluster. You must have at least 3 ZooKeeper instances running at all times for high availability. If you specify `zookeeper`, you must also specify these parameters.
    *   **exhibitor_zk_hosts**
        This parameter specifies a comma-separated list (`<ZK_IP>:<ZK_PORT>, <ZK_IP>:<ZK_PORT>, <ZK_IP:ZK_PORT>`) of one or more ZooKeeper node IP and port addresses to use for configuring the internal Exhibitor instances. Exhibitor uses this ZooKeeper cluster to orchestrate it's configuration. Multiple ZooKeeper instances are recommended for failover in production environments.
    *   **exhibitor_zk_path**
        This parameter specifies the filepath that Exhibitor uses to store data.
*   `exhibitor_storage_backend: aws_s3`
    This option specifies an Amazon Simple Storage Service (S3) bucket for shared storage. If you specify `aws_s3`, you must also specify these parameters:
    *  **aws_access_key_id**
       This parameter specifies AWS key ID.
    *  **aws_region**
       This parameter specifies AWS region for your S3 bucket.
    *  **aws_secret_access_key**
       This parameter specifies AWS secret access key.
    *  **exhibitor_explicit_keys**
       This parameter specifies whether you are using AWS API keys to grant Exhibitor access to S3.
        *  `exhibitor_explicit_keys: 'true'`
           If you're  using AWS API keys to manually grant Exhibitor access.
        *  `exhibitor_explicit_keys: 'false'`
           If you're using AWS Identity and Access Management (IAM) to grant Exhibitor access to s3.
    *  **s3_bucket**
       This parameter specifies name of your S3 bucket.
    *  **s3_prefix**
       This parameter specifies S3 prefix to be used within your S3 bucket to be used by Exhibitor.

       **Tip:** AWS EC2 Classic is not supported.
*   `exhibitor_storage_backend: azure`
   This option specifies an Azure Storage Account for shared storage. The data will be stored under the container named `dcos-exhibitor`. If you specify `azure`, you must also specify these parameters:
    *  **exhibitor_azure_account_name**
       This parameter specifies the Azure Storage Account Name.
    *  **exhibitor_azure_account_key**
       This parameter specifies a secret key to access the Azure Storage Account.
    *  **exhibitor_azure_prefix**
       This parameter specifies the blob prefix to be used within your Storage Account to be used by Exhibitor.


# <a name="master"></a>master_discovery
This required parameter specifies the Mesos master discovery method. The available options are `static` or `master_http_loadbalancer`.

*  `master_discovery: static`
This option specifies that Mesos agents are used to discover the masters by giving each agent a static list of master IPs. The masters must not change IP addresses, and if a master is replaced, the new master must take the old master's IP address. If you specify `static`, you must also specify this parameter:

    *  **master_list**
       This required parameter specifies a list of your static master IP addresses as a YAML nested series (`-`).

*   `master_discovery: master_http_loadbalancer` This option specifies that the set of masters has an HTTP load balancer in front of them. The agent nodes will know the address of the load balancer. They use the load balancer to access Exhibitor on the masters to get the full list of master IPs. If you specify `master_http_load_balancer`, you must also specify these parameters:

    *  **exhibitor_address** This required parameter specifies the location (preferably an IP address) of the load balancer in front of the masters. The load balancer must accept traffic on ports 80, 443, 2181, 5050, 8080, 8181. The traffic must also be forwarded to the same ports on the master. For example, Mesos port 5050 on the load balancer should forward to port 5050 on the master. The master should forward any new connections via round robin, and should avoid machines that do not respond to requests on Mesos port 5050 to ensure the master is up.
    *  **num_masters**
       This required parameter specifies the number of Mesos masters in your DC/OS cluster. It cannot be changed later. The number of masters behind the load balancer must never be greater than this number, though it can be fewer during failures.

*Note*: On platforms like AWS where internal IPs are allocated dynamically, you should not use a static master list. If a master instance were to terminate for any reason, it could lead to cluster instability.

# <a name="public-agent"></a>public_agent_list
This parameter specifies a YAML nested list (`-`) of IPv4 addresses to your [public agent](/docs/1.9/overview/concepts/#public) host names.

# <a name="platform"></a>platform
This parameter specifies the infrastructure platform. The value is optional, free-form with no content validation, and used for telemetry only. Please supply an appropriate value to help inform DC/OS platform prioritization decisions. Example values: `aws`, `azure`, `oneview`, `openstack`, `vsphere`, `vagrant-virtualbox`, `onprem` (default).
