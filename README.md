# Empower Export API v2 Documentation

(This document lives at https://github.com/getempower/api-documentation/blob/master/README.md)

The Empower API provides a set of endpoints to return CSV information for your organizations.

The documentation for API v1 can be found [here](https://github.com/getempower/api-documentation/blob/master/README_v1.md).

### What's changed since v1?

<details>

- The v1 export fetches all available data for an organization. Requests in v2 can receive a `startMts` and `endMts` argument to limit the amount of data retrieved.
- Returns are no longer in a single object returned in JSON. Instead, data that would be returned as an array is now in its own endpoint.

</details>

## Export Order

If you intend to use data from this export in a SQL or similar database, we suggest calling the export in the following order so that necessary keys are present.

```
/regions
/tags
/ctas
/ctas/prompts
/ctas/prompts/answers
/profiles
/profiles/tags
/ctas/results
/ctas/results/answers
/outreachEntries
```

Importing in this order ensures that any key constraints are met. When importing, it is better to include an `endMts` argument to prevent dangling references if new data is created during a series of exports.

### Structure of a Call to Action

A call to action is a set of instructions, often with a set of prompts for the user to ask about when speaking to a contact.

The definition of a call to action looks like this

```
ctas -1:M-> prompts -1:M-> answers
```

When a user records outreach to a contact, it creates a result. `results/answers` records all answer selections the user recorded for the response.

```
result (call to action id, user eid, contact eid) -1:M-> results/answers (result id, answer id, prompt id)
```

## Request

- Make an HTTP GET request to `https://api.getempower.com/v2`

- The request must include an HTTP header with key `secret-token` and the proper value. `secret-token` can also be a comma separated list of tokens (eg: `TOKEN_1,TOKEN2`). Information is returned in one CSV, but each export includes the organization id associated to the row.

- The request must include a `startMts` argument, a millisecond epoch timestamp defining the beginning of the range for export

- A request may include an argument for `endMts`, a millisecond epoch timestamp defining the end of the range for the export. An `endMts` timestamp is recommended to prevent dangling references if data is created during the export process.

### Example Requests

#### cURL

```
curl --request GET \
  --url 'http://localhost:3000/v2/export/profiles?startMts=1767225600000' \
  --header 'secret-token: {YOUR_TOKEN}'
```

#### Python

```python
import requests # Library: https://requests.readthedocs.io/

url = 'http://localhost:3000/v2/export/{EXPORT}'
secret_token = '{YOUR_TOKEN}'
startMts = 1767225600000
result = requests.get(url, headers={'secret-token': secret_token}, params={'startMts': startMts}).text
print(result)
```

### Endpoints

#### /profiles

Information about the people in your organization

<details>

- **eid** - Primary key of a profile

- **organizationId** - Id of profile's organization; foreign key to `organizations.id`

- **parentProfileEid** - EID of profile's current parent

- **role** - Role of profile in organization, one of `"volunteer"`, `"organizer"`, `"campaignDirector"`

- **firstName** - Profile first name

- **lastName** - Profile last name

- **email** - Profile email

- **phone** - Profile phone

- **address** - Profile address line 1

- **address2** - Profile address line 2

- **city** - Profile city

- **state** - Profile state, 2 characters

- **zip** - Profile 5 digit zip code

- **myVotersVanId** - If the organization is connected to VAN with MyVoters integration enabled, the ID of the contact in VAN MyVoters

- **myCampaignVanId** - If the organization is connected to VAN with MyCampaign integration enabled, the ID of the contact in VAN MyCampaign

- **vanMatchStatus** - Whether the contact has been matched to a contact in VAN. One of `"autoMatched"`, `"failedAutoMatch"`, `"manuallyMatched"`, `"failedManualMatch"`, `"matchFromImport"` or `NULL`

- **lastUsedEmpowerMts** - Millisecond epoch timestamp, the last time the user signed into Empower

- **notes** - Notes entered on the profile page

- **regionId** - The region the profile is in; foreign key to `regions.id`

- **createdMts** - When the profile was created, millisecond timestamp

- **updatedMts** - When the profile was last updated, millisecond timestamp. Will be `""` if the profile has never been updated.

- **isDeleted** - Whether is the profile is deleted, `true` or `false`

- **createdByProfileEid** - EID of the profile that created this profile

- **updatedByProfileEid** - EID of the profile that last updated this profile

- **referredByEid** - For referral programs, EID of the profile that referred this profile

- **relationship** - Whether this is a profile added personally or through canvassing. `"personal"` or `"canvassing"`

- **firstUsedEmpowerMts** - When this profile first signed into Empower, millisecond timestamp

- **phoneAddressBookEntryId**

- **canManageOwnCtas** - For organizers (role = `"organizer"`), whether or not they are able to able to modify the calls to action for the region they are in.

- **promotedByEid** - If this profile began as a contact, the EID of the person who promoted them to a user

</details>

#### /profiles/tags

Tags applied to profiles

<details>

- **eid** - EID of profile tag is associated to; foreign key to `profiles.eid`

- **tagId** - Tag ID of tag associated to profile; foreign key to `tags.id`

- **createdMts** - When the tag was associated to the profile

- **organizationId** - ID of the organization the profile is in

- **createdByProfileEid** - EID of profile that applied the tag to profile referenced in `eid`

</details>

#### /regions

Regions in your organization

<details>

- **id** - Primary key, id of the region.

- **organizationId** - ID of the organization the region is in

- **name** - Name of the region

- **inviteCode** - Latest invite code that will put users into the region when used

- **inviteCodeCreatedMts** - When the latest invite code was created

- **description** - Description of the region

- **status** - Whether the region has been deleted. `"Active"` or `"Deleted"`

</details>

#### /tags

Tags in your organization

<details>

- **id** - Primary key, ID of the tag.

- **organizationId** - ID of the organization the tag is in

- **label** - Label of the tag (what is shown in Empower)

- **description** - Description of the tag

- **createdMts** - When the tag was created, millisecond timestamp

- **updatedMts** - When the tag was last updated, millisecond timestamp. Will be `""` if the tag has never been updated.

</details>

#### /ctas

Calls to action in your organization

<details>

- **id** - Primary key, ID of the call to action.

- **organizationId** - ID of the organization the call to action is in.

- **name** - Title of the call to action.

- **description** - Subtitle of the call to action.

- **status** - Whether the call to action is showing to users. `"Active"` or `"Disabled"`.

- **instructionsHtml** - The body of the call to action in HTML format.

- **createdMts** - When the call to action was created, millisecond timestamp.

- **updatedMts** - When the call to action was last updated, millsecond timestamp. Will be `""` if the call to action has never been updated.

- **recruitmentQuestionType** - The recruitment prompt selected under 'Recruitment survey question' in the editor. `"invite"`, `"training"`, `"voteTripling"`, `"none"`, or `""`

- **recruitmentTrainingUrl** - URL for users to access for training. Only set when `recruitmentQuestionType` is `"training"`.

- **prioritizations** - JSON array of prioritizations for the call to action, each row associating a filter to a label.

- **isIntroCta** - Whether this an introductory call to action. `true` or `false`.

- **scheduledLaunchTimeMts** - When the call to action is(was) scheduled to become visible to users.

- **activeUntilMts** - When the call to action will no longer be visible to users.

- **shouldUseAdvancedTargeting** - Whether additional filters will be used to determine which users see the call to action. `true` or `false`.

- **advancedTargetingFilter** - JSON string of the filter to determine which users see the call to action. Will only be set if `shouldUseAdvancedTargeting` is `true`.

- **canvassingTargetingFilter** - Filter applied to users' contacts to determine if the user should see the call to action. Only applies to list based call to actions (phone, text, doors).

- **contactTargetingFilter** - For `"organizerFollowUp"` calls to action, the filter to be applied to users' contacts. If a contact matches, the user will see the call to action.

- **defaultPriorityLabelKey** - The key for the label that will display for all profiles that do not fit a prioritization filter if prioritizations are enabled for the call to action.

- **actionType** - The type of call to action. `"personal"`, `"relational"`, `"canvassing"`, `"phoneCanvassing"`, `"textCanvassing"`, `"doorCanvassing"`, or `"organizerFollowUp"`.

- **turfCuttingType** - For canvassing calls to action, whether the uploaded list assigns turfs or allows users to request turfs dynamically. `"request"`, `"manual"` or `NULL`.

- **conversationStarter** - Text shown to users as an example of how to begin a conversation for the call to action. May include placeholders such as `{recipientname}`, `{sendername}`, and `{orgname}`.

- **customRecruitmentPromptText** - Text shown to users to prompt them based on `recruitmentQuestionType`. May be custom if `recruitmentQuestionType` is `"invite"`.

- **shouldShowMatchButton** - Whether the user will be given the option to match their contacts against a voterfile in the call to action.

</details>

#### /ctas/prompts

The survey questions for your CTAs

<details>

- **id** - Primary key, id of the prompt

- **organizationId** - ID of the organization the call to action the prompt is associated to is in.

- **ctaId** - ID of the call to action the prompt is associated to

- **promptText** - Text of the prompt

- **vanId** - VAN survey question ID

- **isDeleted** - Whether the prompt has been deleted from the call to action. `true` or `false`. Deletion means that the prompt does not show in the call to action, but does not mean there are no responses if the prompt was previously visible.

- **answerInputType** - The input type displayed for the prompt when entering a call to action result. `RADIO`, `CHECKBOX`, `FREE_RESPONSE` or `FORM`.

- **ordering** - The order prompts will be displayed to users in.

- **dependsOnInitialDispositionResponse** - Whether this prompt requires a particular answer to `initialPromptResponse` in results. For canvassing, `3 = SomeoneElseResponded` will cause an additional prompt to be displayed for who they spoke to.

- **usesMessageDrafts** - Whether message drafts were provided for this prompt. `true` or `false`. If `true`, `defaultReplySuggestion` may be provided for answers associated to this prompt, and represent a draft message for the user to modify and send in response during a conversation.

- **usesTalkingPoints** - Whether talking points were provided for this prompt. `true` or `false`. If `true`, `defaultReplySuggestion` may be provided for answers associated to this prompt, and represent additional information provided to users to help respond to someone their speaking with.

- **parentPromptId** - The original prompt this prompt is associated to if it is a saved survey prompt.

</details>

#### /ctas/prompts/answers

The defined answers to the survey questions for your CTAs.

<details>

- **id** - Primary key, ID of the call to action answer.

- **organizationId** - ID of the organization the answer is associated to is in.

- **promptId** - ID of the prompt the answer is to.

- **answerText** - The text of the answer displayed in the application.

- **vanId** - If the prompt is associated to VAN, the response this answer is associated to.

- **isDeleted** - Whether the answer is still selectable, `true` or `false`.

- **ordering** - The order the answer will appear in under the prompt.

- **defaultReplySuggestion** - Depends on `prompts.usesMessageDrafts` and `prompts.usesTalkingPoints`. Either the message draft or the talking points associated to this answer.

- **parentAnswerId** - If the prompt this answer is associated to has a `parentPromptId`, the answer under that prompt that this answer is associated to.

- **isFreeResponse** - Whether this answer is a free response option, giving the user a space to enter freeform text

</details>

#### /ctas/results

A response to the call to action. For a call to action, a user will have one of these to each contact they've spoken to.

<details>

- **id** - Primary key, ID of the result

- **organizationId** - ID of the organization this result is associated to is in.

- **userEid** - EID of the person reaching out to a contact.

- **contactEid** - EID of the contact reached out to in this result.

- **ctaId** - ID of the call to action this result is associated to, foreign key to `ctas.id`.

- **createdMts** - When this response was first logged, millisecond timestamp.

- **updatedMts** - When this response was last updated, millisecond timestamp.

- **initialPromptResponse** - For canvassing, how the initial question about whether the user was able to speak to the contact. 0 = No, 1 = Yes, 2 = InProgress, 3 = SomeoneElseResponded, 4 = NoAnswer.

</details>

#### /ctas/results/answers

Join connecting a result (conversation from user to contact) to the answers selected as part of that conversation.

<details>

- **organizationId** - ID of the organization this result answer is associated to is in.

- **resultId** - ID of the result this answer is associated to, foreign key to `ctas/results/` `id`.

- **answerId** - ID of the answer selected for this result.

- **promptId** - ID of the prompt being responded to for this answer.

- **freeResponseAnswerText** - If the answer allows for free response, the text entered for that response.

</details>

#### /outreachEntries

The notes entered when a director or organizer selects 'Log a note' while viewing another user's profile.

<details>

- **organizationId** - ID of the organization this result answer is associated to is in.

- **organizerEid** - EID of the profile creating the note

- **targetEid** - EID of the profile the note is about

- **outreachCreatedMts** - When the note was logged

- **outreachNote** - Content of the note

- **outreachCurrentCtaId** - The call to action that was most recently assigned to the user when the note was logged.

</details>

## Response

- The response `Content-Type` header will always be `	text/csv`

- The response body will be a CSV document.

- **Processes that consume the export should always use column headers, not column index.**

### Example CSV Response

```
"id","organizationId","label","description","createdMts","updatedMts"
"1","1","TAG","","1767225600000",""
"2","1","TAG2","","1767225600001",""
```

## Security

If your `secret-token` is ever exposed or you would like to rotate it, just let us know.
