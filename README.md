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
you are version *0.3.0* or greater to satisfy the first requirement. Previous versions did not activate the firewall by
default.

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

This is an intentional limitation.

(too many differences between CentOS and Ubuntu).

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this will be considered however.*

See [BARC-]() for further details.

* Managing ports is not supported, firewall services must be used

This is an intentional limitation.

(see BARC-69 for draft comments).

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this limitation will **not** be considered.*

See [BARC-]() for further details.

* This role does not support creating services

This is an intentional limitation.

(see BARC-69 for draft comments).

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this limitation will **not** be considered.*

See [BARC-]() for further details.

* Zones other than the 'public' zone for `firewalld` are not tested

(It is possible to specify another zone, but this is not covered by this roles tests currently as it's quite a lot of 
work to add (get rules for every zone and check)).

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this will be considered however.*

See [BARC-]() for further details.

* The direction of firewall rules on Ubuntu are not tested

(No information is given by `ufw status` to test against.)

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this will be considered however.*

See [BARC-]() for further details.

* *state_ufw* and *state_firewalld* take opposite arguments

(I.e. state_firewalld asks if a rule should be present, state_ufw asks if a rule be deleted)

*This limitation is considered to be significantly limiting, and a solution will be actively pursued. Pull requests 
addressing this will be considered however.*

See [BARC-]() for further details.

* For functional tests, there is no reliable way to test if ports are opened without a service listening on such ports

(These tests do not current work correctly as netcat relies on something listening on each port being tested.
I.e. It is not enough to simply open a port, a service has to listen on it, these tests are therefore disabled.
Functional tests therefore disabled.)

*This limitation is considered to be significantly limiting, and a solution will be actively pursued. Pull requests 
addressing this will be considered however.*

See [BARC-]() for further details.

* For functional tests, there is a gap for services that should be non-contactable on both CentOS and Ubuntu

*This limitation is **not** considered to be significantly limiting, and a solution will not be actively pursued. Pull 
requests addressing this will be considered however.*

See [BARC-]() for further details.






TODO: Review



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
this role, but is relative easy to do if you need to. This role will not interfere with any such modifications if made.

### Firewall rules

TODO: This needs updating as firewalld offers no way to control the direction of a rule (at least not in the same way
as UFW).

Broadly speaking both `firewalld` and `ufw` manage firewall rules in the same way, in that they use the concept of a 
*service* or *application* (*service* will be used as a generic term within this documentation), to represent logical 
groupings of firewall rules to make exceptions to the default firewall policy.

Examples of services include things such as SSH or a web-server. Examples of firewall rules include things such as,
specifying port 22 over the TCP protocol.

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


