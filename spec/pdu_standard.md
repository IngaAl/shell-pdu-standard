# PDU Shell Standard

#### Version 1.0.0


## Introduction
The PDU Shell Standard is a project used to define a standard for all PDU Shells that integrate with CloudShell. The standard defines the Shell’s data model, commands and a set of guidelines that should be followed in the PDU Shells development and customization.



## Revision History

Version | Date | Notes
--- | --- | ---
1.0.0 | 2016-02-15 | First release of the PDU Shell Standard


## Definitions
### Granularity
A PDU Shell should support all PDU devices of the same Vendor. For example, a correct shell granularity will be “Raritan PDU” and not “Raritan PDU PX2”.

### Specific Models Certification
Each release Shell should have a list of certified models. Model certification can be done only by Quali’s engineering, and validates that all the Shell’s capabilities are working for a specific model.
The Shell should also work for non-certified models, and in case some gaps are found a new version of the Shell will be released with the gaps addressed and the model certified.

### Generic Data Model
All PDU Shells share the same generic data model, except the model of the root level which is different per each Shell. The data model shouldn’t be modified.
The attributes associated with those generic models are also shared by all networking Shells and their values are populated by the driver. It is allowed to add custom attributes only to the root level model, and it isn’t allowed to remove attributes from any level.

### Versioning
The shell version follows Semantic Versioning Guidelines (see details in http://semver.org). In short, the version structure is Major.Minor.Patch, for example “1.0.2”. A Path version is promoted when making backward-compatibility bug fixes, a Minor version is promoted when adding functionality in a backwards-compatible manner and a  Major version is promoted when making a backwards incompatible changes.

### Dependencies
In case the  shell is written in Python and has dependencies to Python packages (that follow Semantic Versioning Guidelines) the dependency should be to a range of Patch versions, for example to “cloudshell-pdu-raritan 2.1.X”.
The dependency to cloudshell-automation-api will be to the latest Patch version (cloudshell-automation-api package version is of the format “CloudShell_Version.X”, for example 7.0.X”).



## Data Model
### Families & Models
The PDU Shell Standard supports PDUs and the power managed device associated that is connected to the PDU.


#### PDU Data Model
- PDU
  - Power Socket

#### Power Managed Device Data Model
- Power Managed Device
  - Power Port


##### Example:

- Family: PDU, Model: Raritan PDU
  - Family: Power Socket, Model: Generic Power Socket


  - Family: Power Managed Device, Model: Generic Power Managed Device
    - Family: Power Port, Model: Generic Power Port


In order for the PDU shell to be used correctly within CloudShell both the PDU and devices connected to the PDU should be modeled in CloudShell, and the PDU sockets should be defined as “physically connected” to the devices power ports. Once this configuration is in place the devices connected to the PDU will get the “Power On”, “Power Off” and “Power Cycle” commands and those commands will be applicable on the power ports. The power ports can be defined on any type of device (for example, a networking device). The PDU shell comes with an OOB power managed device model that can be used to model in CloudShell a device connected to the PDU via a power port.



#### Family Rules
The following CloudShell family rules are enabled per family in the data model:

Family | Rules
--- | ---
PDU | Searchable
Power Socket | Searchable, Connectable, Locked By Default
Power Managed Device | Searchable
Power Port | 	Searchable, Connectable, Locked By Default



#### Resource Name and Address
Family | Model | Resource Name | Resource Address
--- | --- | --- | ---
PDU | [Vendor] PDU | (user defined) | Searchable
Power Socket | Generic Power Socket | (name in device) | (name in device)
Power Managed Device | Generic Power Managed Device | Searchable | Searchable
Power Port | Generic Power Port | PP[Container ID][ID] | PP[Container ID][ID]

Note: The [ID] for each sub-resource is taken from the device itself (corresponds to the names defined in the device).


### Attributes
#### Guidelines
- Attributes which aren’t relevant to a devices won’t be populated by the driver.
- All attributes which aren't user-input are "read only"
- The attribute rules are as follows - all attributes which are user input should have the rule "Configuration" enabled, all attributes which aren't user input should have the rules "Settings" and "Available For Abstract Resources" enabled.
- It is possible to customize the attribute rules selection after importing the Shell to CloudShell.
- Attributes shouldn’t be removed.
- Custom attributes should be added only to the root level model.
- All attributes are of type String unless mentioned otherwise

##### [Vendor] PDU

Attribute Name | Details | User input?
--- | --- | ---
User | User with administrative privileges | Yes
Password | Attribute of type Password | Yes
Firmware Version | | No
Vendor | | No
Model | | No
SNMP Read Community | | Yes
SNMP Write Community | | Yes
SNMP V3 User | | Yes
SNMP V3 Password | | Yes
SNMP V3 Private Key | | Yes
SNMP Version | Possible values – v1, v2c, v3 | Yes
CLI Connection Type | Attribute of type Lookup. Possible values – Auto, Console, SSH, Telnet, TCP | Yes
Sessions Concurrency Limit | Numeric, default is 1 (no concurrency) | Yes


#####  Generic Power Socket

No Attributes


#####  Generic Power Managed Device

No Attributes


#####  Generic Power Port

Attribute Name | Details | User input?
--- | --- | ---
Model | | No
Serial Number || No
Version | | No
Port Description | | No



## Commands
Below is a list of all the commands that will be part of the standard Shell, their names and interfaces. Each PDU Shell that will be released by Quali’s engineering will include implementation for all those commands.
When creating a new shell according to the standard it is OK not to implement all commands and/or implement additional command, but a command with a functionality that fits one of the predefined list commands should be implemented according to the standard.
Command outputs: On failure an exception containing the error will be thrown and the command will be shown as failed. A failure is defined as any scenario in which the command didn’t complete its expected behavior, regardless if the issue originates from the command’s input, device or the command infrastructure itself. On success the command will just return as passed with no output. The “Autoload” command has a special output on success that CloudShell reads when building the resource hierarchy. The “Save” command will return an output on success with the file name (exact syntax below).



### Autoload
Queries the devices and loads the structure and attribute values into CloudShell.
  - SNMP Based



### Power On
Starts the power for the selected socket.
  - This command should be tagged as “connected command”


### Power Off
Stops the power for the selected socket.
- This command should be tagged as “connected command”


### Power Cycle
Stops and then starts the power for the selected socket.   - CLI based
- This command should be tagged as “connected command”
   - Inputs:
     - Delay (string input)
     Optional. If kept empty the default from the The value here should be a positive integer and defines the wait time between the power off and power on. If left empty the default from the device will be used.