# System Firewall (`system-firewall`) - Change log

All notable changes to this role will be documented in this file.
This role adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

Note: Developers - make sure to set the `BARC_role_version` variable when releasing new versions of this role.

## [Unreleased][unreleased]

### Fixed

* BARC manifest support updated to fix issues where multiple BARC roles are applied to the same machine
* Typos in BARC manifest documentation

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
