---
title: NEMS Lightweight Events
keywords:  messaging, bundles
tags: [fhir,messaging]
sidebar: overview_sidebar
permalink: lightweight_1.html
summary: "Lightweight events supported by the NEMS"
---

What?
Why?
How?

## Event Types

| Event Type | Description | Pointer Class | Pointer Type | Retrieval Format(s) |
| --- | --- | --- | --- | --- |
| demographic-change-1 | Notification of a change to a patients demographics | Record Heading | Demographics | [PDS_FHIR_API](https://digital.nhs.uk/developer/api-catalogue/personal-demographics-service-fhir) |
| vaccination-2 | A message which represents a vaccingaiton being given. | Record Heading | Vaccination | FHIR_STU3_Vaccination-1<br/>FHIR_STU3_Vaccination_List-1 |


## Document Reference Types and Retrieval

| Event Type | Pointer Class | Pointer Type | Retrieval Format(s) |
| --- | --- | --- | --- |
| demographic-change-1 | Record Heading | Demographics | [PDS_FHIR_API](https://digital.nhs.uk/developer/api-catalogue/personal-demographics-service-fhir) |
| vaccination-2 | Record Heading | Vaccination | FHIR_STU3_Vaccination-1<br/>FHIR_STU3_Vaccination_List-1 |




## FHIR Event Message Structure 
 
The following is the FHIR message structure for all NEMS lightweight event messages.

<div style="text-align:center; margin-bottom:20px" >
	<a href="images/messages/lightweight_1.png" target="_blank"><img src="images/messages/lightweight_1.png"></a>
</div>

Describe what the components mean?


## Onward Delivery 

MESH will use the following generic WorkflowID within the MESH control file. This WorkflowID will need to be added to the receiving MESH mailbox configuration before event messages can be received. 

| MESH WorkflowID | NEMS_EVENT_1 |

{% include important.html content="This workflow ID is generic for multiple NEMS event messages. Subscribers will need to identify the type of event by looking at the '`event`' element within the '`MessageHeader`' resource." %}



## Resource Population Guidance 


### [Bundle](http://hl7.org/fhir/STU3/StructureDefinition/Bundle)

The Bundle resource is the container for the event message and SHALL conform to the [Bundle](http://hl7.org/fhir/STU3/StructureDefinition/Bundle) FHIR profile.

| Resource Cardinality | 1..1 |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| type | 1..1 | Fixed value: `message` |


### [Event-MessageHeader-1](https://fhir.nhs.uk/STU3/StructureDefinition/Event-MessageHeader-1)

The MessageHeader resource included as part of the event message SHALL conform to the [Event-MessageHeader-1](https://fhir.nhs.uk/STU3/StructureDefinition/Event-MessageHeader-1) constrained FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 1..1 |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| extension(routingDemographics) | 1..1 | The extension MUST contain the details of the patient who is the focus of this event message. |
| extension(routingDemographics).extension(nhsNumber) | 1..1 | The extension MUST contain the patient’s NHS Number identifier and is used by the NEMS for routing event messages to subscribers. |
| extension(routingDemographics).extension(name) | 1..1 | The extension MUST contain the human name element containing the patient’s official names as recognised by PDS,  and match the NHS number in the routingDemographics extension. |
| extension(routingDemographics).extension(birthDateTime) | 1..1 | The extension MUST contain the patient’s Date Of Birth which matches the NHS number in the routingDemographics extension. |
| extension(messageEventType) | 1..1 | Fixed value: `new` |
| event | 1..1 | Fixed Value: pds-record-changed-1 (PDS Record Changed) |
| focus | 1..1 | This will reference the focus “Patient” resource. |

**Note:** Where the event message relates to a PDS record being superseded by another, the patient details included in the event will be for the record which is being superceeded rather than the record which is superseding it.


### [DocumentReference](https://www.hl7.org/fhir/stu3/documentreference.html)

| Resource Cardinality | 1..1 |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| patient | 1..1 | abc |
| class | 1..1 | abc |
| type | 1..1 | abc |
| retrieval format |


## Examples ##

```xml
{% include_relative examples/PDS-Record-Change.xml %}
```
