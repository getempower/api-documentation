# Empower Export API Documentation

(This document lives at https://github.com/getempower/api-documentation/blob/master/README.md)

We may add fields to these objects as new features are rolled out. As such, we recommend building pipe-lines in ways that can account for the objects growing.

## Request

- Make an HTTP GET request to `https://api.getempower.com/v1/export`

- The request must include an HTTP header with key `secret-token` and the proper value.

## Response

- The response `Content-Type` header will always be `application/json; charset=utf-8`

- The response body will always be a JSON object

- There will always be a top-level `success` key whose value is a boolean

  - `true` means the request went through without any errors
  - `false` means otherwise
  - If any unexpected errors occur, we will be automatically notified; but please feel free to also let us know.

- In the success case, the keys on the top-level object will be:
  - profiles (an array of `profile` objects)
  - ctas (short for 'calls to action') (an array of `cta` objects)
  - ctaResults (an array of `ctaResult` objects)
  - regions (an array of `region` objects)
  - outreachEntries (an array of `outreachEntry` objects)
  - profileOrganizationTags (an array of `profileOrganizationTag` objects)

### Shape of `profile` object

A profile is a node in the relational tree:

```javascript
{
  "eid": "u-435-34543", // opaque globally unique id
  "parentEid": "u-4-2074", // opaque globally unique id of relational parent
  "role": "volunteer", // one of "campaignDirector", "organizer", "volunteer", "contact"
  "firstName": "Albert", // always a string; could be empty
  "lastName": "Einstein", // always a string; could be empty
  "email": "al@relativity.com", // could be null
  "phone": "8175551234", // could be null
  "city": "San Francisco", // could be null
  "state": "CA", // could be null
  "zip": "94110", // could be null
  "address": null, // could be a string containing first line of address
  "address2": null, // could be a string containing second line of address
  "regionId": 15, // could be null. refers to a particular region
  "vanId": 23234, // could be null if not connected to VAN or if this profile isn't matched
  "myCampaignVanId": 98234, // could be null if not connected to VAN or if this profile isn't matched
  "vanMatchStatus": null, // autoMatched, failedAutoMatch, manuallyMatched, failedManualMatch, matchFromImport, null
  "createdMts": 1592958136539, // millisecond unix timestamp
  "updatedMts": 1714408323525, // millisecond unix timestamp
  "notes": "10/10 Account No Notes", // can be null, can be empty, contains user-defined data
  "lastUsedEmpowerMts": 1592958136789,  // millisecond unix timestamp
  "currentCtaId": 2164, // which CTA is active for them right now
  "activeCtaIDs": [2164] // placeholder for future functionality of having multiple CTAs per person
}
```

For example, a profile with role=organizer may have multiple children with role=volunteer. The parent/child relationship is established via `parentEid`.

Similarly, a profile with role=volunteer who has added some of their friends and family as contacts would have multiple child profiles that each have role=contact.

### Shape of `cta` object

A call to action is something that folks are supposed to do (in particular, profiles with role=volunteer are supposed to reach out to their children, who will generally have role=volunteer or role=contact)

```javascript
{
  "id": 499, // unique identifier
  "name": "Ask if they're registered to vote", // a string title
  "description": "Voting by mail has started!", // a short sub-title description of the CTA

  // may contain arbitrary html
  "instructionsHtml": "<p>Thank you for becoming a leader! Below is your prioritized list, presently prioritized by their likelihood to vote.&nbsp; High priority is your friends and family that are the least likely to vote.&nbsp; Give them a call and let them know why you care so much about issues that affect and are affect by our government, locally and at the State and Federal levels as well. &nbsp;<b r><br>Let them know you'll be checking in with them about these issues throughout the year as elections come and go.<b r><br>And thank you so much for joining the program!</p>",

  // Deprecated. A list of questions that should be asked of the people beneath you
  // in the relational tree
  "questions": [
    {
      "type": "normal", // one of "normal", "van" describes the source of the survey question
      "key": 1,
      "text": "Are they registered?",
      "options": [
        "Yes",
        "No",
        "I helped them register"
      ],
      "values": null, // If a VAN survey question, lists the Survey Response IDs for each option
      "surveyQuestionVanId": null // If a VAN survey question, the Survey Question ID for the question
    },
    {
      "type": "normal",
      "key": 2,
      "text": "Do they know where their polling place is?",
      "options": [
        "Yes",
        "No",
        "I helped them look it up"
      ],
      "values": null,
      "surveyQuestionVanId": null
    }
  ],

  // A list of questions that should be asked of the people beneath you in the
  // relational tree. Supersedes "questions". Allows for more than 3 questions and
  // different types of answer inputs ("RADIO" if the user can select at most 1 answer,
  // "CHECKBOX" for multiple answers).
  "prompts": [
    {
      "id": 123,  // auto-assigned numeric ID
      "ctaId": 499, // ID of parent CTA
      "promptText": "Are they registered?",
      "answerInputType": "RADIO",
      "ordering": 1, // 1-indexed position in parent CTA, equivalent to questions[n].key
      "answers": [{
        "id": 456,  // auto-assigned numeric ID
        "promptId": 123,  // ID of parent prompt
        "answerText": "Yes",
        "ordering": 1  // 1-indexed position in parent prompt
      }, {
        "id": 457,
        "promptId": 123,
        "answerText": "No",
        "ordering": 2
      }, {
        "id": 458,
        "promptId": 123,
        "answerText": "I helped them register",
        "ordering": 3
      }]
    }, {
      "id": 124,
      "ctaId": 499,
      "promptText": "Do they know where their polling place is?",
      "answerInputType": "RADIO",
      "ordering": 2,
      "answers": [{
        "id": 459,
        "promptId": 124,
        "answerText": "Yes",
        "ordering": 1,
      }, {
        "id": 460,
        "promptId": 124,
        "answerText": "No",
        "ordering": 2,
      }, {
        "id": 461,
        "promptId": 124,
        "answerText": "I helped them look it up",
        "ordering": 3
      }]
    }
  ],

  "createdMts": 1563967360107, // millisecond unix timestamp
  "updatedMts": 1561987307860, // millisecond unix timestamp

  // A list of different attachments in the CTA that volunteers can easily share with their contacts, organizations can attach up to 3 shareables
  "shareables": [
    {
     "type": "link", //one of link, image
     "url": "https://www.everyaction.com", // the URL of the shareable link (field only exists when type 'link')
     "imageFilestackHandle": "43q289jfip", // a unique ID for the image (field only exists when type 'image')
     "displayLabel": "Go vote!" // label shown to users around the shareable
    }
  ],


  //if prioritizations are turned on, a map of activist code IDs to label with the corresponding priority
  "prioritizations": [
    {
      "labelKey": "highPriority",
      "vanActivistCodeId": 4356, //legacy for when CTAs were prioritized by activist codes
      "savedListId": 3279 // ID of the saved list used to prioritize contacts on volunteer's lists
    }
  ],
  "defaultPriorityLabelKey": "lowPriority", // In a CTA with prioritizations built in, the label for remaining people not in the high priority bucket
  "regionIds": [245, 289], // regions the CTA is active for

  "recruitmentQuestionType": "VoteTripling", // one of voteTripling, invite, None to indicate which type of recruitment ask should come after the CTA
  "recruitmentTrainingUrl": "www.zoom.us", // a link that volunteers can send to contacts to invite them to join a training on how to relationally organize

  "isIntroCta": false, // whether or not the CTA is an Intro CTA
  "scheduledLaunchTimeMts": 1563967360107, // millisecond unix timestamp; if the CTA created and deployed immediately, same as the createdMts
  "activeUntilMts": null, // millisecond unix timestamp. The CTA will be disabled after this point.
  "shouldUseAdvancedTargeting": true, // true or false

  // a dictionary detailing which filters to use for the targeting of this CTA. All unused filters are set to null
  "advancedTargetingFilter": {
      "joinDate": {"type": "between", "toMts": 1600981504354, "fromMts": 1600549504354}, // currently, only the "between" type is supported (in the last X days is implemented as a between filter)
      "region": [3368], // list of regions of the users who should see the CTA
      "role": {"campaignDirector": true, "organizer": false, "volunteer": true, "contact": false}, // only keys possible
      "assignedTo": ["fbei678"], // when targeting by a leader's parent, a list of those parents' EIDs
      "listSize": {"max": 10, "min": 0, "type": "between"}, //  In addition to the "between" type, you can use "greaterThan" with min or "lessThan" with max.
      "hasContactsInState": ["WI"], // filter by whether volunteers have contacts in this list of states
      "hasCtaResponse": [{"ctaId": 648, "promptId": 123, "answerId": 456}] // a list of dictionaries if multiple filters applied
      "tag": {"ids": [59], "matchingType": "allTags"}, // ids of the tags.  matchingType is "anyTags", "allTags", or "noneTags"
      "hasContactsWithTags": {"ids": [59], "matchingType": "allTags"}, // same format as tag filter
      "city": ["Madison"], // list of cities of the users who should see the CTA
      "state": ["WI"], // list of states of the users who should see the CTA
      "zipCode": ["53201"], // list of zip codes of the users who should see the CTA
   },

  "organizationId": 4 // should always be the same as your organization ID
}
```

### Shape of `ctaResult` object

If someone reaches out to someone with role=contact and logs their outreach, potentially including answers to the question(s) in the call to action, it will show up as a record here

```javascript
{
   // the profileEid of the person that was contacted.
   // whoever their parent is should receive "credit" for the outreach.
  "profileEid": "c-33129",
  "ctaId": 8, // the call to action they were contacted about
  "contactedMts": 1410169604253, // millisecond unix timestamp
  "updatedMts": 1520169604226, // millisecond unix timestamp
  "initialPromptResponse": 1, // value of the prompt response, as defined below

  // Deprecated. A map from question key to a single selected option.
  "answers": {
    "1": "Yes", // one of the options from the cta; could be null
    "2": null,
    "3": null
  },

  // A map from prompt ID to a list of selected answer IDs. Supersedes "answers".
  // Values and list elements are never null, but lists may be empty.
  "answerIdsByPromptId": {
    "123": [456],
    "124": []
  },

  "notes": "They said they're going to vote" // could be null
}
```

The initialPromptResponse value is an enum that represents the response. The mapping is as follows:
- 0: No
- 1: Yes
- 2: InProgress
- 3: SomeoneElseResponded
- 4: NoAnswer

### Shape of `region` object

A region is not necessarily geographic; it's however an organization has decided to divide up its people for targeting purposes.

```javascript
{
  "id": 1154, // a unique numeric identifer
  "name": "North Side",
  "inviteCode": "norside", // could be null
  "ctaId": 597, // after the introduction of filtered and targeted CTAs, this became a deprecated field.
  "organizationId": 4, // can ignore
  "description": null // a description of the region; can be null
}
```

### Shape of `outreachEntry` object

The outreachEntries is a list of objects that represent the organizer-to-volunteer outreach data (including notes, follow up schedule, etc) for a specific organizer and volunteer combination.

```javascript
{
  "organizerEid": "u-4-18710",                 // Organizer's id; string
  "targetEid": "u-4-121",                      // Who they talked to, id; string
  "outreachCreatedMts": 1590552745646,         // When outreach was logged, millisecond epoch; int
  "outreachDidGetResponse": false,             // ; bool
  "outreachContactMode": null,                 // Currently 'phone', 'text' or 'messenger'; string or NULL
  "outreachEngagementLevel": null,             // ; NULL
  "outreachNote": null,                        // Typed notes; string or NULL
  "outreachCtaProgress": null,                 // 'notStarted', 'inProgress', 'done'; string or NULL
  "outreachSnoozeType": "untilMts",            // 'untilMts', 'indefinite'; string or NULL
  "outreachSnoozeUntilMts": 1590811944832,     // when snooze expires, must be for 'untilMts' snooze type; int or NULL
  "outreachScheduledFollowUpMts": null,        // when a follow up reminder is scheduled, millisecond epoch; int
  "outreachCurrentCtaId": 682                  // What CTA the outreach corresponds to; int
}
```

### Shape of `profileOrganizationTag` object

The profileOrganizationTags is a list of objects that represent the tags assigned to each profile in organization.

```javascript
{
  "profileEid": "000LOjiX0OjZKy",  // eid of the profile that has the tag; string
  "tagId": 86705                   // id of the tag assigned to that profile; number
}
```

## Example Access Code

### Bash

```bash
# Remember to replace the secret-token with your own
curl -i -H "secret-token: ea4fc8aa-cfc0-48ca-932d-5ff3782ab7ae" https://api.getempower.com/v1/export
```

### Python

```python
import requests # Library: https://requests.readthedocs.io/

url = 'https://api.getempower.com/v1/export'
secret_token = 'ea4fc8aa-cfc0-48ca-932d-5ff3782ab7ae' # Replace with your own
result = requests.get(url, headers={'secret-token': secret_token}).json()
print(result)
```

## Security

If your `secret-token` is ever exposed or you would like to rotate it, just let us know.
