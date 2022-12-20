# PTP

Arista best practices are used, simplifying configuration of several global and interface-specific PTP settings:

- PTP can be enabled on various levels of the AVD configuration:
  - fabric level
  - per node_group
  - per node

- Only when explicitly enabled, will the following global PTP settings take effect:
  - PTP mode boundary is used by default.
  - One of three different PTP profiles can be used:
    - AES67
    - SMPTE2059-2
    - AES67-R16-2016 (used by default if no profile is specified)

      The profile will apply PTP parameters for the following interfaces:
      - All routed interfaces
      - Individual PTP-enabled interfaces for connected endpoints

- Defaults used when PTP is enabled:
  - All interfaces between leaf and spine switches participate in the Best Master Clock Algorithm (BMCA).
  - All interfaces used for endpoints with ptp specifically enabled use `ptp role master`.
  - PTP priority 1 and priority 2 are automatically set based on the node_type and switch_id.
  - PTP Clock Identity is automatically set based on a prefix (00:1C:73 by default) + PTP priority 1 and priority 2.

## Enabling PTP

PTP must be specifically enabled:

- on the fabric level, for example FABRIC.yml

  ```yml
  ptp:
    enabled: true
  ```

- per node_group, for example for all spine nodes in SPINES.yml

  ```yml
  spine:
    defaults:
      ptp:
        enabled: true
  ```

- per node for a specific node/device inside a node_group, for example for a specific leaf in LEAFS.yml

  ```yml
  nodes:
    leaf1:
      ptp:
        enabled: true
  ```

> **Please note:** If present, you need to remove the legacy PTP notation shown below, for example for all spine nodes.
> If this is ***not*** removed, the PTP profile-specific interface configuration will not be applied to the interfaces between leaf and spine switches.

```yml
spine:
  defaults:
    uplink_ptp:
      enable: true
```

## Fabric wide PTP settings

There are three settings that can be specified for the entire topology to greatly simplify the configuration of PTP:

```yml
ptp:
  enabled: < true | false | default -> false >
  profile: < aes67 | smpte2059-2 | aes67-r16-2016 | default -> aes67-r16-2016  >
  domain: < 0-255 | default -> 127 >
```

All can also be defined on a more specific level, if the network design requires this.

### PTP Profiles

Based on the PTP profile selection the following parameters are applied to all interfaces between spine and leaf switches:

`profile: aes67` is the *slow* PTP profile setting the following interface parameters:

- ptp enable
- ptp sync-message interval 0 (1 message every second)
- ptp announce interval 2 (1 message every 4th second)
- ptp transport ipv4
- ptp announce timeout 3
- ptp delay-req interval 0 (1 message every second)

`profile: smpte2059-2` is the *fast* PTP profile setting the following interface parameters:

- ptp enable
- ptp sync-message interval -4 (16 messages every second)
- ptp announce interval -2 (4 messages every second)
- ptp transport ipv4
- ptp announce timeout 3
- ptp delay-req interval -4 (16 messages every second)

`profile: aes67-r16-2016` is the ***default*** PTP profile, setting the following interface parameters:

- ptp enable
- ptp sync-message interval -3 (8 messages every second)
- ptp announce interval 0 (1 message every second)
- ptp transport ipv4
- ptp announce timeout 3
- ptp delay-req interval -3 (8 messages every second)

## Group or device specific PTP settings

### PTP Priorities

#### Automatic PTP priorities

Regardless of the PTP profile selection, the PTP priorities are automatically generated by default:

- Priority 1 is set automatically based on node_type.
- Priority 2 is set automatically based on node id. (modulus 256)

As shown in the table below, PTP priority 1 is defined by default, based on the node_type setting, which currently is either `spine` or `l3leaf`.
Spine switches will have a PTP priority 1 of `20` and leaf switches will have a PTP priority 1 of `30`. This supports the recommended Amber/Blue topologies where PTP GrandMasters are connected to the spine switches directly.
All other node_types will have a default PTP priority 1 of `127` to ensure they don't win the BMCA by accident.

| Node_Type     | Priority 1 | Priority 2              |
| :-----------: | :--------: | :---------------------: |
| spine         | 20         | (`node_id` modulus 256) |
| l3leaf        | 30         | (`node_id` modulus 256) |
| anything else | 127        | (`node_id` modulus 256) |

