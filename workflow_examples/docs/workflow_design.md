# Table of contents

 - [Description](#descr)
 - [Variables](#variables)
 - [Configs](#configs)
   - [Edit Template](#edit_template)
   - [Manifest](#manifest_template)
 - [Resources](#resources)
   - [Nodes](#nodes)
   - [Networks](#networks)
   - [Services](#services)
 - [Dependencies](#dependencies)

# <a name="descr"></a>Description
This readme file describes the sense workflow model. The model consist of the following high level classes:
- [ ] variable
- [ ] config
- [ ] resource

The `resource` class support two types:
- [ ] pool
- [ ] service

The `config` class support two types:
- [ ] edit_template
- [ ] manifest_template

# <a name="variables"></a>Variables
Variables have their own class named <i>variable</i>. A variable consists of a name and a value. Its declaration 
uses the <i>default</i> attribute signifying that it can be overriden at runtime using an external var-file. 
The variable <i>bandwidth<i> declared below can be referred to by the expression ```'{{ var.bandwidth }}'```.
 

```
variable:             # Class 
  - bandwidth:        # Label
      default: 2000   # Default value
 
```

A var-file consists of a set of key-value pairs and can be specified using the --var-file option. Each key-value pair is written as ```key: value```. The sample var-file below would override the value ```2000``` of the <i>bandwidth<i> declared above.

 ```
 bandwidth: 3000
 ```
 <b>NOTE</b>: The parsing process would halt if a variable is not bound to a value other than ```None```.

# <a name="configs"></a>Configs

A config consists of a <i>type</i>, a <i>label</i> and a dictionary specifying its attributes. The parsing process guarantees that the combination of the type and the label is unique. One can think of Configs as glorifed variables. 
We have two types `edit_template` and `manifest_template` referred to by <i>service</i> resources.

### <a name="edit_template"></a>Edit Template

In the example below, the sense service <i>fabric_l2vpn</i> refers to the <i>edit_template</i> fabric_l2vpn_edit_template.

```
config:
  - edit_template:
      - fabric_l2vpn_edit_template:
          data.connections[0].bandwidth.capacity: '{{ var.bandwidth }}'
resource:
  - service:
      - fabric_l2vpn:
          profile: FABRIC-L2-Net
          edit_template: '{{ edit_template.fabric_l2vpn_edit_template }}'
          count: '{{ var.count }}'

```

### <a name="manifest_template"></a>Manifest

In the example below, the sense service <i>fabric_l2vpn</i> refers to the <i>manifest_template</i> fabric_l2vpn_manifest_template.

```
config:
  - manifest_template:
      - fabric_l2vpn_manifest_template:
          terminals:
            - id: "?slice_id?"
              port: "?vlanport?"
              vlan: "?vlantag?"
              name: "?slice_name?"
              bw: ?bw?
              sparql: 'SELECT DISTINCT ?bw ?slice_name ?slice_id ?vlantag ?vlanport WHERE {
                    ?swsvc mrs:providesSubnet ?subnet. ?subnet nml:hasBidirectionalPort ?vlanport.
                    ?subnet nml:name ?slice_name.
                    ?subnet mrs:hasNetworkAddress ?na_slice_id. ?na_slice_id mrs:type "slice-id".
                    ?na_slice_id mrs:value ?slice_id. ?vlanport nml:hasLabel ?alabel. ?alabel nml:value ?vlantag.
                    ?terminal nml:hasService ?bw_svc. ?bw_svc mrs:type ?qos_type. ?bw_svc mrs:maximumCapacity ?bw. ?bw_svc mrs:unit ?bw_unit.
                    }'
resource:
  - service:
      - fabric_l2vpn:
          profile: FABRIC-L2-Net
          manifest_template: '{{ manifest_template.fabric_l2vpn_manifest_template }}'
```
# <a name="resources"></a>Resources
A resource consists of a <i>type</i>, a <i>label</i> and a dictionary. The parsing process guarantees that the combination of the type and the label is unique. Resources can refer to each other using the expression ```'{{ type.label }}'```. They can also refer to a resource's attribute using ```'{{ type.label.attribute_name }}'```. 

As of now we support the following types: <i>pool</i> and <i>service</i>. The <i>label</i> can be any string and is used to name of the resource. Resources are declared under their own class named <i>resource<i>. 
 
### <a name="nodes"></a>Nodes
A <i>node</i> <b>must</b> refer to a provider. Here it refers to the provider declared above. 
 
A <i>service</i> or a <i>network</i> would refer to this node using its type and label like so: ```'{{ node.fabric_node }}'```
 
```
resource:                                               # Class
  - node:                                               # Type must be one of node, network, or service
      - fabric_node:                                    # Label can be any string
            provider: '{{ fabric.fabric_provider }}'
            site: '{{ var.fabric_site }}'
            image: default_rocky_8   
            count: 1                               
```
### <a name="networks"></a>Networks
A <i>network</i> <b>must</b> refer to a provider. Here it refers to the provider declared above. 
 
A <i>node</i> or a <i>network</i> would refer to this network using its type and label like so: ```'{{ network.fabric_network }}'```
 
```
resource:                                               # Class
   - network:                                           # Type can be node or network
      - fabric_network:                                 # Label can be any string
            provider: '{{ fabric.fabric_provider }}'
            site: '{{ var.fabric_site }}'
            name: my_network
```
### <a name="services"></a>Services
A <i>service</i> <b>must</b> refer to a provider. Here it refers to a Janus provider for container management.
 
 
```
provider:
  - janus:
      - janus_provider:
          credential_file: ~/.fabfed/fabfed_credentials.yml
          profile: janus


resource:                                                          # Class
   - service:                                                      # Must be one of node, network, or service
      - dtn_service:                                               # Label can be any string
          - provider: '{{ janus.janus_provider }}'
            node: [ '{{ node.my_node0 }}', '{{ node.my_node1 }}' ] # List of nodes to apply the service to
```
 
# <a name="dependencies"></a>Dependencies
A resource can refer to other resources that it depends on. Line 14 in the example below, states that the <i>fabric_node</i> depends 
on the <i>fabric_network</i>. This is an `internal dependency` as both resources are handled by the same provider. The fabfed 
controller detects internal and external dependencies processes the resources in the correct during the apply phase and the destroy phase

Internally the controller handles external and internal dependencies differently. In the example below the controller would `add` the fabric node after `adding` the fabric network. And would `add` the fabric node after `creating` the chameleon network. 
```
resource:
  - network:
      - chi_network:
          provider: '{{ chi.chi_provider }}'
          layer3: "{{ layer3.my_layer }}"
      - fabric_network:
          provider: '{{ fabric.fabric_provider }}'
          layer3: "{{ layer3.my_layer }}"
          stitch_with:
            - network: '{{ network.chi_network }}' # External dependency 
      - fabric_node:
          provider: '{{ fabric.fabric_provider }}'
          network: '{{ network.fabric_network }}'  # Internal dependency
```