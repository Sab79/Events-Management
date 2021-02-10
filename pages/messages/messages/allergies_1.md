---
title: Allergies and Adverse Reactions v1
keywords:  messaging, bundles
tags: [fhir,messaging]
sidebar: overview_sidebar
permalink: allergies_1.html
summary: "Guidance and requirements for the Allergies and Adverse Reactions v1 event message"
---

## Event Message Content

The `Allergies and Adverse Reactions v1` event message represents a single allergy or adverse reaction relating to a patient, and any relevant supporting information.

All "Allergies and Adverse Reactions v1" event messages that are published to the NEMS **MUST** be created inline with guidance and requirements specified on this page and on the [Generic Event Message Requirements](explore_generic_event_requirements.html) page.


## Bundle structure

- The event message will contain a mandatory `MessageHeader` resource as the first element within the event message bundle as per FHIR messaging requirements.
- The MessageHeader resource references a `List` resource as the focus of the event message.
- The `List` resource SHOULD reference a "DocumentReference" resource which points to where the latest allergies and adverse reactions information can be retrieved
- The `List` resource MAY reference an `AllergyIntolerance` resource contained in the message. This resource must represent the allergy or adverse reaction that is the focus of the event message.


The diagram below shows the referencing between FHIR resources within the event message bundle:

<div style="text-align:center; margin-bottom:20px" >
	<a href="images/messages/allergy_intolerance_1.png" target="_blank"><img src="images/messages/allergy_intolerance_1.png"></a>
</div>


## Event Life Cycle ##

The `MessageHeader` resource contains the `messageEventType` extension which represents the action the event message represents at a resource level, for example the `Allergy` being shared is new, the `Allergy` or supporting resources have been updated or the `Allergy` has been deleted. The `messageEventType` extension shall contain a values as per the table below:

| Value | Description |
| --- | --- |
| new |  The `new` value must be used when the Allergy or Adverse Reaction is being shared for the first time. |
| update | The `update` value must be used when the Allergy or Adverse Reaction and supporting resources have previously been shared, but have been updated and the updated resources are being shared. |
| delete | The `delete` value must be used when the Allergy or Adverse Reaction record has been deleted and the record no longer exists. |


### Identifying Information

To allow subscribers to identify information between `new`, `update` and `delete` event messages the publisher must:

- publish a complete event message for all event types
- included identifiers within resources which are maintained between different event messages


### Message Sequencing

As allergies and adverse reactions shared using the allergies and adverse reactions event message may change and therefore `new`, `update` and `delete` types of the event are supported. To allow a consumer to perform message sequencing, the event MUST include the `meta.lastUpdated` element within the `MessageHeader` resource allowing the consumer to identify the latest and most up to date information.


## Onward Delivery ##

The delivery of the `Allergies and Adverse Reactions v1` event messages to subscribers via MESH will use the following `WorkflowID` within the MESH control file. This `WorkflowID` will need to be added to the receiving MESH mailbox configuration before event messages can be received.

| MESH WorkflowID | `ALLERGIES_ADVERSE_REACTIONS_1` |


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
| event | 1..1 | Fixed Value: allergies-and-adverse-reactions-1 (Allergies and Adverse Reactions) |
| focus | 1..1 | This will reference the `List` resource which contains reference to the allergy or adverse reaction information this event relates to. |


### [List](http://hl7.org/fhir/STU3/list.html)

The `List` resource **MUST** conform to the `List` base FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 1..1 |

|Element|Cardinality|Additional Guidance|
|-------|-----------|-------------------|
| entry.item.reference(DocumentReference) | 0..1 | Reference to the DocumentReference resource representing retrieval endpoint |
| entry.item.extension `(valueIdentifier).valueIdentifier` | 0..1 | This extension may be included on the `entry.item` for the entry related to the NRL DocumentReference.<br/><br/>The business identifier for the allergy or adverse reaction, which is the trigger/focus for this event, **MAY** be included within this extension to reference the specific allergy or adverse reaction which will be returned within the list of AllergyIntolerance resources retrieved from the endpoint. |
| entry.item.reference(AllergyIntolerance) | 0..1 | Reference to AllergyIntolerance resource if included in the bundle |


### [DocumentReference](https://fhir.nhs.uk/STU3/StructureDefinition/NRL-DocumentReference-1)

The event message SHOULD contain a "NRL-DocumentReference-1" Resource which is a pointer to the endpoint URLs exposed by the publisher, where allergy and adverse reaction information can be retrieved by the subscriber.

| Resource Cardinality | 0..1 |

The DocumentReference resource **MUST**:

- conform to the requirements in the [National Record Locator (NRL)](https://nrl-data-format-draft.netlify.app/) specification 
- be a [pointer](https://nrl-data-format-draft.netlify.app/pointer_data_model_overview.html) of the `information type` ["Allergies and adverse reactions"](https://nrl-data-format-draft.netlify.app/supported_pointer_types.html)
- point to allergies and adverse reactions information and support at least the [CareConnect Allergy Intolerance FHIR STU3](https://nrl-data-format-draft.netlify.app/retrieval_careconnect_allergies_fhir_stu3.html) retrieval format and interaction
- be successfully created on the NRL, using the [create interaction](https://nrl-data-format-draft.netlify.app/api_interaction_create.html) before publishing this event message


### [CareConnect-Organization-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Organization-1)

All Organization resources included in the bundle SHALL conform to the [CareConnect-Organization-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Organization-1) constrained FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 1..* |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| identifier | 1..* | The organization ODS code identifier SHALL be included within the `odsOrganizationCode` identifier slice. |
| name | 1..1 | A human readable name for the organization SHALL be included in the organization resource. |


## [CareConnect-AllergyIntolerance-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-AllergyIntolerance-1) Resource and Supporting Resources

| Resource Cardinality | 0..* |

The contained `AllergyIntolerance` resource and any supporting resources MUST be populated and formatted in the bundle in the same way they would be if the consumer had retrieved the information using the [CareConnect Allergy Intolerance FHIR STU3 - Read](https://nhsconnect.github.io/CareConnectAPI/api_clinical_allergyintolerance.html#1-read) intertaction.


## Examples


<div class="tabPanel">

	<div class="tabHeadings">
		<span class="tabHeading" id="newPoint">New (Pointer Only)</span>
		<span class="tabHeading" id="new">New</span>
		<span class="tabHeading" id="update">Update</span>
		<span class="tabHeading" id="delete">Delete</span>
	</div>
	
	<div class="tabBodies">
	
		<div class="tabBody" id="newPointBody" markdown="span">
			```{% include_relative examples/allergy-intolerance-1-new-pointer-only.xml %}```
		</div>
		
		<div class="tabBody" id="newBody" markdown="span">
			```{% include_relative examples/allergy-intolerance-1-new.xml %}```
		</div>
		
		<div class="tabBody" id="updateBody" markdown="span">
			```{% include_relative examples/allergy-intolerance-1-update.xml %}```
		</div>
		
		<div class="tabBody" id="deleteBody" markdown="span">
			```{% include_relative examples/allergy-intolerance-1-delete.xml %}```
		</div>
	
	</div>
</div>

