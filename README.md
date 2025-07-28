## Table of contents

 - [Introduction](#intro)
 - [Installation](#install)
 - [Quick Start](#quickstart)
   - [Add Driver](#add_driver)
   - [Create Service Profile](#create_service_profile)
   - [Create/Delete Service Instance](create_delete_service_instance)
 - [Next Steps](#next_steps)
   - [Supported Drivers](#drivers)
   - [Example Service Profiles](#example_service_profiles)
   - [Users And Role Mappings](#users_and_role_mappings)
   - [Rest API](rest_api)

## <a name="intro"></a>Introduction
The SENSE system provides the mechanisms to enable multi-domain orchestration for a wide variety of network and other cyberinfrastructure resources in a highly customized manner.  These orchestrated services can be customized for individual domain science workflow systems and requirements.  SENSE services include:
- Layer 2 Point to Point Network Connections
- Layer 2 Multipoint Network Topologies
- Layer 3 Virtual Private Network (VPN) services.


## <a name="install"></a>Installation
SENSE-O consists of two major components: the Orchestrator server and the Keycloak SSO server that can be separately deployed. The SENSE-O deployment is container based and a guide for a kubernets deployment can be found [here](https://github.com/StackV/sense-helm/)

## <a name="quickstart"></a>Quick Start

- [ ] Assumes you have completed the installation.  
- [ ] Assumes you have created an <i>admin</i> user to access sense-o portal during the installation.

TODO Example driver and creating an instance. 

#### <a name="add_driver"></a>Add Driver
TODO

```
{
  "topologyUri": "urn:ogf:network:stack-fabric:2024:topology",
  "driverEjbPath": "java:module/FabricDriver",
  "publicKey": "replace_me",
  "credUrl": "https://cm.fabric-testbed.net",
  "idToken": "replace_me",
  "projectId": "replace_me"
}
```

#### <a name="create_service_profile"></a>Create Service Profile
TODO
```
{
  "data": {
    "type": "Multi-Path P2P VLAN",
    "connections": [
      {
        "bandwidth": {
          "qos_class": "bestEffort"
        },
        "name": "Connection 1",
        "terminals": [
          {
            "vlan_tag": "3912",
            "assign_ip": false,
            "uri": "urn:ogf:network:es.net:2013::newy32aoa-cr6:1_1_c26_1:mghpcc"
          },
          {
            "vlan_tag": "3912",
            "assign_ip": false,
            "uri": "urn:ogf:network:es.net:2013::newy32aoa-cr6:1_1_c13_1:+"
          }
        ]
      }
    ]
  },
  "service": "dnc"
}
```
#### <a name="create_delete_service_instance"></a>Create/Delete Service Instance
TODO

## <a name="next_steps"></a>Next Steps
TODO

#### <a name="drivers"></a>Supported Drivers
TODO

#### <a name="example_service_profiles"></a>Example Service Profiles
TODO

#### <a name="users_and_role_mappings"></a>Users And Role Mappings
TODO

#### <a name="rest_api"></a>Rest API
TODO
