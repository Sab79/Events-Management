---
title: Vaccinations v2
keywords:  messaging, bundles
tags: [fhir,messaging]
sidebar: overview_sidebar
permalink: vaccinations_2.html
summary: "Guidance and requirements for the Vaccinations v2 event message"
---

## Event Message Content

The `Vaccinations v2` event message represents a single vaccination given or not-given to a patient and any relevant supporting information.

All "Vaccinations v2" event messages that are published to the NEMS **MUST** be created inline with guidance and requirements specified on this page and on the [Generic Event Message Requirements](explore_generic_event_requirements.html) page.


## Bundle structure

The event message will contain a mandatory `MessageHeader` resource as the first element within the event message bundle as per FHIR messaging requirements. The MessageHeader resource references a `List` resource as the focus of the event message. The `List` resource will contain at least one "DocumentReference" resource which will point to where the latest immunization information can be retrieved.

The List may also reference an `Immunization` resource contained in the message. This resource must represent the vaccination that was given or not given to the patient and is the focus of the event message.


The diagram below shows the referencing between FHIR resources within the event message bundle:

<div style="text-align:center; margin-bottom:20px" >
	<a href="images/messages/vaccinations_2.png" target="_blank"><img src="images/messages/vaccinations_2.png"></a>
</div>


## Event Life Cycle ##

The `MessageHeader` resource contains the `messageEventType` extension which represents the action the event message represents at a resource level, for example the `Vaccination` being shared is new, the `Vaccination` or supporting resources have been updated or the `Vaccination` has been deleted. The `messageEventType` extension shall contain a values as per the table below:

| Value | Description |
| --- | --- |
| new |  The `new` value must be used when the Vaccination is being shared for the first time. |
| update | The `update` value must be used when the Vaccination and supporting resources have previously been shared, but have been updated and the updated resources are being shared. |
| delete | The `delete` value must be used when the Vaccination record has been deleted and the record no longer exists. |


### Identifying Information

To allow subscribers to identify information between `new`, `update` and `delete` event messages the publisher must:

- publish a complete event message for all event types
- included identifiers within resources which are maintained between different event messages


### Message Sequencing

As vaccinations shared using the vaccinations event message may change and therefore `new`, `update` and `delete` types of the event are supported. To allow a consumer to perform message sequencing, the event MUST include the `meta.lastUpdated` element within the `MessageHeader` resource allowing the consumer to identify the latest and most up to date information.


## Onward Delivery ##

The delivery of the `Vaccinations v2` event messages to subscribers via MESH will use the following `WorkflowID` within the MESH control file. This `WorkflowID` will need to be added to the receiving MESH mailbox configuration before event messages can be received.

| MESH WorkflowID | `VACCINATIONS_2` |


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
| event | 1..1 | Fixed Value: vaccinations-2 (Vaccinations v2) |
| focus | 1..1 | This will reference the `List` resource which contains reference to the vaccination information this event relates to. |


### [List](http://hl7.org/fhir/STU3/list.html)

The `List` resource **MUST** conform to the `List` base FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 1..1 |

|Element|Cardinality|Additional Guidance|
|-------|-----------|-------------------|
| entry.item.reference(DocumentReference) | 1..1 | Reference to the DocumentReference resource representing retrieval endpoint |
| entry.item.extension `(valueIdentifier).valueIdentifier` | 0..1 | This extension may be included on the `entry.item` for the entry related to the NRL DocumentReference.<br/><br/>The business identifier for the immunization, which is the trigger/focus for this event, **MAY** be included within this extension to reference the specific immunization which will be returned within the list of immunizations retrieved from the endpoint. |
| entry.item.reference(Immunization) | 0..1 | Reference to contained Immunization resource if included |


### [DocumentReference](https://fhir.nhs.uk/STU3/StructureDefinition/NRL-DocumentReference-1)

