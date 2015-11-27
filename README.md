# System Firewall (`system-firewall`)

Develop:
[![Build Status](https://semaphoreci.com/api/v1/projects/a0d5bc1d-8247-4dc5-93e3-c25ecdd9f483/615676/badge.svg)](https://semaphoreci.com/antarctica/ansible-system-firewall)

Configures the system firewall and manages system firewall services for a machine

**Part of the BAS Ansible Role Collection (BARC)**

## Overview

* Ensures the relevant system firewall package is installed (`ufw` on Ubuntu and `firewalld` on CentOS)
* Optionally, enables or disables one or more firewall services (such as SSH) as exceptions in the relevant firewall
* Ensures the relevant system firewall is activated and enabled on system startup

## Quality Assurance

This role uses manual and automated testing to ensure the features offered by this role work as advertised. 
See `tests/README.md` for more information.

## Dependencies

* None

## Requirements

* If managing CentOS systems, the `firewalld` service must already be active
* For any system, if the system firewall is already running an exception must be in place to allow SSH connections

If you are using the BAS maintained CentOS 7 base image, 
from the [BAS Packer VM Templates project](https://github.com/antarctica/packer-vm-templates), you will need to ensure
you are using version *0.3.0* or greater to satisfy the first requirement. Previous versions did not activate the 
firewall by default.

## Limitations

* The system firewall cannot be disabled

This is an intentional limitation. Other roles, which will configure the system firewall for specific purposes, depend 
on this role to ensure the system firewall is correctly enabled.

Were it possible to disable the system firewall using this role, dependent roles would need to introduce additional
logic to either, ensure this role configured the firewall, making an option in this role redundant, or add logic to 
check if the firewall is disabled, and then ensure firewall configuration tasks are skipped. Across multiple roles, the 
effort to implement, test and support this additional logic is significant. Therefore this role will always ensure the
system firewall is enabled.

In cases where the system firewall should not be modified, or should be modified in a context specific way, it is 
recommended to use the Ansible tags assigned to tasks in this role, and others, to skip the tasks in this role and 
others. By using the conventional tags used in this role, other roles will be automatically adjusted such as to not
rely on this role ensuring an active firewall is available.

Note: It is strongly recommended that you always use a firewall, whether it is configured by this role or not.

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this limitation will **not** be considered.*

See [BARC-76](https://jira.ceh.ac.uk/browse/BARC-76) for further details.

* It is not possible to change default firewall policies

This is an intentional limitation. This ability, setting default actions for different types/sources of traffic, is not
consistent across system firewall providers supported by this role. `ufw` on Ubuntu implements this using the concept 
of firewall *policies*, whereas `firewalld` on CentOS use a concept of *zones*.

These implementations are different enough that there is no intuitive, or obvious, way for this role to support both. 
The defaults used for each implementation are considered to be reasonable for most situations.

Where these default implements do need to be modified, this role will not interfere with any changes you may make. 
When managing firewall services for `firewalld` this role will use always use the default zone, as it is configured on 
each machine. By default this will be the *public* zone, but if different, another zone will be used instead.

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this will be considered however.*

See [BARC-77](https://jira.ceh.ac.uk/browse/BARC-77) for further details.

* Managing ports is not supported, firewall services must be used

This is an intentional limitation. This ensures that any firewall exceptions that are made have meta-data associated 
with them to explain what is being excepted and why. This allows others to, at a glance, and without consulting any
project documentation for example, understand, broadly, the state of the system firewall.

Meta-data in this case includes attributes such as the name of the service and a longer description. A firewall service
definition will include this meta-data as well as details of the various port(s) that are needed (including multiple 
ports, port ranges and specific protocols if relevant).

It is accepted that this limitation forces additional work in order to manage the system firewall using this role. 
However, in most cases, it is expected that other roles that need to make any firewall exceptions will create any 
firewall services, using templates to set custom port information for example. Where this is the case the impact of 
this limitation should be negligible.

For some services, such as SSH, the raw port information would typically be enough information, however where a custom
port is used for SSH, or for less common services, having this additional information aids in auditing and debugging 
scenarios immensely.

Where this limitation is too great, it is possible to use the `ufw` and `firewalld` Ansible modules directly to allow
any ports as needed outside of this role. Unless these exceptions conflict with a firewall service managed by this 
role, this role will not interfere with such exceptions.

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this limitation will **not** be considered.*

See [BARC-78](https://jira.ceh.ac.uk/browse/BARC-78) for further details.

* This role does not support creating services

This is an intentional limitation. It is assumed services for all system related services (such as SSH) will exist by 
default and so can simply be enabled or disabled by this role.

This role will support enabling any pre-defined services that exist, but will not assist in creating or changing these 
services, except to enable or disable them. Other, package related, roles should enable/disable firewall services 
relevant to them, rather than this role.

E.g. The `apache2` role will manage firewall services to enable HTTP (typically on port 80) and HTTPS (typically on 
port 443). Predefined firewall services typically include services for things like HTTP and HTTPS. Though this role 
could be used to enable/disable such services, it is recommended that this is done by roles that manage the services 
binding to these protocols (such as a web-server). This allows for non-conventional ports to be used as a web-server 
role can create a firewall service with the relevant ports specified (e.g. through Ansible templating). This approach 
also means the name of the web-server can be specified when inspecting firewall exceptions, which is more useful than 
a generic a generic name such as 'HTTP'.

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this limitation will **not** be considered.*

See [BARC-79](https://jira.ceh.ac.uk/browse/BARC-79) for further details.

* Zones other than the default are not tested for `firewalld`

This role does support using non-default zones when managing firewall services with `firewalld` on CentOS machines.
However the tests ran as part of this role do not test this.

As this only affects testing, and it is not expected non-default zones will be typically used by services managed by 
this role, this is not a high priority to add.

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this will be considered however.*

See [BARC-80](https://jira.ceh.ac.uk/browse/BARC-80) for further details.

* The direction of firewall rules on Ubuntu are not tested

This role does support setting the direction for a firewall service with `ufw` on Ubuntu machines. However the tests 
ran as part of this role do not test this.

This is because the `ufw status` command does not report the direction of a rule/service. It may be possible to 
interpret information from the underlying `iptables` rules but this has not yet been explored.

As this only affects testing, this is not a high priority to add. Additional functional tests may be able to test this
feature more broadly, though without specifically testing if the direction attribute is set correctly.

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this will be considered however.*

See [BARC-81](https://jira.ceh.ac.uk/browse/BARC-80) for further details.

* *state_ufw* and *state_firewalld* take opposite arguments

I.e. The *state_firewalld* option checks if a service should be present, whereas the *state_ufw* option checks if a 
service should be deleted. This is obviously confusing and non-intuitive.

This is a consequence of the arguments offered by the underlying `ufw` and `firewalld` Ansible modules used by this 
role. It appears these modules were developed without respect for each other, leading to this issue.

A workaround for this issue would logically reverse the relevant option for either the `ufw` or `firewalld` modules, 
however I am not aware of ways to do this using Jinja2. As the arguments for both modules also require different valid
values (*enabled* vs *yes*) it is not possible to use a consistent option for this either.

*This limitation is considered to be significantly limiting, and a solution will be actively pursued. Pull requests 
addressing this will be considered however.*

See [BARC-82](https://jira.ceh.ac.uk/browse/BARC-82) for further details.

* For functional tests, there is no reliable way to test if ports are opened without a service listening on such ports

These tests do not current work correctly when using {{netcat}}, as this relies on something listening on each port 
being tested. I.e. It is not enough just to open a port. Functional tests are therefore disabled.

These tests are considered important to ensuring this role is fit for purpose and achieves its stated objectives.

*This limitation is considered to be significantly limiting, and a solution will be actively pursued. Pull requests 
addressing this will be considered however.*

See [BARC-83](https://jira.ceh.ac.uk/browse/BARC-83) for further details.

* For functional tests, there is no test where a firewall service should be non-contactable on both CentOS and Ubuntu

This is possible to add but preluded by the the ability to test if ports are opened at all. However the scenario this
would cover is not expected to be commonly used, and is therefore a lower priority.

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this will be considered however.*

See [BARC-84](https://jira.ceh.ac.uk/browse/BARC-84) for further details.

## Usage

### Supported firewalls

* For CentOS based systems, `firewalld` is used and is the only firewall supported by this role
* For Ubuntu based systems, `ufw` is used and is the only firewall supported by this role

Note: Although both of these firewalls are wrappers around `iptables`, direct configuration of `iptables` is not 
supported by this role.

### Default policies

Both CentOS and Ubuntu ship with default policies for their respective firewalls. Though the concepts used to implement
these policies differ (CentOS/`firewalld` uses *zones*, Ubuntu/`ufw` uses *policies*) the practical are the same:

* Incoming traffic, on all ports and protocols, is blocked
* Outgoing traffic, on all ports and protocols, is allowed

Modifying these default policies (rather than making exceptions for specific services) is not currently supported by 
this role, see the *limitations* section for more information.

### Firewall services

Broadly speaking both `ufw` and `firewalld` manage firewall rules in the same way, in that they use the concept of a 
*service* or *application* (*service* will be used as a generic term within this documentation), to represent logical 
groupings of firewall rules to make exceptions to the default firewall policy.

Examples of services include things such as SSH or a web-server. Examples of firewall rules include things such as,
specifying port 22 over the TCP protocol.

Despite their similarities, it is not possible to use a common firewall module to manage both `ufw` and `firewalld`.





--- edited to here ---





TODO: This needs updating as firewalld offers no way to control the direction of a rule (at least not in the same way
as UFW).

These services can be *enabled* or *disabled* and *allowed* or *denied*. That is to say, the firewall can be told to 
consider  rules within a service, *enable*, or ignore them, *disable*. Where a rules for a service are considered, the 
firewall will either permit, *allow*, connections to the ports specified by a service, or reject, *deny*, them.

This allows for exceptions to allow connections which are usually blocked (i.e. incoming connections, in which case 
enabled/allowed would be used), or where connections are usually allowed, (i.e. outgoing connections, in which case 
enabled/denied would be used).

As these concepts are similar between both supported firewalls, a common approach is used for specify where services 
should be enabled or disabled and allowed or denied.

Sadly the terms used for these properties do differ between providers and their associated Ansible modules, as do in 
many cases the names of services, such as SSH. This role tends to expose such differences between modules, where they 
exist, to avoid introducing ambiguity and allowing knowledge of either provider/module to be used directly. Some 
differences are masked by this role however, such as:

* *name* - The name of the firewall service being managed, AKA *service* (`firewalld` module)
* *action_* - Whether traffic matching rules for a service should be allowed or denied, AKA *state* (`firewalld` 
module) or *rule* (`ufw` module)

In some cases an individual firewall may require some options the other does not use. In these cases you can 
omit non-relevant values if you are only targeting machines that use the other firewall, if you are targeting a mixture
of machines, you can safely set the option and it will only be used where needed.

As an example, setting *system_firewall_rules* to the example below, will enable/allow incoming connections to the SSH 
service for both `firewalld` and `ufw`:

```yml
system_firewall_rules:
  - name_firewalld: ssh
    name_ufw: OpenSSH
```

### Pre-enabled services

When using suitable BAS base images some firewall services will be automatically enabled prior to this role being 
applied. These services can theoretically be disabled using this role, or manually, but doing so may cause Ansible to 
break is therefore not recommended. The following services are pre-enabled:

* *SSH* - Required for Ansible to communicate with a machine (AKA *ssh* to `firewalld` and *OpenSSH* to `ufw`)