For leaf switches connecting to a PTP GrandMaster we recommend to manually set PTP priority 1 lower than the other leaf switches, for example to "10" as shown below:

```yml
nodes:
  ptp-leaf1:
    ptp:
      enabled: true
      priority1: 10
```

Alternatively the default `node_type_keys` can be overridden to add a `ptp_leaf` or similar node type with `default_ptp_priority1: 10`.

#### Manually setting PTP priorities

The automatic PTP priorities can be manually overriden if required, for example for blue-leaf1:

```yml
nodes:
  blue-leaf1:
    ptp:
      enabled: true
      priority1: < 0-255 | default -> automatically set based on node_type >
      priority2: < 0-255 | default -> (node_id modulus 256) >
```

### PTP Clock Identity

#### Setting PTP Clock Identity automatically

By default PTP clock identity is generated and set automatically.

```yml
auto_clock_identity = < true | false | default -> true >
clock_identity = < (clock_identity_prefix = 00:1C:73 (default)) + (PTP priority 1 as HEX) + ":00:" + (PTP priority 2 as HEX) >
```

#### PTP Clock Identity prefix

By default the 3-byte prefix is `00:1C:73`, but this can be overridden if `auto_clock_identity` is set to `true` (which is the default).
> **Please note:** Remember to use double-quotes around the value, as otherwise it will be not be rendered correctly.

For example for all spine nodes:

```yml
spine:
  defaults:
    ptp:
      enabled: true
      clock_identity_prefix: "01:02:03"
```

#### Using Arista EOS default PTP clock identity

If you prefer to have PTP clock identity be the system MAC-address of the switch, which is the default EOS behaviour, simply disable the automatic PTP clock identity as shown here:

```yml
spine:
  defaults:
    ptp:
      enabled: true
      auto_clock_identity: false
```

#### Setting PTP Clock Identity manually

PTP Clock identity can be set manually per node_group or node, for example for blue-spine1:
> **Please note:** Remember to use double-quotes around the value, as otherwise it will be not be rendered correctly.

```yml
blue-spine1:
  defaults:
    ptp:
      enabled: true
      clock_identity: "01:02:03:04:05:06"
```

#### Enable PTP unicast forwarding

With this feature enabled, multicast PTP packets will continue to be sent to the control plane, but unicast PTP packets will be hardware forwarded through the data plane.
This feature enables the use of protocols such as Meinbergs NetSync to monitor downstream PTP slaves in the network without having the PTP packets dropped by Arista Switches acting as boundary clocks.

```yml
forward_unicast: < true | false | default -> false >
```

### PTP Source IP address

By default in EOS, PTP packets are sourced with an IP address from the routed port or from the relevant SVI, which is the recommended behaviour. This can be set manually if required, for example, to a value of `10.1.2.3`:

```yml
spine:
  defaults:
    ptp:
      enabled: true
      source_ip: 10.1.2.3
```

### PTP Time-To-Live (TTL)

By default in EOS, PTP packets are sent with a TTL of 1. This is perfectly acceptable when using the recommended topology with routed interfaces or VLANs with a local SVI, but can be overridden and set manually, for example to a value of `64`:

```yml
spine:
  defaults:
    ptp:
      enabled: true
      ttl: 64
```

### PTP Monitor Threshold configuration

PTP monitor thresholds is enabled by default, when PTP is enabled. Default parameters are provided.

This will result in the following configuration:

```shell
ptp monitor threshold offset-from-master 250
ptp monitor threshold mean-path-delay 1500
ptp monitor sequence-id
ptp monitor threshold missing-message announce 3 sequence-ids
ptp monitor threshold missing-message delay-resp 3 sequence-ids
ptp monitor threshold missing-message follow-up 3 sequence-ids
ptp monitor threshold missing-message sync 3 sequence-ids
```

All parameters can be overridden if required:

```yml
ptp:
  enabled: true
  monitor:
    enabled: < false | true | default -> true >
    threshold:
      offset_from_master: < 0-1000000000 | default -> 250 >
      mean_path_delay: < 0-1000000000 | default -> 1500 >
      drop:
        offset_from_master: < 0-1000000000 >
        mean_path_delay: < 0-1000000000 >
    missing_message:
      intervals:
        announce: < 2-255 >
        follow_up: < 2-255 >
        sync: < 2-255 >
      sequence_ids:
        enabled: < false | true | default -> true >
        announce: < 2-255 | default -> 3 >
        delay_resp: < 2-255 | default -> 3 >
        follow_up: < 2-255 | default -> 3 >
        sync: < 2-255 | default -> 3 >
```