The event message must contain a "NRL-DocumentReference-1" Resource which is a pointer to the endpoint URLs exposed by the publisher, where vaccination information can be retrieved by the subscriber.

The DocumentReference resource **MUST**:

- conform to the requirements in the [National Record Locator (NRL)](https://nrl-data-format-draft.netlify.app/) specification 
- be a [pointer](https://nrl-data-format-draft.netlify.app/pointer_data_model_overview.html) of the `information type` ["Immunisations"](https://nrl-data-format-draft.netlify.app/supported_pointer_types.html)
- point to immunization information and support at least the [Vaccination List - FHIR STU3](https://nrl-data-format-draft.netlify.app/retrieval_vaccinations_fhir_stu3.html) retrieval format and interaction
- be successfully created on the NRL, using the [create interaction](https://nrl-data-format-draft.netlify.app/api_interaction_create.html) before publishing this event message

| Resource Cardinality | 1..1 |



### [CareConnect-Organization-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Organization-1)

All Organization resources included in the bundle SHALL conform to the [CareConnect-Organization-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Organization-1) constrained FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 1..* |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| identifier | 1..* | The organization ODS code identifier SHALL be included within the `odsOrganizationCode` identifier slice. |
| name | 1..1 | A human readable name for the organization SHALL be included in the organization resource. |



## Optional Immunization Resource and Supporting Resources


### [CareConnect-Immunization-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Immunization-1)

If the contained Immunization resource is included as part of the event message, it SHALL conform to the [CareConnect-Immunization-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Immunization-1) constrained FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 0..1 |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| extension(vaccinationProcedure) | 1..1 |  A code from the SNOMED Clinical Terminology UK coding system, to record a vaccination procedure that is either given or not given.<br/><br/>If a vaccination is given, then an immunisation procedure concept or an immunisation situation (given) concept SHALL be used.<br/><br/>If a vaccination has not been given, then an immunisation situation (not done) concept SHALL be used.<br/><br/>Free text field should be used if no coded text available using `extension(vaccinationProcedure).valueCodeableConcept.text` |
| identifier | 1..1 | A publisher defined unique identifier for the vaccination which will be maintained across different event messages to allow subscribers to be identify the information within update or delete event messages. |
| notGiven | 1..1 | Value SHALL be `FALSE` when the vaccination was given or reported as given, `TRUE` when not given |
| vaccineCode | 1..1 | Immunization.vaccineCode SHALL use a value from  the [CareConnect-VaccineCode-1](https://fhir.hl7.org.uk/STU3/ValueSet/CareConnect-VaccineCode-1) value set |
| date | 1..1 | The date or partial date that the vaccination was administered, or reported vaccination was given in the opinion of the child and/or parent carer |
| primarySource | 1..1 | Value should be `FALSE` if the vaccination was reported, `TRUE` if the vaccination was administered |
| reportOrigin | 0..1 | If the vaccination was reported then the original source should be include |
| manufacturer | 0..1 | Where available this should be included |
| site | 0..1 | Where available this should be included |
| route | 0..1 | Where available this should be included |
| explanation.reasonNotGiven | 0..1 | If the vaccination was `notGiven` then the `reasonNotGiven` element SHALL be included |
| vaccinationProtocol.doseSequence | 0..1 | Where available the `doesSequence` should be include |


### [CareConnect-Patient-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Patient-1)

The patient resource included in the event message SHALL conform to the [CareConnect-Patient-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Patient-1) constrained FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 0..1 |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| identifier | 1..1 | Patient NHS Number identifier SHALL be included within the nhsNumber identifier slice |
| name (official) | 1..1 | Patients name as registered on PDS, included within the resource as the official name element slice |
| birthDate | 1..1 | The patients date of birth |


### [CareConnect-Practitioner-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Practitioner-1)

The Practitioner resources included as part of the event message SHALL conform to the [CareConnect-Practitioner-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Practitioner-1) constrained FHIR profile.

| Resource Cardinality | 0..* |


### [CareConnect-PractitionerRole-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-PractitionerRole-1)

The PractitionerRole resources included as part of the event message SHALL conform to the [CareConnect-PractitionerRole-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-PractitionerRole-1) constrained FHIR profile.

| Resource Cardinality | 0..* |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| organization | 1..1 | Reference to the Organization where the practitioner performs this role |
| practitioner | 1..1 | Reference to the Practitioner who this role relates to |
| code | 1..* | The practitioner role SHALL included a value from the [ProfessionalType-1](https://fhir.nhs.uk/STU3/ValueSet/ProfessionalType-1) value set. The PractitionerRole.code should also include the SDS Job Role name where available. |
| specialty | 1..1 | PractitionerRole.specialty SHALL use a value from [Specialty-1](https://fhir.nhs.uk/STU3/ValueSet/Specialty-1) value set |


### [CareConnect-Encounter-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Encounter-1)

The Encounter resource included as part of the event message SHALL conform to the [CareConnect-Encounter-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Encounter-1) constrained FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 0..1 |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| Encounter.type | 1..* | The encounter type SHOULD include a value from the [EncounterType-1](https://fhir.nhs.uk/STU3/ValueSet/EncounterType-1) value set. This value set is extensible so additional values and code systems may be added where required. |
| location | 0..1 | Reference to the location at which the encounter took place |
| subject | 1..1 | A reference to the patient resource representing the subject of this event |



### [CareConnect-HealthcareService-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-HealthcareService-1)

The HealthcareService resource included as part of the event message SHALL conform to the [CareConnect-HealthcareService-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-HealthcareService-1) constrained FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 0..1 |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| providedBy | 1..1 | Reference to the organization who provides the healthcare service |
| type | 1..1 | This will represent the type of service responsible for the event message. This will have a fixed value from the ValueSet [CareConnect-CareSettingType-1](https://fhir.hl7.org.uk/STU3/ValueSet/CareConnect-CareSettingType-1) |
| specialty | 1..1 | The specialty SHALL be a value from the [Specialty-1](https://fhir.nhs.uk/STU3/ValueSet/Specialty-1) value set |

### [CareConnect-Location-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Location-1)

The Location resources included as part of the event message SHALL conform to the [CareConnect-Location-1](https://fhir.hl7.org.uk/STU3/StructureDefinition/CareConnect-Location-1) constrained FHIR profile and the additional population guidance as per the table below:

| Resource Cardinality | 0..* |

| Element | Cardinality | Additional Guidance |
| --- | --- | --- |
| identifier | 0..* | Where available the ODS Site Code slice should be populated |


## Examples


<div class="tabPanel">

	<div class="tabHeadings">
		<span class="tabHeading" id="new-pointer">New (Pointer Only)</span>
		<span class="tabHeading" id="new-given">New Given (inc Imms)</span>
		<span class="tabHeading" id="new-notgiven">New Not Given (inc Imms)</span>
		<span class="tabHeading" id="update">Update (inc Imms)</span>
		<span class="tabHeading" id="delete">Delete (inc Imms)</span>
	</div>
	
	<div class="tabBodies">
	
		<div class="tabBody" id="new-pointerBody" markdown="span">
			```{% include_relative examples/vaccinations-1-new-pointer.xml %}```
		</div>
		
		<div class="tabBody" id="new-givenBody" markdown="span">
			```{% include_relative examples/vaccinations-1-new.xml %}```
		</div>
		
		<div class="tabBody" id="new-notgivenBody" markdown="span">
			```{% include_relative examples/vaccinations-1-notgiven-new.xml %}```
		</div>
		
		<div class="tabBody" id="updateBody" markdown="span">
			```{% include_relative examples/vaccinations-1-update.xml %}```
		</div>
		
		<div class="tabBody" id="deleteBody" markdown="span">
			```{% include_relative examples/vaccinations-1-delete.xml %}```
		</div>
		
	</div>
</div>

