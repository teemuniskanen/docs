# FHIR resources in EBMeDS

## SelfCareActivityDefinition

*Inherits from*: [ActivityDefinition](https://www.hl7.org/fhir/activitydefinition.html)

*FHIR profile*: [SelfCareActivityDefinition](https://simplifier.net/DuodecimCDS/SelfCareActivityDefinition/)

The main point of the FHIR resource [SelfCareActivityDefinition](https://simplifier.net/DuodecimCDS/SelfCareActivityDefinition/) is to describe an action. In the traditional CDS context the primary unit is the reminder, an informative message that may have an encoded action suggestion attached to it. In the self-care context it is the other way around: we return actions that have messages attached to them.

In practice, this means that if several reminders suggest the same action, only one SelfCareActivityDefinition is returned, with several reminder texts attached to it. This also means that the card structure of CDS hooks becomes meaningless, see the discussion at the bottom of [this page](cds-hooks.md#customselfcarecardformat).

### Data model

The ActivityDefinitions in the `selfcare-*` hooks separate the reminder texts from their suggested actions. Given a set of reminders, each with 0 or more suggested actions attached to them, the reminder texts and actions are split up into several SelfCareActivityDefinitions:

1. Zero or one ActivityDefinition containing the code system `https://duodecim.fi/fhir/stu3/CodeSystem/activity-definition-custom-actions` and the code `read-reminder`. This resource contains all reminder texts with severity level `reminder` in its `topic.text` field.
2. Zero or one ActivityDefinition containing the code system `https://duodecim.fi/fhir/stu3/CodeSystem/activity-definition-custom-actions` and the code `read-prompt`. This resource contains all reminder texts with severity level `prompt` in its `topic.text` field.
3. Zero or one ActivityDefinition containing the code system `https://duodecim.fi/fhir/stu3/CodeSystem/activity-definition-custom-actions` and the code `read-alert`. This resource contains all reminder texts with severity level `alert` in its `topic.text` field.
4. Zero or more ActivityDefinition containing any other code system and code, representing the actual actions.

In other words: ActivityDefinitions of type 1-3 may or may not be present, but at least one of them contains at least one reminder text. See below for an example.

### Code systems

These code systems are used when coding an ActivityDefinition. See the [code systems page](code-systems.md) for more detailed information.

* `https://duodecim.fi/fhir/stu3/CodeSystem/activity-definition-custom-actions`: custom codes identifying special types of ActivityDefinitions, like above.
* `http://91.202.112.142/codeserver/pages/classification-view-page.xhtml?classificationKey=2023&versionKey=2283` a.k.a. OID `1.2.246.537.6.49`: THL Sosiaali- ja terveysalan palvelunimikkeistö
* `http://91.202.112.142/codeserver/pages/classification-view-page.xhtml?classificationKey=88&versionKey=120` a.k.a. OID `1.2.246.537.6.3`: Kuntaliitto

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
* `topic` (array, **optional**):
    * `text` (markdown string): The reminder text itself. May contain http links and text with light styling.
* `participant` (array of objects): A list of participants and their roles. In practice there is only ever one entry in this list. Each object contains the following field:
    * `type` (string): one of `patient`, `practitioner` or `related-person`. Only `patient` is in practical use at the moment.
* `copyright` (string): A standard copyright notice.

## SelfCareQuestionnaireResponse

*Inherits from*: [QuestionnaireResponse](https://www.hl7.org/fhir/questionnaireresponse.html)

*FHIR profile*: [SelfCareQuestionnaireResponse](https://simplifier.net/DuodecimCDS/SelfCareQuestionnaireResponse)

The [SelfCareQuestionnaireResponse](https://simplifier.net/DuodecimCDS/SelfCareQuestionnaireResponse) is the counterpart to a Questionnaire: it contains the answers that the user has provided. This can be sent to EBMeDS for decision support.

Here is a complete example:

```json
{
  "resourceType": "QuestionnaireResponse",
  "id": "0685d814-f4af-41a3-8547-5c7e5f97c923",
  "questionnaire": {
    "identifier": {
      "system": "https://duodecim.fi/fhir/sid/vkt-questionnaire-id",
      "value": "21"
    }
  },
  "status": "completed",
  "item": [
    {
      "linkId": "introduction",
      "text": "A container item. Has no answer, but contains other items."
      "item": [
        {
          "linkId": "266",
          "text": "A display-type question in the Questionnaire, has no answer but must be present according to the FHIR spec."
        },
        {
          "linkId": "3",
          "text": "A question with a numeric answer.",
          "answer": [
            {
              "valueDecimal": 45
            }
          ]
        },
        {
          "linkId": "305",
          "text": "A multiple-choice question. We need to see the Questionnaire resource to know if it is a 'radio button' or 'check box' type multiple choice.",
          "answer": [
            {
              "valueCoding": {
                "id": "462"
              }
            }
          ]
        },
        {
          "linkId": "306",
          "text": "'Check box' type multiple choice questions may naturally have > 1 answers.",
          "answer": [
            {
              "valueCoding": {
                "id": "452"
              }
            },
            {
              "valueCoding": {
                "id": "453"
              }
            }
          ]
        },
        {
          "linkId": "23"
          "text": "A boolean-type question."
          "answer": [
            {
              "valueBoolean": true
            }
          ]
        }
      ]
    }
  ]
}
```

So the structure has the following fields:

* `resourceType` (string): Always `QuestionnaireResponse`.
* `id` (string, **optional**): A unique ID, UUID works great. Not needed for CDS, but good to have for archiving purposes.
* `questionnaire.identifier` (object): A unique identifier for the Questionnaire that this QuestionnaireResponse answers.
    * `system` (string): Always `https://duodecim.fi/fhir/sid/vkt-questionnaire-id`
    * `value` (string): A unique ID for the Questionnaire.
* `status` (string): Always `completed`.
* `item` (array of objects): Tree hierarchy of the answers to the questionnaire. The hierarchy must be the same as the question hierarchy in the Questionnaire.
    * `item` (see above, **optional**): Items may contain other items.
    * `linkId` (string): The unique ID of the question in the Questionnaire resource (also named `linkId` there). This ID is in fact globally unique for all Duodecim Questionnaires.
    * `answer` (array, **optional**): array of objects with property `value[x]`, corresponding to the Questionnaire `answer[x]` field for the question. The length of the array is always 1, or >= 1 if it is a multiple choice question (`valueCoding`).
        * `valueBoolean` (boolean): A yes or no question answer.
        * `valueDecimal` (number): A decimal-valued question answer.
        * `valueCoding` (object): has one property, `id` (string), the ID of the wanted answer. Also globally unique ID in all Duodecim Questionnaires.
    * `text` (**optional**): The question text from the Questionnaire repeated here, mostly to aid debugging.