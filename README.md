# System Firewall (`system-firewall`)

Master:
[![Build Status](https://semaphoreci.com/api/v1/projects/a0d5bc1d-8247-4dc5-93e3-c25ecdd9f483/617819/badge.svg)](https://semaphoreci.com/antarctica/ansible-system-firewall)

Develop:
[![Build Status](https://semaphoreci.com/api/v1/projects/a0d5bc1d-8247-4dc5-93e3-c25ecdd9f483/615676/badge.svg)](https://semaphoreci.com/antarctica/ansible-system-firewall)

Configures the system firewall and manages system firewall services for a machine

**Part of the BAS Ansible Role Collection (BARC)**

## Overview

* Ensures the relevant system firewall package is installed (`ufw` on Ubuntu and `firewalld` on CentOS)
* Ensures the relevant system firewall is activated and enabled on system startup
* Optionally, enables or disables one or more firewall services (such as SSH) as exceptions in the relevant firewall

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

*This limitation is **NOT** considered to be significant. Solutions will **NOT** be actively pursued.*
*Pull requests to address this will **NOT** be considered.*

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

*This limitation is **NOT** considered to be significant. Solutions will **NOT** be actively pursued.*
*Pull requests to address this will be considered.*

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

*This limitation is **NOT** considered to be significant. Solutions will **NOT** be actively pursued.*
*Pull requests to address this will **NOT** be considered.*

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

*This limitation is **NOT** considered to be significant. Solutions will **NOT** be actively pursued.*
*Pull requests to address this will **NOT** be considered.*

See [BARC-79](https://jira.ceh.ac.uk/browse/BARC-79) for further details.

* Zones other than the default are not tested for `firewalld`

This role does support using non-default zones when managing firewall services with `firewalld` on CentOS machines.
However the tests ran as part of this role do not test this.

As this only affects testing, and it is not expected non-default zones will be typically used by services managed by 
this role, this is not a high priority to add.

*This limitation is **NOT** considered to be significant. Solutions will **NOT** be actively pursued.*
*Pull requests to address this will be considered.*

See [BARC-80](https://jira.ceh.ac.uk/browse/BARC-80) for further details.

* The direction of firewall rules on Ubuntu are not tested

This role does support setting the direction for a firewall service with `ufw` on Ubuntu machines. However the tests 
ran as part of this role do not test this.

This is because the `ufw status` command does not report the direction of a rule/service. It may be possible to 
interpret information from the underlying `iptables` rules but this has not yet been explored.

As this only affects testing, this is not a high priority to add. Additional functional tests may be able to test this
feature more broadly, though without specifically testing if the direction attribute is set correctly.

*This limitation is **NOT** considered to be significant. Solutions will **NOT** be actively pursued.*
*Pull requests to address this will be considered.*

See [BARC-81](https://jira.ceh.ac.uk/browse/BARC-80) for further details.

* *state_ufw* and *state_firewalld* take opposite arguments

I.e. The *state_firewalld* option checks if a service should be present, whereas the *state_ufw* option checks if a 
service should be deleted. This is obviously confusing and non-intuitive.

This is a consequence of the arguments offered by the underlying `ufw` and `firewalld` Ansible modules used by this 
role. It appears these modules were developed without respect for each other, leading to this issue.

A workaround for this issue would logically reverse the relevant option for either the `ufw` or `firewalld` modules, 
however I am not aware of ways to do this using Jinja2. As the arguments for both modules also require different valid
values (*enabled* vs *yes*) it is not possible to use a consistent option for this either.

*This limitation is considered to be significant. Solutions will be actively pursued.*
*Pull requests to address this will be gratefully considered and given priority.*

See [BARC-82](https://jira.ceh.ac.uk/browse/BARC-82) for further details.

* For functional tests, there is no reliable way to test if ports are opened without a service listening on such ports

These tests do not current work correctly when using {{netcat}}, as this relies on something listening on each port 
being tested. I.e. It is not enough just to open a port. Functional tests are therefore disabled.

These tests are considered important to ensuring this role is fit for purpose and achieves its stated objectives.

*This limitation is considered to be significant. Solutions will be actively pursued.*
*Pull requests to address this will be gratefully considered and given priority.*

See [BARC-83](https://jira.ceh.ac.uk/browse/BARC-83) for further details.

* For functional tests, there is no test where a firewall service should be non-contactable on both CentOS and Ubuntu

This is possible to add but preluded by the the ability to test if ports are opened at all. However the scenario this
would cover is not expected to be commonly used, and is therefore a lower priority.

*This limitation is **NOT** considered to be significant. Solutions will **NOT** be actively pursued.*
*Pull requests to address this will be considered.*

See [BARC-84](https://jira.ceh.ac.uk/browse/BARC-84) for further details.

## Usage

### BARC manifest

By default, BARC roles will record that they have been applied to a system. This is recorded using a set of 
[Ansible local facts](http://docs.ansible.com/ansible/playbooks_variables.html#local-facts-facts-d), specifically:

* `ansible_local.barc-nginx.general.role_applied` - to indicate that this role has been applied to a system
* `ansible_local.barc-nginx.general.role_version` - to indicate the version of this this role that has been applied

Note: You **SHOULD** use this feature to determine whether this role has been applied to a system.

If you do not want these facts to be set by this role, you **MUST** skip the **BARC_SET_MANIFEST** tag. No support is 
offered in this case, as other roles or use-cases may rely on this feature. Therefore you **SHOULD** not disable this
feature.

### Supported firewalls

* For CentOS based systems, `firewalld` is used and is the only firewall supported by this role
* For Ubuntu based systems, `ufw` is used and is the only firewall supported by this role

Note: Although both of these firewalls are wrappers around `iptables`, direct configuration of `iptables` is not 
supported by this role.

Despite their similarities, it is not possible to use a common firewall module to manage both `ufw` and `firewalld`.
Unfortunately, these modules, in part due to the differences in their respective firewall, do not offer the same 
arguments in terms of: 

* Argument names - some arguments have equivalents with a different name, others are only available based on the 
features of the relevant firewall. Where arguments are equivalent, a generic option will be used by this role.
* Argument values - for arguments that are similar, different firewalls accept different values (e.g. one module may 
accept 'enabled' or 'disabled', another may accept 'yes' or 'no'). There is currently no mechanism to mask these
differences, see the *limitations* section for more information.
* Argument 'logic' - for arguments that are similar, different Ansible firewall modules use opposing 'logical' values.
I.e. One module may ask, 'should this rule exist?', where a truthy value would cause the rule to exist. Whereas another
module may ask, for an equivalent argument, 'should this rule not exist?', where a truthy value would cause the rule to
not exist. This is understandably confusing and non-intuitive and is a recognised limitation, see the *limitations* 
section for more information.

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

Each firewall uses its own format for defining firewall services, as well as conventional locations to store such
service definitions.

This role does not support creating firewall service definitions, see the *limitations* section for more information.

### Managing firewall services

Each service can be made active (where its rules will be applied) or in-active (where its rules will be ignored). Some
firewalls support controlling the direction of rules (i.e. make an exception to *allow* traffic which would normally be
denied, or *deny* traffic which would normally by allowed).

This role is only intended to manage firewall services for system level utilities, such as SSH. Services for other
applications, such as a web server, should be managed by dedicated roles. See the *limitations* section for more 
information.

### Pre-enabled firewall services

Due to the requirement of this role for the system firewall to be active, the firewall service for SSH is assumed to be
enabled before this role is applied. This is assumed as without such a firewall exception, Ansible would be unable to
manage a machine.

### Typical playbook

```yaml
---

- name: configure firewall
  hosts: all
  become: yes
  vars:
    - system_firewall_rules:
      -
        name_firewalld: ssh
        name_ufw: OpenSSH
  roles:
    - BARC.system-firewall
```

Note: This example is somewhat contrived as the SSH firewall service will be typically enabled already. As this role is
only intended for managing system level firewall services, an actual typical example would not manage any service, it
would like this:

```yaml
---

- name: setup system firewall
  hosts: all
  become: yes
  vars: []
  roles:
    - BARC.system-firewall
```

### Tags

BARC roles use standardised tags to control which aspects of an environment are changed by roles. Where relevant, tags
will be applied at a role, or task(s) level, as indicated below.

This role uses the following tags, for various tasks:

TODO: Add other tags for package configurations etc.

* [**BARC_CONFIGURE**](https://antarctica.hackpad.com/BARC-Standardised-Tags-AviQxxiBa3y#:h=BARC_CONFIGURE)
* [**BARC_CONFIGURE_SYSTEM**](https://antarctica.hackpad.com/BARC-Standardised-Tags-AviQxxiBa3y#:h=BARC_CONFIGURE_SYSTEM)
* [**BARC_CONFIGURE_FIREWALL**](https://antarctica.hackpad.com/BARC-Standardised-Tags-AviQxxiBa3y#:h=BARC_CONFIGURE_FIREWALL)
* [**BARC_CONFIGURE_SECURITY**](https://antarctica.hackpad.com/BARC-Standardised-Tags-AviQxxiBa3y#:h=BARC_CONFIGURE_SECURITY)
* [**BARC_CONFIGURE_PACKAGES**](https://antarctica.hackpad.com/BARC-Standardised-Tags-AviQxxiBa3y#:h=BARC_CONFIGURE_PACKAGE)
* [**BARC_INSTALL**](https://antarctica.hackpad.com/BARC-Standardised-Tags-AviQxxiBa3y#:h=BARC_INSTALL)
* [**BARC_INSTALL_PACKAGES**](https://antarctica.hackpad.com/BARC-Standardised-Tags-AviQxxiBa3y#:h=BARC_INSTALL_PACKAGE)
* [**BARC_SET_MANIFEST**](https://antarctica.hackpad.com/BARC-Standardised-Tags-AviQxxiBa3y#:h=BARC_SET_MANIFEST)

### Variables

#### *system_users_users*
#### *BARC_role_name*

* **MUST NOT** be specified
* Specifies the name of this role within the BAS Ansible Roles Collection (BARC) used for setting local facts
* See the *BARC roles manifest* section for more information
* Example: system-firewall

#### *BARC_role_version*

* **MUST NOT** be specified
* Specifies the name of this role within the BAS Ansible Roles Collection (BARC) used for setting local facts
* See the *BARC roles manifest* section for more information
* Example: 2.0.0


* **MAY** be specified

A list of operating system user accounts, and their properties, to be managed by this role.

Structured as a list of items, with each item having the following properties:

* *name_ufw*
    * **MUST** be specified where a firewall service is to be managed on Ubuntu/`ufw`, otherwise **MUST NOT** be
    specified
    * Specifies the name of the firewall application (service) to be managed
    * The presence of this option determines this item applies to Ubuntu/`ufw` machines
    * Values **MUST** be valid user firewall applications (services), as determined by the `ufw` firewall
    * Example: `foo`
* *rule*
    * **MAY** be specified
    * Specifies whether traffic associated with the firewall service specified by the *name_ufw* variable is to be 
    allowed or denied
    * Values **MUST** use one of these options, as determined by the `ufw` firewall:
      * `allow`
      * `deny`
    * Where not specified, a value of `allow` will be assumed
    * Default: `allow`
* *direction*
    * **MAY** be specified
    * Specifies how traffic associated with the firewall service specified by the *name_ufw* variable is to be managed 
    in terms of its direction
    * Values **MUST** use one of these options, as determined by the `ufw` firewall:
      * `in`
      * `out`
      * `incoming`
      * `outgoing`
      * `routed`
    * Typically, the values, `in` and `out` will be used, with `in` the most common
    * Where not specified, a value of `in` will be assumed
    * Default: `in`
* *state_ufw*
    * **MAY** be specified
    * Specifies if the rules associated with the firewall service specified by the *name_ufw* variable should exist
    * I.e. Where this option is `yes`, associated rules will be removed, if `no` associated rules will be kept
    * Note: The logical state of this option differs from the *state_firewalld* option as, though similar in purpose, 
    these options ask a logically different question.
    * Values **MUST** use one of these options, as determined by the `ufw` firewall:
      * `yes`
      * `no`
    * Values **MUST** be quoted to prevent Ansible coercing values to True/False which is invalid for this option
    * Where not specified, a value of `no` will be assumed
    * Default: `no`
* *name_firewalld*
    * **MUST** be specified where a firewall service is to be managed on CentOS/`firewalld`, otherwise **MUST NOT** be
    specified
    * Specifies the name of the firewall service to be managed
    * The presence of this option determines this item applies to CentOS/`firewalld` machines
    * Values **MUST** be valid user firewall services, as determined by the `firewalld` firewall
    * Example: `foo`
* *zone*
    * **MAY** be specified
    * Specifies in which firewall zone rules associated with the service specified by the *name_firewalld* variable 
    will be managed within (e.g. created or removed)
    * Values **MUST** be a valid firewall zone, as determined by the `firewalld` firewall
    * Where not specified, the default zone configured by the operating system will be used, this defaults to `public`
    * Example: `public`
* *permanent*
    * **MAY** be specified
    * Specifies if the rules associated with the service specified by the *name_firewalld* variable should apply only
    until the machine is restarted, or if it should persist until changed or removed
    * Values **MUST** use one of these options, as determined by the `firewalld` firewall:
      * `True`
      * `False`
    * Values **SHOULD** not be quoted to ensure the value is treated as a binary data-type, as opposed to a string
    * Where not specified, a value of `True` will be assumed
    * Default: `True`
* *state_firewalld*
    * **MAY** be specified
    * Specifies if the rules associated with the service specified by the *name_firewalld* variable should exist
    * I.e. Where this option is `yes`, associated rules will be kept, if `no` associated rules will be removed
    * Note: The logical state of this option differs from the *state_firewalld* option as, though similar in purpose, 
    these options ask a logically different question.
    * Values **MUST** use one of these options, as determined by the `firewalld` firewall:
      * `enabled`
      * `disabled`
    * Where not specified, a value of `enabled` will be assumed
    * Default: `enabled`

Default: `[]` - an empty list

## Developing

### Issue tracking

Issues, bugs, improvements, questions, suggestions and other tasks related to this package are managed through the 
[BAS Ansible Role Collection](https://jira.ceh.ac.uk/projects/BARC) (BARC) project on Jira.

This service is currently only available to BAS or NERC staff, although external collaborators can be added on request.
See our contributing policy for more information.

### Source code

All changes should be committed, via pull request, to the canonical repository, which for this project is:

`ssh://git@stash.ceh.ac.uk:7999/barc/system-firewall.git`

A mirror of this repository is maintained on GitHub. Changes are automatically pushed from the canonical repository to
this mirror, in a one-way process.

`git@github.com:antarctica/ansible-system-firewall.git`

Note: The canonical repository is only accessible within the NERC firewall. External collaborators, please make pull 
requests against the mirrored GitHub repository and these will be merged as appropriate.

### Contributing policy

This project welcomes contributions, see `CONTRIBUTING.md` for our general policy.

The [Git flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow/) 
workflow is used to manage the development of this project:

* Discrete changes should be made within feature branches, created from and merged back into develop 
(where small changes may be made directly)
* When ready to release a set of features/changes, create a release branch from develop, update documentation as 
required and merge into master with a tagged, semantic version (e.g. v1.2.3)
* After each release, the master branch should be merged with develop to restart the process
* High impact bugs can be addressed in hotfix branches, created from and merged into master (then develop) directly

### Release procedure

See [here](https://antarctica.hackpad.com/BARC-Overview-and-Policies-SzcHzHvitkt#:h=Release-procedures) for general 
release procedures for BARC roles.

## Licence

Copyright 2015 NERC BAS.

Unless stated otherwise, all documentation is licensed under the Open Government License - version 3. All code is
licensed under the MIT license.

Copies of these licenses are included within this role.