### Setting PTP DSCP values

You can manually set the global DSCP values used for PTP messages if this is required:

```yml
ptp:
  enabled: true
  dscp:
    general_messages: 46
    event_messages: 46
```

## PTP Settings for connected endpoints

By default all interfaces with connected endpoints do not have PTP enabled. It must be manually enabled.
These are the default settings:

- The global PTP profile parameters will be applied to all connected endpoints where ptp is manually enabled.
- `ptp role master` is set to ensure control over the PTP topology.

```yml
servers:
  <server-name>:
    adapters:
    - type: <type>
      server_ports: [ <server-ports> ]
      switch_ports: [ <switchports> ]
      switches: [ <switches> ]
      ptp:
        enable: < false | true | default -> false >
        endpoint_role: < follower | bmca | default -> follower >
        profile: < aes67 | smpte2059-2 | aes67-r16-2016 | default -> aes67-r16-2016 >
```

### Examples of connected endpoints

If you want to configure a routed interface with no IP address, for example to connect to a PTP Grandmaster, you can do as shown below. Please note in this example, the default PTP role has been overridden, making the interface Ethernet5 on blue-spine1 towards the Blue PTP Grandmaster run BMCA. Depending on the topology, this interface will end up in PTP mode Slave or Passive.

```yml
servers:
  Blue-Grandmaster:
    adapters:
    - type: server
      server_ports: [ eth1 ]
      switch_ports: [ Ethernet5 ]
      switches: [ blue-spine1 ]
      structured_config:
        type: routed
      ptp:
        enable: true
        endpoint_role: bmca
```

If you want to use a different PTP profile on an interface towards a connected endpoint, it can be set manually:

```yml
servers:
  Endpoint-with-specific-PTP-profile:
    adapters:
    - type: server
      server_ports: [ eth3 ]
      switch_ports: [ Ethernet7 ]
      switches: [ blue-leaf1 ]
      ptp:
        enable: true
        profile: smpte2059-2
```

## Interfaces only used for PTP traffic

In a Media and Entertainment Layer 3 Leaf Spine network topology with redundant connectivity to either a single or dual PTP GrandMasters, you might require network connectivity between parts of the network ***only*** for PTP traffic.

Such switch-to-switch links must be defined on a fabric level, for example using two links between blue-leaf1 and blue-leaf2 as shown here:

```yml
core_interfaces:
  p2p_links:
      # Unique id
    - id: 1
      # Nodes where this link should be configured
      nodes: [ blue-leaf1, blue-leaf2 ]
      # Interfaces where this link should be configured | Required unless using port-channels
      interfaces: [ Ethernet10, Ethernet10 ]
      # Enable PTP
      ptp_enable: true
      # Do not add this interface to underlay routing protocol
      include_in_underlay_protocol: false
    - id: 2
      nodes: [ blue-leaf1, blue-leaf2 ]
      interfaces: [ Ethernet11, Ethernet11 ]
      ptp_enable: true
      include_in_underlay_protocol: false
```

This will result in the two interfaces being configured as routed interfaces without an IP address, with PTP enabled using the same PTP profile as defined globally:

```shell
! blue-leaf1
!
interface Ethernet10
   description P2P_LINK_TO_blue-leaf2_Ethernet10
   no shutdown
   mtu 1500
   no switchport
   ptp enable
   ptp sync-message interval -3
   ptp announce interval 0
   ptp transport ipv4
   ptp announce timeout 3
   ptp delay-req interval -3
!
interface Ethernet11
   description P2P_LINK_TO_blue-leaf2_Ethernet11
   no shutdown
   mtu 1500
   no switchport
   ptp enable
   ptp sync-message interval -3
   ptp announce interval 0
   ptp transport ipv4
   ptp announce timeout 3
   ptp delay-req interval -3
```

## PTP Options ***not*** included

- `ptp delay-mechanism` parameters are not included.
- `vlan` settings are not included.

However, these settings ***can*** be set using custom cli configuration if needed.