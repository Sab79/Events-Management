---
title: Vaccinations v3
keywords:  messaging, bundles
tags: [fhir,messaging]
sidebar: overview_sidebar
permalink: vaccinations_3.html
summary: "Guidance and requirements for the Vaccinations v3 event message"
---

## Event Message Content

The `Vaccinations v3` event message represents a vaccination given or not-given to a patient.

All "Vaccinations v3" event messages that are published to the NEMS **MUST** be created inline with guidance and requirements specified on this page and on the [Generic Event Message Requirements](explore_generic_event_requirements.html) page.


## Bundle structure

- The event message will contain a `MessageHeader` resource as the first element within the event message bundle as per FHIR messaging requirements.
- The focus of the MessageHeader resource will be atleast one `DocumentReference` resource.
- The message MUST contain one `DocumentReference` resource which points to information about the administered vaccination this event relates to.
- The message may also contain additional `DocumentReference` resource which point to were a list of recorded vaccinations for the patient can be retrieved. If included this may be consumed by the Record Locator Service to enable consumers who did not recieve the event message to retrieve the patients medications at a later date.


The diagram below shows the referencing between FHIR resources within the event message bundle:

<div style="text-align:center; margin-bottom:20px" >
	<a href="images/messages/vaccinations_3.png" target="_blank"><img src="images/messages/vaccinations_3.png"></a>
</div>


## Event Life Cycle ##

The `MessageHeader` resource contains the `messageEventType` extension which represents the action the event message represents at a resource level, for example the `Vaccination` being shared is new, the `Vaccination` or supporting resources have been updated or the `Vaccination` has been deleted. The `messageEventType` extension shall contain a values as per the table below:

| Value | Description |
| --- | --- |
| new |  The `new` value must be used when the Vaccination is being shared for the first time. |
| update | The `update` value must be used when the Vaccination and supporting resources have previously been shared, but have been updated and the updated resources are being shared. |
| delete | The `delete` value must be used when the Vaccination record has been deleted and the record no longer exists. |

**Note:** If a DocumentReference pointing to a list of patient medicaitons is included, the Record Locator will store the pointer if the event type is `new` or `update`. The pointer stored on the Record Locator will not be deleted if the event type is `delete`. To remove a pointer from NRL the [Pointer Management](pointer_management_1.html) event must be used.


### Message Sequencing

As vaccinations shared using the vaccinations event message may change and therefore `new`, `update` and `delete` types of the event are supported. To allow a consumer to perform message sequencing, the event MUST include the `meta.lastUpdated` element within the `MessageHeader` resource allowing the consumer to identify the latest and most up to date information.


## Onward Delivery ##

The delivery of the `Vaccinations v3` event messages to subscribers via MESH will use the following `WorkflowID` within the MESH control file. This `WorkflowID` will need to be added to the receiving MESH mailbox configuration before event messages can be received.

| MESH WorkflowID | `NEMS_EVENT_1` |

{% include important.html content="This workflow ID is generic for multiple NEMS event messages. Subscribers will need to identify the type of event by looking at the '`event`' element within the '`MessageHeader`' resource." %}

## Resource Population Requirements and Guidance ##

The following requirements and resource population guidance must be followed in addition to the requirements and guidance outlined in the [Generic Requirements](explore_generic_event_requirements.html) page.


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
| meta.lastUpdated | 1..1 | The dateTime when the information was changed within the publishing system, for the use of event sequencing. |
| extension(messageEventType) | 1..1 | See the "Event Life Cycle" section above. |
| event | 1..1 | Fixed Value: vaccinations-3 (Vaccinations v3) |
| focus | 1..* | This will reference `DocumentReference` resource(s) which contains reference to the vaccination information. A pointer to the specific vaccinaiton this event relates too must be included and an additional DocumentReference pointing to a list of vaccinations for the patient may be included. |


### [DocumentReference](https://fhir.nhs.uk/STU3/StructureDefinition/NRL-DocumentReference-1)

The event message MUST contain atleast one "NRL-DocumentReference-1" Resource which points to the endpoint URLs exposed by the publisher, where the vaccination information for this event can be retrieved by the subscriber. The provider MAY also include an additional DocumentReference pointing to where a list of all the patients vaccinations can be retrieved.

| Resource Cardinality | 1..* |


**MANDATORY - Pointer to event specific vaccination**

The DocumentReference resource **MUST**:

- conform to the requirements in the [National Record Locator (NRL)](https://nrl-data-format-draft.netlify.app/) specification 
- be a [pointer](https://nrl-data-format-draft.netlify.app/pointer_data_model_overview.html) of the `information type` ["Immunisation"](https://nrl-data-format-draft.netlify.app/supported_pointer_types.html)
- point to a single Immunizaiton resource. Calling the endpoint must return a Immunizaiton resource and any included links must be resolvable.


**OPTIONAL - Pointer to list of the patient's vaccinations**

The DocumentReference resource **MUST**:

- conform to the requirements in the [National Record Locator (NRL)](https://nrl-data-format-draft.netlify.app/) specification 
- be a [pointer](https://nrl-data-format-draft.netlify.app/pointer_data_model_overview.html) of the `information type` ["Immunisation List"](https://nrl-data-format-draft.netlify.app/supported_pointer_types.html)
- point to all the vaccination information held by the provider for the patient. Calling the endpoint must return a Bundle of Immunizaiton resources in the same way as would be returned by a FHIR search for Immunizations.


### [CareConnect-Organization-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Organization-1)

All Organization resources included in the bundle SHALL conform to the [CareConnect-Organization-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Organization-1) constrained FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 1..* |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| identifier | 1..* | The organization ODS code identifier SHALL be included within the `odsOrganizationCode` identifier slice. |
| name | 1..1 | A human readable name for the organization SHALL be included in the organization resource. |


## Examples


<div class="tabPanel">

	<div class="tabHeadings">
		<span class="tabHeading" id="new-pointer">New</span>
		<span class="tabHeading" id="update">Update</span>
		<span class="tabHeading" id="delete">Delete</span>
	</div>
	
	<div class="tabBodies">
	
		<div class="tabBody" id="new-pointerBody" markdown="span">
			```{% include_relative examples/vaccinations-3-new.xml %}```
		</div>
		
		<div class="tabBody" id="updateBody" markdown="span">
			```{% include_relative examples/vaccinations-3-update.xml %}```
		</div>
		
		<div class="tabBody" id="deleteBody" markdown="span">
			```{% include_relative examples/vaccinations-3-delete.xml %}```
		</div>
		
	</div>
</div>

