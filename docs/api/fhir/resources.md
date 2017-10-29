# FHIR resources in EBMeDS

## SelfCareActivityDefinition

Base resource: [ActivityDefinition](https://www.hl7.org/fhir/activitydefinition.html)

The main point of the FHIR resource [SelfCareActivityDefinition](https://simplifier.net/DuodecimCDS/SelfCareActivityDefinition/) is to describe an action. In the traditional CDS context the primary unit is the reminder, an informative message that may have an encoded action suggestion attached to it. In the self-care context it is the other way around: we return actions that have messages attached to them.

In practice, this means that if several reminders suggest the same action, only one SelfCareActivityDefinition is returned, with several reminder texts attached to it. This also means that the card structure of CDS hooks becomes meaningless, see the discussion at the bottom of [this page](cds-hooks.md#customselfcarecardformat).

### Data model

The ActivityDefinitions in the `selfcare-*` hooks separate the reminder texts from their suggested actions. Given a set of reminders, each with 0 or more suggested actions attached to them, the reminder texts and actions are split up into several SelfCareActivityDefinitions:

1. Zero or one ActivityDefinition containing the code system `https://duodecim.fi/fhir/stu3/CodeSystem/activity-definition-custom-actions` and the code `read-reminder`. This resource contains all reminder texts with severity level `reminder` in its `topic.text` field.
2. Zero or one ActivityDefinition containing the code system `https://duodecim.fi/fhir/stu3/CodeSystem/activity-definition-custom-actions` and the code `read-prompt`. This resource contains all reminder texts with severity level `prompt` in its `topic.text` field.
3. Zero or one ActivityDefinition containing the code system `https://duodecim.fi/fhir/stu3/CodeSystem/activity-definition-custom-actions` and the code `read-alert`. This resource contains all reminder texts with severity level `alert` in its `topic.text` field.
4. Zero or more ActivityDefinition containing any other code system and code, representing the actual actions.

In other words ActivityDefinitions of type 1-3 may or may not be present, but at least one of them contains at least one reminder text. See below for an example.

### Code systems

See the [code systems page](code-systems.md) for more detailed information.

* `https://duodecim.fi/fhir/stu3/CodeSystem/activity-definition-custom-actions`: custom codes identifying special types of ActivityDefinitions, like above.
* `http://91.202.112.142/codeserver/pages/classification-view-page.xhtml?classificationKey=2023&versionKey=2283` a.k.a. OID `1.2.246.537.6.49`: THL Sosiaali- ja terveysalan palvelunimikkeistö

### Restrictions

Some restrictions have been put in place to simplify data processing, using the [SelfCareActivityDefinition](https://simplifier.net/DuodecimCDS/SelfCareActivityDefinition/) profile.

* A SelfCareActivityDefinition is described by exactly one code, i.e. `code` must be present and `code.coding` may contain only one coding. The coding represents some kind of action.
* A reminder must have exactly one `participant`, i.e. the intended recipient of this SelfCareActivityDefinition. This is either the patient or a medical professional.
* The ActivityDefinitions are always freshly generated, so their `status` is always `active`.

### Example

Below is an example of the two kinds of SelfCareActivityDefinitions (one describing an action, one containing reminder texts) in a CDS response:

```json
{
  "cards": [
    {
      "summary": "Selfcare action suggestions",
      "indicator": "info",
      "source": {
        "name": "EBMeDS v2.0.6",
        "url": "https://ebmeds.org/version/v2.0.6/data-version/v0.9.2"
      },
      "suggestions": [
        {
          "resourceType": "ActivityDefinition",
          "status": "active",
          "copyright": "Kustannus Oy Duodecim, 2017",
          "participant": [
            {
              "type": "patient"
            }
          ],
          "kind": "Observation",
          "code": {
            "coding": [
              {
                "code": "ADA001-P1",
                "system": "http://91.202.112.142/codeserver/pages/classification-view-page.xhtml?classificationKey=2023&versionKey=2283",
                "display": "Yleislääkärin päivystyksen palvelut - Hoidetaan päivystyksenä"
              }
            ]
          }
        },
        {
          "resourceType": "ActivityDefinition",
          "status": "active",
          "copyright": "Kustannus Oy Duodecim, 2017",
          "participant": [
            {
              "type": "patient"
            }
          ],
          "kind": "CommunicationRequest",
          "code": {
            "coding": [
              {
                "system": "https://duodecim.fi/fhir/stu3/CodeSystem/activity-definition-custom-actions",
                "code": "read-reminder",
                "display": "Lue muistutteet"
              }
            ]
          },
          "topic": [
            {
              "text": "Oireesi voivat johtua virtsatietulehduksesta, joka voi olla munuaistasoinen. Hakeudu päivystysvastaanotolle."
            }
          ]
        }
      ]
    }
  ]
}
```

And the fields are described as follows.

* `resourceType` (string): Always `ActivityDefinition` for this resource.
* `status` (string): Only `active` ActivityDefinitions are returned.
* `code` (object, CodeableConcept): The code describing the action.
    * `coding`: container object for a code + code system
        * `system` (string): The code system used in this code. See [a list of used code systems](code-systems.html).
        * `code` (string): The actual code, a symbol in the given system.
        * `display` (string): Human-readable description of the action.
* `topic`: array of objects
    * `text` (markdown string): The reminder text itself. May contain http links and text with light styling.
* `participant` (array of objects): A list of participants and their roles. In practice there is only ever one entry in this list. Each object contains the following field:
    * `type` (string): one of `patient`, `practitioner` or `related-person`. Only `patient` is in practical use at the moment.
* `copyright` (string): A standard copyright notice.