---
layout: post
title:  "Hierarchical Configuration, Part 1"
date:   2018-06-06 00:00:00 -0600
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

I've written about hierarchical configuration in the past, ([here](https://www.packetgeek.net/2016/07/network-lifecycle-management-with-hierarchical-configuration/)). We intended to maintain it as an open source project, but we really did a horrible job of it. Soon, our internal version was very different than what was maintained on the public github. 

We decided to give it another shot. This time maintaining hierarchical configuration as a installable library, available on pypi. We would then install hierarchical configuration as a requirement for our internal tooling. It's a learning curve; our first attempt at truly maintaining an open source project.

In this post, I'll go over installing hierarchical configuration and provide a basic example of how it's used. In future posts, I'll provide other examples of how to perform more advanced tasks with hierarchical configuration.

Installing
==========

Hierarchical Configuration is a python library, which is installable via pip. It currently is supported on Python 2.7 and Python 3.6. Python 2.7 goes EOL on January 1, 2020. At which point, future versions of hierarchical configuration will be Python 3+ only, as we'll likely implement python features that are only available in Python 3.

For this post, I'll be installing and working with hierarchical configuration in a python virtual environment.

{% highlight python %}
jamess-mbp:tmp jtdub$ python3 -m venv env
jamess-mbp:tmp jtdub$ source env/bin/activate
(env) jamess-mbp:tmp jtdub$
(env) jamess-mbp:tmp jtdub$ pip install pyyaml hier-config
Collecting pyyaml
Cache entry deserialization failed, entry ignored
Collecting hier-config
Cache entry deserialization failed, entry ignored
Downloading https://files.pythonhosted.org/packages/47/89/7e98fe947dad85ae5989d9fedb65fbb937c446b411e025b96b66ccccca4d/hier_config-1.3.0.tar.gz
Installing collected packages: pyyaml, hier-config
Running setup.py install for hier-config ... done
Successfully installed hier-config-1.3.0 pyyaml-3.12
(env) jamess-mbp:tmp jtdub$
{% endhighlight %}

At this point, hierarchical configuration is installed, as is pyyaml, which will be used to consume the hierarcical configuraiton options file. For the sake of demo, I'll utilize the test options and router configuration files from the hier-config github repository.

