# System Firewall (`system-firewall`)

## Requirements

This role is only supported when used with BAS base image greater than the minimum version specified below:

| Base Box / Template  | Minimum Version | Notes |
| -------------------- | --------------- | ----- |
| `antarctica/trusty`  | 3.0.0           | -     |
| `antarctica/centos7` | 0.3.0           | -     |

## Limitations

* It is not possible to change default firewall policies

(This is by design)

* Zones other than the 'public' zone in firewalld are not tested

It is possible to specify another zone, but this is not covered by this roles tests currently as it's quite a lot of 
work to add (get rules for every zone and check).

* The direction of firewall rules on Ubuntu are not tested

No information is given by `ufw status` to test against.

* *state_ufw* and *state_firewalld* take reverse arguments

I.e. state_firewalld asks if a rule should be present, state_ufw asks if a rule be deleted :/

* For functional tests, there is no reliable way to test if ports are opened without a service listening on such ports

These tests do not current work correctly as netcat relies on something listening on each port being tested.
I.e. It is not enough to simply open a port, a service has to listen on it, these tests are therefore disabled.

Functional tests therefore disabled.

* For functional tests, there is a gap for services that should be non-contactable on both CentOS and Ubuntu

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


