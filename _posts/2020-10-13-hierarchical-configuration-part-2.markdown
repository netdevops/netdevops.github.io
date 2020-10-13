---
layout: post
title:  "Hierarchical Configuration, Part 2"
date:   2018-08-13 23:00:00 -0600
tags:
- hierarchical configuration
- hconfig
- hier-config
- python
- netdevops
---
{% highlight python %}
print("Hello")
{% endhighlight %}

In [Part 1](https://netdevops.io/2018/06/07/hierarchical-configuration-part-1.html) of the hierarchical configuration blog series, I demonstrated the most basic usage of hierarchical configuration, which is rendering the configuration steps necessary to bring a device configuration into spec with the compiled template. In this blog, I'll give a refresher of part 1, but utilizing the host object in a simplified manner. I will also discuss how to tag parts of the configuration to target specific sections of config or exclude them.

In part 1, I demonstrated using hierarchical configuration in a method that forced the user to create a hconfig object per loaded configuration object. This is a lot of duplicated effort and inefficient. Since the last blog, we've updated the host object to hold the `running` and `compiled` configuration objects. Here is an example of the updated usage.

{% highlight python %}
In [1]: from hier_config.host import Host
In [1]: from hier_config.host import Host

In [2]: import yaml

In [3]: options = yaml.load(open('./tests/files/test_options_ios.yml'), Loader=yaml.SafeLoader)

In [4]: host = Host(hostname='example.rtr', os='ios', hconfig_options=options)

In [5]: host.load_config_from(config_type="running", name="./tests/files/running_config.conf", load_file=True)
Out[5]: HConfig(host=Host(hostname=example.rtr))

In [6]: host.load_config_from(config_type="compiled", name="./tests/files/compiled_config.conf", load_file=True)
Out[6]: HConfig(host=Host(hostname=example.rtr))

In [7]: host.load_remediation()
Out[7]: HConfig(host=Host(hostname=example.rtr))

In [8]: for line in host.remediation_config.all_children():
   ...:     print(line.cisco_style_text())
   ...:
vlan 3
  name switch_mgmt_10.0.3.0/24
vlan 4
  name switch_mgmt_10.0.4.0/24
interface Vlan2
  no shutdown
  mtu 9000
  ip access-group TEST in
interface Vlan3
  description switch_mgmt_10.0.3.0/24
  ip address 10.0.3.1 255.255.0.0
interface Vlan4
  mtu 9000
  description switch_mgmt_10.0.4.0/24
  ip address 10.0.4.1 255.255.0.0
  ip access-group TEST in
  no shutdown

In [9]:
{% endhighlight %}

That simply use case in itself is a very powerful. However, it's very rare that the entire difference in the config is allowed to be pushed to a device. Very often there are disruptful changes that have to be mitigated and pushed in a more graceful manner. Thus, being able to target specific sections of configs becomes necessessary.

To target specific configuration sections, a yaml file is created that describes the sections of configuration and tags them. This description is called a lineage. Below is a example tags file:

{% highlight bash %}
$ cat ./tests/files/test_tags_ios.yml
---
- lineage:
  - equals:
    - no ip http secure-server
    - no ip http server
    - vlan
    - no vlan
  add_tags: safe
- lineage:
  - startswith: interface Vlan
  - startswith:
    - description
  add_tags: safe
- lineage:
  - startswith:
    - ip access-list
    - no ip access-list
    - access-list
    - no access-list
  add_tags: manual
- lineage:
  - startswith: interface Vlan
  - startswith:
    - ip address
    - no ip address
    - mtu
    - no mtu
    - ip access-group
    - no ip access-group
    - shutdown
    - no shutdown
  add_tags: manaual
{% endhighlight %}

In the file, there are a series of lineages that target specific sections of configuration. Each section is tagged with either `safe` or `manual`. Based on the tags, you can assume that configuration lines tagged with `safe` are low risk changes and that configuration lines tagged with `manual` are higher risk changes. Items like making changes to access-lists or applying access-lists to interfaces or making mtu changes to interfaces could be disruptful, while making changes to interface descriptions is low risk.

To load the tags into the host object, the `add_tags` function is called.

{% highlight python %}
In [10]: host.load_tags(name="./tests/files/test_tags_ios.yml", load_file=True)
Out[10]:
[{'add_tags': 'safe',
  'lineage': [{'equals': ['no ip http secure-server',
     'no ip http server',
     'vlan',
     'no vlan']}]},
 {'add_tags': 'safe',
  'lineage': [{'startswith': 'interface Vlan'},
   {'startswith': ['description']}]},
 {'add_tags': 'manual',
  'lineage': [{'startswith': ['ip access-list',
     'no ip access-list',
     'access-list',
     'no access-list']}]},
 {'add_tags': 'manaual',
  'lineage': [{'startswith': 'interface Vlan'},
   {'startswith': ['ip address',
     'no ip address',
     'mtu',
     'no mtu',
     'ip access-group',
     'no ip access-group',
     'shutdown',
     'no shutdown']}]}]
{% endhighlight %}

Once the tags are loaded, the remediation can be re-rendered and the remediation can be filtered by the tags.

{% highlight python %}
In [11]: host.load_remediation()
Out[11]: HConfig(host=Host(hostname=example.rtr))
In [13]: print(host.filter_remediation(include_tags=['safe'], exclude_tags=[]))
interface Vlan3
  description switch_mgmt_10.0.3.0/24
interface Vlan4
  description switch_mgmt_10.0.4.0/24
{% endhighlight %}




[jtdub][jtdub-gh]

[jtdub-gh]: https://github.com/jtdub
