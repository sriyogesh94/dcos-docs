---
post_title: Configuration
menu_order: 600
---

The DC/OS configuration parameters are specified in [YAML][1] format in a config.yaml file. This file is stored on your [bootstrap node](/docs/1.9/installing/custom/system-requirements/#bootstrap-node) and is used during DC/OS installation to generate a customized DC/OS build.

# Format

## Key-value pairs
The config.yaml file is formatted as a list of key-value pairs. For example:

```yaml
bootstrap_url: file:///opt/dcos_install_tmp
```

## Config blocks and lists

```yaml
master_list:
- <master-private-ip-1>
- <master-private-ip-2>
- <master-private-ip-3>
```

or

```yaml
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

A config block is a group of settings. It consists of:

- A key followed by a colon (e.g. `agent_list:`). The key of the config block must be on its own line, with no leading space.
- A list of values formatted by using single dash (`-`) followed by a space; or an indented set of one or more key-value pairs. The indentation for each key-value pair must be exactly two spaces. Do not use tabs.
- Any number of empty lines or comment lines.

When a new config block appears in the file, the former config block is closed and the new one begins. A config block must only occur once in the file.

## Comments

```yaml
master_list:
- <master-private-ip-1>
# here is a comment
- <master-private-ip-2>
- <master-private-ip-3>
```

Comment lines start with a hash symbol (`#`). They can be indented with any amount of leading space.

Partial-line comments (e.g. `agent_list # this is my agent list`) are not allowed. They will be treated as part of the value of the setting. To be treated as a comment, the hash sign must be the first non-space character on the line.

## Dependencies
Some parameters are dependent on others. These dependent parameters are ignored unless all dependencies are specified. These dependencies are shown in the documentation by nesting within the parent. For example, `master_list` is required only if you specify ` master_discovery: static`.

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

## Advanced

See the [configuration reference](/docs/1.9/installing/custom/configuration/configuration-reference/) and [examples](/docs/1.9/installing/custom/configuration/examples/).