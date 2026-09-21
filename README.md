# tobias_richter.librenms_agent

[![Build Status](https://travis-ci.org/tobias-richter/ansible-librenms-agent.svg?branch=master)](https://travis-ci.org/tobias-richter/ansible-librenms-agent)

This role setups snmp and extensions on agents which then can be polled
by librenms. See also
[tobias_richter.librenms](https://galaxy.ansible.com/tobias_richter/librenms)
for setting up librenms.

This role is tested with the following snmp extensions (and may work
with more):
* osupdate
* osupdate unpriviledged
* distro
* entropy
* apache-stats.py
* mysql
* certificate.py
* fail2ban
* ntp-client.sh
* ntp-server.sh
* pi-hole
* ups-nut.sh
* smart
* postfixdetailed (with https://github.com/tobias-richter/librenms-agent)
* postfix-queues (with https://github.com/tobias-richter/librenms-agent)
* raspberrypi
* redis.py (with https://github.com/librenms/librenms-agent/pull/327 or https://github.com/tobias-richter/librenms-agent)

Beside the snmp extensions the role is tested with the following checkmk
extensions (and may work with more):
* dpkg
* apache
* proxmox

## Requirements

This role requires Ansible 2.7 or higher.

## Role Variables

See [defaults/main.yml](defaults/main.yml) for the documented role
variables.

Mandantory variables are:

* `librenms_agent_snmp_user`
* `librenms_agent_snmp_password`
* `librenms_agent_snmp_encryption`

### Quiet sudo logging

The SNMP extends that need root (postfix, wireguard, raspberry, ...) run through `sudo`, which writes
the command and a PAM session open/close line to the journal on every poll. With
`librenms_agent_snmp_sudo_quiet: true` (default) the role installs
`/etc/sudoers.d/<snmp user>_quiet` with `Defaults:<snmp user> !log_allowed, !pam_session`.
Denied calls are still logged.

* Requires sudo >= 1.8.29. The role reads `sudo -V` and skips the file (and removes an existing one)
  on older versions.
* `sudo-rs` (default `sudo` on Ubuntu 26.04) does not know `log_allowed`, so the file is skipped there
  too and the allowed calls are still logged.
* Set `librenms_agent_snmp_sudo_quiet: false` to always remove the file.

The sudoers files for the extends are written with mode `0440` and checked with `visudo -cf`.

## Configure SNMP Extensions

This is the complete set of configuration options:
          # custom name, must be unique
        - name: pi-hole
          # must match an available script in https://github.com/tobias-richter/librenms-agent/tree/master/snmp
          script: pi-hole
          # controls the force flag of the copy task
          copy_force: no
          # a simple comment
          comment: enable pi hole in librenms
          # custom arguments that are passed to the script call
          args: -c  
          # custom packages that must be present for the script
          packages:
            - jq

## Examples for SNMP extensions:

        librenms_agent_snmp_extensions:
        - name: osupdate
          script: osupdate
          comment: enable os updates in librenms
          state: absent
        - name: osupdate
          executable: /usr/local/bin/osupdates-unpriv-gather.sh
          comment: enable unpriv os updates in librenms
          state: present
        - name: ".1.3.6.1.4.1.2021.7890.1 distro"
          script: distro
          comment: enable distribution
        - name: entropy
          script: entropy.sh
          comment: monitor entropy
        - name: apache
          script: apache-stats.py
          comment: enable apache stats for librenms
        - name: certificate
          script: certificate.py
          comment: enable certificate check for librenms
        - name: mysql
          script: mysql
          comment: enable mysql in librenms
        - name: fail2ban
          script: fail2ban
          comment: enable fail2ban stats for librenms
          args: -c       
        - name: pi-hole
          script: pi-hole
          copy_force: no
          comment: enable pi hole in librenms
          packages:
            - jq
        - name: raspberry
          script: raspberry.sh
          script_prefix: "/usr/bin/sudo /bin/sh "
          comment: enable raspberry in librenms
        - name: smart
          script: "smart"
          args: "-c /etc/snmp/snmpd.d/smart.config"
          comment: enable smart in librenms
        - name: ntp-server
          script: ntp-server.sh
          comment: enable ntp-server stats for librenms
        - name: ntp-client
          script: ntp-client
          comment: enable ntp-client stats for librenms
        - name: ups-nut
          script: ups-nut.sh
          comment: enable monitoring for UPS     

## Examples for checkmk extensions:

        librenms_agent_check_mk_extensions
        - script: dpkg
        - script: apache
        - script: proxmox    

## Example Playbook

This playbook setups a snmpd agent for librenms with osupdate, distro and dpkg extension.

    - hosts: librenms_agent
	  roles:
	    - role: tobias_richter.librenms_agent
          librenms_agent_snmp_extensions:
            - name: osupdate
              executable: /usr/local/bin/osupdates-unpriv-gather.sh
              comment: enable unpriv os updates in librenms
            - name: ".1.3.6.1.4.1.2021.7890.1 distro"
              script: distro
              comment: enable distribution
          librenms_agent_check_mk_extensions
            - script: dpkg                      