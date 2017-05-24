---
post_title: Networking
menu_order: 200
---

```yaml

```


### <a name="dcos-overlay-enable"></a>dcos_overlay_enable

This parameter specifies whether to enable DC/OS virtual networks.

**Important:** Virtual networks require minimum Docker version 1.11. If you are using Docker 1.10 or earlier, you must specify `dcos_overlay_enable: 'false'`. For more information, see the [system requirements](/docs/1.9/installing/custom/system-requirements/).

*  `dcos_overlay_enable: 'false'` Do not enable the DC/OS virtual network.
*  `dcos_overlay_enable: 'true'` Enable the DC/OS virtual network. This is the default value. When the virtual network is enabled you can also specify the following parameters:

    *  `dcos_overlay_config_attempts` This parameter specifies how many failed configuration attempts are allowed before the overlay configuration modules stop trying to configure an virtual network.

        __Tip:__ The failures might be related to a malfunctioning Docker daemon.

    *  `dcos_overlay_mtu` This parameter specifies the maximum transmission unit (MTU) of the Virtual Ethernet (vEth) on the containers that are launched on the overlay.

    *  `dcos_overlay_network` This group of parameters define an virtual network for DC/OS. The default configuration of DC/OS provides an virtual network named `dcos` whose YAML configuration is as follows:

        ```
        dcos_overlay_network:
            vtep_subnet: 44.128.0.0/20
            vtep_mac_oui: 70:B3:D5:00:00:00
            overlays:
              - name: dcos
                subnet: 9.0.0.0/8
                prefix: 26
        ```

        *  `vtep_subnet` This parameter specifies a dedicated address space that is used for the VxLAN backend for the virtual network. This address space should not be routeable from outside the agents or master.
        *  `vtep_mac_oui` This parameter specifies the MAC address of the interface connecting to it in the public node.
            
            **Important:** The last 3 bytes must be `00`.
        *  __overlays__
            *  `name` This parameter specifies the canonical name (see [limitations](/docs/1.9/networking/virtual-networks/) for constraints on naming virtual networks).
            *  `subnet` This parameter specifies the subnet that is allocated to the virtual network.
            *  `prefix` This parameter specifies the size of the subnet that is allocated to each agent and thus defines the number of agents on which the overlay can run. The size of the subnet is carved from the overlay subnet.

 For more information see the [example](#overlay) and [documentation](/docs/1.9/networking/virtual-networks/).

### <a name="dns-search"></a>dns_search
This parameter specifies a space-separated list of domains that are tried when an unqualified domain is entered (e.g. domain searches that do not contain &#8216;.&#8217;). The Linux implementation of `/etc/resolv.conf` restricts the maximum number of domains to 6 and the maximum number of characters the setting can have to 256. For more information, see <a href="http://man7.org/linux/man-pages/man5/resolv.conf.5.html">man /etc/resolv.conf</a>.

A `search` line with the specified contents is added to the `/etc/resolv.conf` file of every cluster host. `search` can do the same things as `domain` and is more extensible because multiple domains can be specified.

In this example, `example.com` has public website `www.example.com` and all of the hosts in the datacenter have fully qualified domain names that end with `dc1.example.com`. One of the hosts in your datacenter has the hostname `foo.dc1.example.com`. If `dns_search` is set to &#8216;dc1.example.com example.com&#8217;, then every DC/OS host which does a name lookup of foo will get the A record for `foo.dc1.example.com`. If a machine looks up `www`, first `www.dc1.example.com` would be checked, but it does not exist, so the search would try the next domain, lookup `www.example.com`, find an A record, and then return it.

```yaml
dns_search: dc1.example.com dc1.example.com example.com dc1.example.com dc2.example.com example.com
```
### <a name="#resolvers"></a>resolvers

This required parameter specifies a YAML nested list (`-`) of DNS resolvers for your DC/OS cluster nodes. You can specify a maximum of 3 resolvers. Set this parameter to the most authoritative nameservers that you have.

-  If you want to resolve internal hostnames, set it to a nameserver that can resolve them.
-  If you do not have internal hostnames to resolve, you can set this to a public nameserver like Google or AWS. For example, you can specify the [Google Public DNS IP addresses (IPv4)](https://developers.google.com/speed/public-dns/docs/using):

    ```bash
    resolvers:
    - 8.8.4.4
    - 8.8.8.8
    ```
-  If you do not have a DNS infrastructure and do not have access to internet DNS servers, you can specify `resolvers: []`. By specifying this setting, all requests to non-`.mesos` will return an error. For more information, see the Mesos-DNS [documentation](/docs/1.9/networking/mesos-dns/).

**Caution:** If you set the `resolvers` parameter incorrectly, you will permanently damage your configuration and have to reinstall DC/OS.

### use_proxy

This parameter specifies whether to enable the DC/OS proxy. 

*  `use_proxy: 'false'` Do not configure DC/OS [components](/docs/1.9/overview/architecture/components/) to use a custom proxy. This is the default value.
*  `use_proxy: 'true'` Configure DC/OS [components](/docs/1.9/overview/architecture/components/) to use a custom proxy. If you specify `use_proxy: 'true'`, you can also specify these parameters:
    **Important:** The specified proxies must be resolvable from the provided list of [resolvers](#resolvers).
    *  `http_proxy: http://<user>:<pass>@<proxy_host>:<http_proxy_port>` This parameter specifies the HTTP proxy.
    *  `https_proxy: https://<user>:<pass>@<proxy_host>:<https_proxy_port>` This parameter specifies the HTTPS proxy.
    *  `no_proxy: - .<(sub)domain>` This parameter specifies YAML nested list (-) of addresses to exclude from the proxy.

For more information, see the [examples](#http-proxy).

**Important:** You should also configure an HTTP proxy for [Docker](https://docs.docker.com/engine/admin/systemd/#/http-proxy). 
