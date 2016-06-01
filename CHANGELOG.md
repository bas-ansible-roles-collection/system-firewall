# System Firewall (`system-firewall`) - Change log

All notable changes to this role will be documented in this file.
This role adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

Remember: Make sure to update system_firewall_barc_role_version variable when a new version is released.

## [Unreleased][unreleased]

### Changed

* Updating role to use Pristine project template - BARC flavour, version 0.3.0

## 0.2.1 - 05/02/2016

### Fixed

* Missing 'system-hostname' role application in test playbook
* BARC manifest support updated to fix issues where multiple BARC roles are applied to the same machine
* Typos in BARC manifest documentation

### Removed

* Un-needed role defaults .gitignore file

## 0.2.0 - 05/02/2016

### Added

* Build badge for master branch
* Missing 'BARC_INSTALL' tag
* Missing instructions for setting up CI for the master branch
* Local facts to record this role has been applied to a system and its version, plus supporting documentation sections

### Fixed

* README formatting and typos
* Minor corrections to other files for formatting and typos
* Pinning Ansible to pre-2.0 version in CI
* Typo in test task file name

### Changed

* Name of CI playbook to fit with new conventions
* Migrating from old Ansible Galaxy namespace, 'BARC', to 'bas-ansible-roles-collection'
* Migrating from old Ansible Galaxy 'categories' to new 'tags' meta-data
* Migrating from old Repository in 'antarctica' to 'bas-ansible-roles-collection'
* Migrating from old Semaphore 'antarctica' organisation to 'bas-ansible-roles-collection'
* Simplifying CI setup tasks

## 0.1.0 - 30/11/2015

### Added

* Initial version - with support for enabling the system firewall and managing firewall services
