..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Support for new OOB driver - Redfish
==========================================

The blueprint introduces Redfish as new OOB driver to Drydock.

Problem description
===================

Drydock uses Phygmi as OOB driver to manage/configure servers. Phygmi is
basically a python implementation for IPMI functionality. Currently phygmi
supports few commands related to power on/off, boot, events and Lenovo OEM
functions. To introduce a new IPMI command in pyghmi requires to know the low
level details of the functionality like Network Function, Command and the
data bits to be sent. This makes the system complex to operate at very low
level details.

DMTF's have proposed a new Standard Platform management API Redfish using a
data model representation inside of hypermedia RESTful interface. Vendors like
Dell, HP supports Redfish and API are exposed to perform the configurations.
Being a REST and model based standard makes it easy for the client side
implementations.

Proposed change
===============

Proposal is to add new OOB driver Redfish for Drydock. Already drydock provides
users configurable parameter to choose OOB driver of their choice.

This blueprint covers the following
- Add Redfish driver as OOB driver and implement existing OOB actions for Redfish
- Support to set/update BIOS attributes
- Support to update firmware
- Support to configure RAID

Add Redfish driver as OOB driver
--------------------------------

* Implement Redfish driver with oob_types_supported as 'redfish' to execute actions.

* Implement the following existing OOB actions for Redfish
    - ValidateOobServices
    - ConfigNodePxe
    - SetNodeBoot
    - PowerOffNode
    - PowerOnNode
    - PowerCycleNode
    - InterrogateOob

* Implement an abstract Redfish client which can internally call Redfish client libraries
  of Dell and HP.
  The above mentioned actions requires the following management functionalities
  - GET/SET power [on/off]
  - GET/SET bootdev [pxe]

Support to set/update BIOS attributes
-------------------------------------

This functionality is to provide the deployment engineer flexibility to update
BIOS attributes using site yaml files.

* Implement new function get_bios_attributes, set_bios_attributes in abstract Redfish
  client, internally calling corresponding HP, dell Redfish libraries.

  URL to configure Bios attributes using Redfish
  https://<OOB IP>/redfish/v1/Systems/<system_name>/Bios
  
* Create new action to BIOS configuration in Prepare Node task.

TODO:
Sample site yaml to update BIOS attributes

Support to update Firmware
--------------------------

This functionality is to provide the deployment engineer flexibility to update the
Firmware using site yaml files.

* Implement new function update_firmware in abstract Redfish client, internally calling
  corresponding HP, dell Redfish libraries.

* Trigger the functionality under the action created in the above step.

TODO:
Sample site yaml to update Firmware

Support to configure RAID
-------------------------

This functionality is to provide the deployment engineer flexibility to configure RAID
using site yaml files.

* Introduce 2 functions in abstract Redfish client, internally calling
  corresponding HP, dell Redfish libraries
  - To verify raid configuration is as specified in site yaml
  - To update the raid configuration as per site yaml specification.

* Trigger the functionality under the action 'Setup commissioning configuration' in task
  'PrepareNode'

TODO:
Sample site yaml with RAID configuration

Alternatives
------------

Existing Phygmi driver can be updated to support BIOS attributes, update firmware
and RAID configuration.

REST API impact
---------------

None

Security impact
---------------

None

Other end user impact
---------------------

Site yaml configuration updates as specified in Proposed change

Performance Impact
------------------

None

Other deployer impact
---------------------

Following configuration can be modified to invoke Redfish driver.

[plugins]

# List of module path strings of OOB drivers to enable (list value)
#oob_driver = drydock_provisioner.drivers.oob.redfish_driver.RedfishDriver

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Hemanth Nakkina/ hemanth-n

Other contributors:
  PradeepKumar KS
  Gurpreet Singh


Dependencies
============

Following external libraries will be used for Redfish client
https://github.com/dell/iDRAC-Redfish-Scripting/
https://github.com/HewlettPackard/python-ilorest-library

Testing
=======

* Greenfield deployment with HP and Dell servers.
  Ensure Dell servers that support iDRAC8/9 are verified.

* Brownfield deployment on existing lab switching OOB driver

Documentation Impact
====================

Update drydock documentation for configuration changes introduced
by this feature.


References
==========

* https://www.dmtf.org/standards/redfish

* https://redfish.dmtf.org/redfish/mockups/v1

* https://github.com/openstack/pyghmi