{% highlight python %}
(env) jamess-mbp:tmp jtdub$ ls
env
(env) jamess-mbp:tmp jtdub$ git clone https://github.com/netdevops/hier_config.git
Cloning into 'hier_config'...
remote: Counting objects: 1358, done.
remote: Compressing objects: 100% (64/64), done.
remote: Total 1358 (delta 6), reused 50 (delta 3), pack-reused 1285
Receiving objects: 100% (1358/1358), 7.28 MiB | 10.16 MiB/s, done.
Resolving deltas: 100% (749/749), done.
(env) jamess-mbp:tmp jtdub$ mv hier_config/tests/files/* .
(env) jamess-mbp:tmp jtdub$ ls
compiled_config.conf    hier_config     test_options_ios.yml
env         running_config.conf test_tags_ios.yml
(env) jamess-mbp:tmp jtdub$
{% endhighlight %}

Demo Code
=========

At this point, you should have a `running_config.conf`, `compiled_config.conf`, and a `test_options_ios.yml` file. These will be used for this demo. So, let's start up Python IDLE.

The first thing that we'll need to do is import the necessary libraries. These libraries are `Host` and `HConfig` from `hier_config` and `yaml`.

{% highlight python %}
(env) jamess-mbp:tmp jtdub$ python3
Python 3.6.3 (default, Oct 24 2017, 20:24:01)
[GCC 4.2.1 Compatible Apple LLVM 9.0.0 (clang-900.0.38)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> from hier_config.host import Host
>>> from hier_config import HConfig
>>> import yaml
>>>
{% endhighlight %}

The `Host` object is used for creating a host object. A host object is a data structure that describes a host. For the purposes of hierarchical configuraiton, the data structure contains the `hostname`, `os` (operating system), and `hconfig_options` properties by default, but can be easily expanded on as necessary. Let's create a host object.

{% highlight python %}
>>> options = yaml.load(open('test_options_ios.yml'))
>>> host = Host(hostname='example.rtr', os='ios', hconfig_options=options)
{% endhighlight %}

At this point, a `Host` object has been created named `host`. Below is a bit of exploring the `Host` object.

{% highlight python %}
>>> host
Host(hostname=example.rtr)
>>> dir(host)
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'facts', 'hconfig_options', 'hostname', 'os']
>>> host.hostname
'example.rtr'
>>> host.os
'ios'
>>> host.facts
{}
>>> host.hconfig_options
{'style': 'ios', 'sectional_overwrite': [], 'sectional_overwrite_no_negate': [], 'ordering': [{'lineage': [{'startswith': 'no vlan filter'}], 'order': 700}, {'lineage': [{'startswith': 'interface'}, {'startswith': 'no shutdown'}], 'order': 700}], 'indent_adjust': [], 'parent_allows_duplicate_child': [], 'sectional_exiting': [{'lineage': [{'startswith': 'router bgp'}, {'startswith': 'template peer-policy'}], 'exit_text': 'exit-peer-policy'}, {'lineage': [{'startswith': 'router bgp'}, {'startswith': 'template peer-session'}], 'exit_text': 'exit-peer-session'}, {'lineage': [{'startswith': 'router bgp'}, {'startswith': 'address-family'}], 'exit_text': 'exit-address-family'}], 'full_text_sub': [], 'per_line_sub': [{'search': '^Building configuration.*', 'replace': ''}, {'search': '^Current configuration.*', 'replace': ''}, {'search': '^! Last configuration change.*', 'replace': ''}, {'search': '^! NVRAM config last updated.*', 'replace': ''}, {'search': '^ntp clock-period .*', 'replace': ''}, {'search': '^version.*', 'replace': ''}, {'search': '^ logging event link-status$', 'replace': ''}, {'search': '^ logging event subif-link-status$', 'replace': ''}, {'search': '^\\s*ipv6 unreachables disable$', 'replace': ''}, {'search': '^end$', 'replace': ''}, {'search': '^\\s*[#!].*', 'replace': ''}, {'search': '^ no ip address', 'replace': ''}, {'search': '^ exit-peer-policy', 'replace': ''}, {'search': '^ exit-peer-session', 'replace': ''}, {'search': '^ exit-address-family', 'replace': ''}, {'search': '^crypto key generate rsa general-keys.*$', 'replace': ''}], 'idempotent_commands_blacklist': [], 'idempotent_commands': [{'lineage': [{'startswith': 'vlan'}, {'startswith': 'name'}]}, {'lineage': [{'startswith': 'interface'}, {'startswith': ['description', 'ip address']}]}], 'negation_default_when': [], 'negation_negate_with': []}
{% endhighlight %}

Here is an example of being able to extend the `Host` object.

{% highlight python %}
>>> host.facts['chassis_model'] = 'WS-C4948E'
>>> host.facts
{'chassis_model': 'WS-C4948E'}
>>>
{% endhighlight %}

With the `Host` object created. We can now create a `HConfig` object and load a device running configuration into it.

{% highlight python %}
>>> running = HConfig(host=host)
>>> running
HConfig(host=Host(hostname=example.rtr))
>>> running.load_from_file('running_config.conf')
>>> list(running.all_children())
[HConfigChild(HConfig, hostname aggr-example.rtr), HConfigChild(HConfig, ip access-list extended TEST), HConfigChild(HConfigChild, 10 permit ip 10.0.0.0 0.0.0.7 any), HConfigChild(HConfig, vlan 2), HConfigChild(HConfigChild, name switch_mgmt_10.0.2.0/24), HConfigChild(HConfig, vlan 3), HConfigChild(HConfigChild, name switch_mgmt_10.0.4.0/24), HConfigChild(HConfig, interface Vlan2), HConfigChild(HConfigChild, descripton switch_10.0.2.0/24), HConfigChild(HConfigChild, ip address 10.0.2.1 255.255.255.0), HConfigChild(HConfigChild, shutdown), HConfigChild(HConfig, interface Vlan3), HConfigChild(HConfigChild, mtu 9000), HConfigChild(HConfigChild, description switch_mgmt_10.0.4.0/24), HConfigChild(HConfigChild, ip address 10.0.4.1 255.255.0.0), HConfigChild(HConfigChild, ip access-group TEST in), HConfigChild(HConfigChild, no shutdown)]
>>>
{% endhighlight %}

With a `HConfig` instance created with the running config of `host`, we can create another instance of `HConfig` with the compiled config or intended config.

{% highlight python %}
>>> compiled = HConfig(host=host)
>>> compiled.load_from_file('compiled_config.conf')
>>> list(compiled.all_children())
[HConfigChild(HConfig, hostname aggr-example.rtr), HConfigChild(HConfig, ip access-list extended TEST), HConfigChild(HConfigChild, 10 permit ip 10.0.0.0 0.0.0.7 any), HConfigChild(HConfig, vlan 2), HConfigChild(HConfigChild, name switch_mgmt_10.0.2.0/24), HConfigChild(HConfig, vlan 3), HConfigChild(HConfigChild, name switch_mgmt_10.0.3.0/24), HConfigChild(HConfig, vlan 4), HConfigChild(HConfigChild, name switch_mgmt_10.0.4.0/24), HConfigChild(HConfig, interface Vlan2), HConfigChild(HConfigChild, mtu 9000), HConfigChild(HConfigChild, descripton switch_10.0.2.0/24), HConfigChild(HConfigChild, ip address 10.0.2.1 255.255.255.0), HConfigChild(HConfigChild, ip access-group TEST in), HConfigChild(HConfigChild, no shutdown), HConfigChild(HConfig, interface Vlan3), HConfigChild(HConfigChild, mtu 9000), HConfigChild(HConfigChild, description switch_mgmt_10.0.3.0/24), HConfigChild(HConfigChild, ip address 10.0.3.1 255.255.0.0), HConfigChild(HConfigChild, ip access-group TEST in), HConfigChild(HConfigChild, no shutdown), HConfigChild(HConfig, interface Vlan4), HConfigChild(HConfigChild, mtu 9000), HConfigChild(HConfigChild, description switch_mgmt_10.0.4.0/24), HConfigChild(HConfigChild, ip address 10.0.4.1 255.255.0.0), HConfigChild(HConfigChild, ip access-group TEST in), HConfigChild(HConfigChild, no shutdown)]
>>>
{% endhighlight %}

Now, this is where the magic comes in. We can let hierarchical configuration compare the running config to the compiled config and automatically create a set of remediation commands that will bring the device into compliance with the intended config.

{% highlight python %}
>>> remediation = running.config_to_get_to(compiled)
>>> for line in remediation.all_children():
...     print(line.cisco_style_text())
...
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
 >>>
{% endhighlight %}

Pretty awesome, right? That's hierarchical configuration at its core. There are other features from getting granular with sectional exiting, command ordering, or idempotent commands; to targeting specific parts of configuration with a tagging mechanism. I'll explain those in future blogs.

Hierarchical Configuration is on [github]([https://github.com/netdevops/hier_config]), code documentation is available [here]([https://netdevops.io/hier_config/hier_config/]), and you can ask question on networktocode.slack.com, #hier_config.

[jtdub][jtdub-gh]

[jtdub-gh]: https://github.com/jtdub
