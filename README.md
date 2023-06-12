ansible-nebula
=========

This role installs and deploys a configuration for [Nebula](https://github.com/slackhq/nebula)

Requirements
------------

Currently you need to generate and deploy certificates before running this (see example)

Supported Nebula Version
------------------------

Currently this role is tested against version `1.5.0`

Role Variables
--------------

| Variable Name | Type | Purpose | Default | Required |
|---|---|---|---|---|
| `nebula_version` | String | Version to download | `1.5.0` | Yes |
| `nebula_force_install` | Boolean | Force overwrite of the existing nebula binary | `false` | No |
| `ca` | String | Path to CA file | NA | Yes |
| `cert` | String | Path to Certificate | NA | Yes |
| `key` | String | Path to Certificate Key| NA | Yes |
| `blocklist` | List | List of Blocklisted certificate hashes | NA | No |
| `lighthouses` | String | Static hosts for discovery | "{{ groups['nebula_lighthouses'] }}" | No |
| `lighthouses_override` | List | List of static hosts for discovery | NA | No |
| `lighthouse.am_lighthouse` | Boolean | Is this instance a Lighthouse | `false` | Yes |
| `lighthouse.serve_dns` | Boolean | Should this instance serve DNS | `false` | Yes |
| `lighthouse.interval` | Integer | Report interval to lighthouses | `60` | No |
| `listen.host` | String | IP to listen on | `0.0.0.0` | Yes |
| `listen.port` | Integer | Port to listen on | `4242` | Yes |
| `listen.batch` | Integer | Sets the max number of packets to pull from the kernel for each syscall | `64` | Yes |
| `listen.read_buffer` | Integer | Configure socket buffers for the udp side | NA | No |
| `listen.write_buffer` | Integer | Configure socket buffers for the udp side | NA | No |
| `punchy` | Boolean | Punchy continues to punch inbound/outbound at a regular interval to avoid expiration of firewall nat mappings | `true` | Yes |
| `punch_back` | Boolean | punch_back means that a node you are trying to reach will connect back out to you if your hole punching fails | `true` | Yes |
| `cipher` | String | Cipher allows you to choose between the available ciphers for your network. | NA | No |
| `local_range` | String | Local range is used to define a hint about the local network range | NA | No |
| `sshd.enabled` | Boolean | sshd can expose informational and administrative functions via ssh | NA | No |
| `sshd.listen` | String | IP / Port for admin SSH functions | NA | No |
| `relay.relays` | List | IP of hosts to use as a relay | NA | No |
| `relay.am_relay` | String | Indicate whether host should act as a relay  | `false` | No |
| `relay.use_relays` | String | Indicate whether host should attempt to connect through relays | `true` | No |
| `metrics.prometheus` | Boolean | Enables prometheus server | NA | No |
| `outbound` | List | Outbound rules for the built in firewall | `See Below` | Yes |
| `inbound` | List | Inbound rules for the built in firewall | `See Below` | Yes |


Firewall rule example
```
outbound:
  - port: any
    proto: any
    host: any

inbound:
  - port: any
    proto: icmp
    host: any
```


Dependencies
------------

None

Example Playbook
----------------

```
---
- hosts: all
  remote_user: root
  vars:
    lighthouses:
      - nebula_ip: 10.255.0.1
        external_addr: 123.231.1.2
    lighthouse:
      nodes:
        - 10.255.0.1
  pre_tasks:
    - name: Create Nebula directory
      file:
        path: /etc/nebula
        state: directory
        mode: '0750'
    - name: Deploy Nebula certificates
      copy:
        src: files/{{item}}
        dest: /etc/nebula/{{item}}
        owner: root
        group: root
        mode: '0600'
      with_items:
        - ca.crt
        - host.crt
        - host.key
  roles:
    - ansible-nebula
```

```
---
- hosts: all
  remote_user: root
  vars:
    lighthouses:
      - nebula_ip: 10.255.0.1
        external_addr: 123.231.1.2
  roles:
    - ansible-nebula
```

Optionally you may declare a lighthouse with a non-default external port
```
---
- hosts: all
  remote_user: root
  vars:
  lighthouse:
    am_lighthouse: yes
  lighthouses:
    - nebula_ip: 10.255.0.1
      external_addr: 123.231.1.2
      external_port: 4242
  roles:
    - ansible-nebula
```

License
-------

MIT

Author Information
------------------

This role is provided as is, Nebula is maintained by Slack and the community
