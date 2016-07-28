# OpenStack HEAT for Netfloc

This is an OpenStack HEAT plugin for the [SDK for SDN - Netfloc](http://icclab.github.io/netfloc/ "Netfloc's github page") in order to use the resources of its service funciton chaining (SFC) library. It contains ```netfloc.py``` to be copied in the heat plugins library and two yaml templates: ```create_sfc.yaml``` to create all the required OpenStack resources for SFC (VNF VMs, networks, and ports) and two service chain examples. The example of service chain demo deployed in OpenStack testbed evnironment is included in the [T-Nova project repository] (https://github.com/T-NOVA/netfloc-demo).

## Prepare the plugin

* Create a heat plugin directory under /usr/lib and copy inside the file ***netfloc.py*** (or place it alternatively under existing user-defined library):

```
mkdir /usr/lib/heat
cp src/netfloc.py /usr/lib/heat/
```

* Uncomment the ```plugin_dirs``` line in ```/etc/heat/heat.conf``` and include the path to the library.

* For the chain to work, the port ID has to be returned by the ```handle_create()``` method of the native heat implementation for the [port.py resource](https://github.com/openstack/heat/blob/master/heat/engine/resources/openstack/neutron/port.py#L406). To do so include the following line at the end of this method:

```
return port['id']  
```
This will enable to return the Port ID as a resource in the yaml template for chain creation.
The heat port.py resource library is located under: 

``` 
/usr/lib/python2.7/site-packages/heat/engine/resources/openstack/neutron/port.py
```

* Restart the heat engine service:

```
service openstack-heat-engine restart
```

* Run ```heat resource-type-list``` and verify that the following two Netfloc resources show up:

```
Netfloc::Service::Chain
```

## Test the service chain

* Before creating a chain, Netfloc must be running in a clean OpenStack environment and a public network has to be manualy created with assigning its ID as a parameter in the ```create_sfc.yaml``` file. 

* Once the setup is done and the chain is created:
```heat stack-create -f create_sfc.yaml [name_of_stack]``` 
the chain ID is listed in the Outputs section of the Stack Overview. Verify that the traffic steering is correct (ex. tcpdump) inside the VNF and on the endpoints. 

* To delete the chain run: 
```heat stack-delete [name_of_stack]```
specifying in the template the ID of the chian to be deleted.

* This is the output from the heat-engine log file:

```
tail -f /var/log/heat/heat-engine.log | grep --line-buffered netfloc:
18589 DEBUG urllib3.connectionpool [-] "POST /restconf/operations/netfloc:create-service-chain HTTP/1.1" 200 None _make_request /usr/lib/python2.7/dist-packages/urllib3/connectionpool.py:430
18589 DEBUG urllib3.connectionpool [-] "POST /restconf/operations/netfloc:delete-service-chain HTTP/1.1" 200 0 _make_request /usr/lib/python2.7/dist-packages/urllib3/connectionpool.py:430
```
* This is the output from the Netfloc log file:

```
root@odl:~/netfloc_current# tail -f karaf/target/assembly/data/log/karaf.log | grep --line-buffered chainID
2016-05-26 10:29:44,826 | INFO  | tp1443878269-438 | ServiceChain                     | 269 - ch.icclab.netfloc.impl - 1.0.0.SNAPSHOT | ServiceChain chainID: 1
2016-05-26 10:29:44,826 | INFO  | tp1443878269-438 | NetflocServiceImpl               | 269 - ch.icclab.netfloc.impl - 1.0.0.SNAPSHOT | chainID: 1
```

A simple demo example of service chain setup is shown in the following ICCLab [blog post](https://blog.zhaw.ch/icclab/service-function-chaining-using-the-sdk4sdn/).
Following the basic template, the complexity of the chain can be modified by defining additional VNF resources in the yaml template. 