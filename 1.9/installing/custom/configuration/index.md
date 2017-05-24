---
post_title: Configuration
menu_order: 600
---

The DC/OS configuration parameters are specified in [YAML][1] format in a config.yaml file. This file is stored on your [bootstrap node](/docs/1.9/installing/custom/system-requirements/#bootstrap-node) and is used during DC/OS installation to generate a customized DC/OS build.



# Format

## Key-value pairs
The config.yaml file is formatted as a list of key-value pairs. Each pair is in the format KEY: VALUE. For example:

```yaml
bootstrap_url: file:///opt/dcos_install_tmp
```

## Config sections and lists

```yaml
<key>:
- <value>
- <value>
```

or

```yaml
<key>:
  <key>: <value>
  <key>: <value>
```

A config section is a group of settings. It consists of:

- A key followed by a colon (e.g. `agent_list:`). The key of the config section must be on its own line, with no leading space.
- A list of values formatted by using single dash (`-`) followed by a space; or an indented set of one or more key-value pairs. The indentation for each key-value pair must be exactly two spaces. Do not use tabs.
- Any number of empty lines or comment lines.

When a new config section appears in the file, the former config section is closed and the new one begins. A config section must only occur once in the file.

## Comment lines

```yaml
<key>:
  <key>: <value>
# here is a comment
  <key>: <value>
```

Comment lines start with a hash symbol (`#`). They can be indented with any amount of leading space.

Partial-line comments (e.g. `agent_list # this is my agent list`) are not allowed. They will be treated as part of the value of the setting. To be treated as a comment, the hash sign must be the first non-space character on the line.

# Required settings

## Basics

- `agent_list` - YAML nested list (`-`) of IPv4 addresses to your [private agent](/docs/1.9/overview/concepts/#private) host names.
- `bootstrap_url` - The URI path for the DC/OS installer to store the customized DC/OS build files.
- `customer_key` (Enterprise DC/OS only) - The Enterprise DC/OS customer key.
- `cluster_name`- The name of your cluster.
- `exhibitor_storage_backend` - The type of storage backend to use for Exhibitor.
- `master_discovery` - The Mesos master discovery method.
- `master_list` - YAML nest list (`-`) of your static master IP addresses.
- `public_agent_list` - YAML nested list (`-`) of IPv4 addresses to your [public agent](/docs/1.9/overview/concepts/#public) host names. 
- `resolvers` - YAML nested list (`-`) of DNS resolvers for your DC/OS cluster nodes.
- `security` (Enterprise DC/OS only) - The security mode: `permissive`, `strict`, or `disabled`.
- `ssh_port` - The port to SSH.
- `ssh_user` - SSH username.
- `superuser_username` (Enterprise DC/OS only) - The user name of the superuser.
- `use_proxy` - Whether to enable the DC/OS proxy.
- `http_proxy` - The HTTP proxy.
- `https_proxy` - The HTTPS proxy.
- `no_proxy` -  YAML nested list (`-`) of addresses to exclude from the proxy.


# <a name="examples1"></a>Example Configurations

#### DC/OS cluster with three masters, five private agents, and Exhibitor/ZooKeeper managed internally.

```yaml
---
agent_list:
- <agent-private-ip-1>
- <agent-private-ip-2>
- <agent-private-ip-3>
- <agent-private-ip-4>
- <agent-private-ip-5>
bootstrap_url: 'file:///opt/dcos_install_tmp'
cluster_name: '<cluster-name>'
log_directory: /genconf/logs
master_discovery: static
master_list:
- <master-private-ip-1>
- <master-private-ip-2>
- <master-private-ip-3>
process_timeout: 120
resolvers:
- <dns-resolver-1>
- <dns-resolver-2>
ssh_key_path: /genconf/ssh-key
ssh_port: '<port-number>'
ssh_user: <username>
```

#### <a name="aws"></a>DC/OS cluster with three masters, an Exhibitor/ZooKeeper backed by an AWS S3 bucket, AWS DNS, five private agents, and one public agent node

```yaml
---
agent_list:
- <agent-private-ip-1>
- <agent-private-ip-2>
- <agent-private-ip-3>
- <agent-private-ip-4>
- <agent-private-ip-5>
aws_access_key_id: AKIAIOSFODNN7EXAMPLE
aws_region: us-west-2
aws_secret_access_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
bootstrap_url: file:///tmp/dcos
cluster_name: s3-example
exhibitor_storage_backend: aws_s3
exhibitor_explicit_keys: 'true'
log_directory: /genconf/logs
master_discovery: static
master_list:
- <master-private-ip-1>
- <master-private-ip-2>
- <master-private-ip-3>
process_timeout: 120
resolvers:
- <dns-resolver-1>
- <dns-resolver-2>
s3_bucket: mybucket
s3_prefix: s3-example
ssh_key_path: /genconf/ssh-key
ssh_port: '<port-number>'
ssh_user: <username>
```

#### <a name="zk"></a>DC/OS cluster with three masters, an Exhibitor/ZooKeeper backed by ZooKeeper, masters that have an HTTP load balancer in front of them, one public agent node, five private agents, and Google DNS

```yaml
---
agent_list:
- <agent-private-ip-1>
- <agent-private-ip-2>
- <agent-private-ip-3>
- <agent-private-ip-4>
- <agent-private-ip-5>
bootstrap_url: file:///tmp/dcos
cluster_name: zk-example
exhibitor_storage_backend: zookeeper
exhibitor_zk_hosts: 10.0.0.1:2181, 10.0.0.2:2181, 10.0.0.3:2181
exhibitor_zk_path: /zk-example
log_directory: /genconf/logs
master_discovery: master_http_loadbalancer
  num_masters: 3
exhibitor_address: 67.34.242.55
public_agent_list:
- 10.10.0.139
process_timeout: 120
resolvers:
- <dns-resolver-1>
- <dns-resolver-2>
ssh_key_path: /genconf/ssh-key
ssh_port: '<port-number>'
ssh_user: <username>
```

#### <a name="overlay"></a>DC/OS cluster with three masters, an Exhibitor/ZooKeeper managed internally, two DC/OS virtual networks, two private agents, and Google DNS

```yaml
    agent_list:
    - <agent-private-ip-1>
    - <agent-private-ip-2>
    # Use this bootstrap_url value unless you have moved the DC/OS installer assets.
    bootstrap_url: file:///opt/dcos_install_tmp
    cluster_name: <cluster-name>
    master_discovery: static
    master_list:
    - <master-private-ip-1>
    - <master-private-ip-2>
    - <master-private-ip-3>
    resolvers:
    # You probably do not want to use these values since they point to public DNS servers.
    # Instead use values that are more specific to your particular infrastructure.
    - 8.8.4.4
    - 8.8.8.8
    ssh_port: 22
    ssh_user: centos
    dcos_overlay_enable: true
    dcos_overlay_mtu: 9001
    dcos_overlay_config_attempts: 6
    dcos_overlay_network:
      vtep_subnet: 44.128.0.0/20
      vtep_mac_oui: 70:B3:D5:00:00:00
      overlays:
        - name: dcos
          subnet: 9.0.0.0/8
          prefix: 26
        - name: dcos-1
          subnet: 192.168.0.0/16
          prefix: 24
```

#### <a name="http-proxy"></a>DC/OS cluster with three masters, an Exhibitor/ZooKeeper managed internally, a custom HTTP proxy, two private agents, and Google DNS

```yaml
    agent_list:
    - <agent-private-ip-1>
    - <agent-private-ip-2>
    # Use this bootstrap_url value unless you have moved the DC/OS installer assets.
    bootstrap_url: file:///opt/dcos_install_tmp
    cluster_name: <cluster-name>
    master_discovery: static
    master_list:
    - <master-private-ip-1>
    - <master-private-ip-2>
    - <master-private-ip-3>
    resolvers:
    # You probably do not want to use these values since they point to public DNS servers.
    # Instead use values that are more specific to your particular infrastructure.
    - 8.8.4.4
    - 8.8.8.8
    ssh_port: 22
    ssh_user: centos
    use_proxy: 'true'
    http_proxy: http://<proxy_host>:<http_proxy_port>
    https_proxy: https://<proxy_host>:<https_proxy_port>
    no_proxy:
    - 'foo.bar.com'
    - '.baz.com'
```

#### <a name="docker-credentials"></a>DC/OS cluster with three masters, an Exhibitor/ZooKeeper managed internally, custom Docker credentials, two private agents, and Google DNS

```yaml
    agent_list:
    - <agent-private-ip-1>
    - <agent-private-ip-2>
    # Use this bootstrap_url value unless you have moved the DC/OS installer assets.
    bootstrap_url: file:///opt/dcos_install_tmp
    cluster_docker_credentials:
      auths:
        'https://registry.example.com/v1/':
          auth: foo
          email: user@example.com
    cluster_docker_credentials_dcos_owned: false
    cluster_docker_registry_url: https://registry.example.com
    cluster_name: <cluster-name>
    master_discovery: static
    master_list:
    - <master-private-ip-1>
    - <master-private-ip-2>
    - <master-private-ip-3>
    resolvers:
    # You probably do not want to use these values since they point to public DNS servers.
    # Instead use values that are more specific to your particular infrastructure.
    - 8.8.4.4
    - 8.8.8.8
    ssh_port: 22
    ssh_user: centos
```

#### <a name="cosmos-config"></a>DC/OS cluster with one master, an Exhibitor/ZooKeeper managed internally, three private agents, Google DNS, and the package manager (Cosmos) configured with persistent storage.

```yaml
    agent_list:
    - <agent-private-ip-1>
    - <agent-private-ip-2>
    - <agent-private-ip-3>
    # Use this bootstrap_url value unless you have moved the DC/OS installer assets.
    bootstrap_url: file:///opt/dcos_install_tmp
    cluster_name: <cluster-name>
    master_discovery: static
    master_list:
    - <master-private-ip-1>
    resolvers:
    # You probably do not want to use these values since they point to public DNS servers.
    # Instead use values that are more specific to your particular infrastructure.
    - 8.8.4.4
    - 8.8.8.8
    ssh_port: 22
    ssh_user: centos
    cosmos_config:
      staged_package_storage_uri: file:///var/lib/dcos/cosmos/staged-packages
      package_storage_uri: file:///var/lib/dcos/cosmos/packages
```



