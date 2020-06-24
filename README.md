# Empower Export API Documentation

(This document lives at https://github.com/getempower/api-documentation/blob/master/README.md)

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

  // may contain arbitrary html
  "instructionsHtml": "<p>Thank you for becoming a leade! Below is your prioritized list, presently prioritized by their likelihood to vote.&nbsp; High priority is your friends and family that are the least likely to vote.&nbsp; Give them a call and let them know why you care so much about issues that affect and are affect by our government, locally and at the State and Federal levels as well. &nbsp;<b r><br>Let them know you'll be checking in with them about these issues throughout the year as elections come and go.<b r><br>And thank you so much for joining the program!</p>",

  // a list of questions that should be asked of the people beneath you
  // in the relational tree
  "questions": [
    {
      "key": 1,
      "text": "Are they registered?",
      "options": [
        "Yes",
        "No",
        "I helped them register"
      ]
    },
    {
      "key": 2,
      "text": "Do they know where their polling place is?",
      "options": [
        "Yes",
        "No",
        "I helped them look it up"
      ]
    }
  ],
  "createdMts": 1563967360107, // millisecond unix timestamp
  "organizationId": 4 // can ignore this
}
```

### Shape of `ctaResult` object

If someone reaches out to someone with role=contact and logs their outreach, potentially including answers to the question(s) in the call to action, it will show up as a reocrd here

```javascript
{
   // the profileEid of the person that was contacted.
   // whoever their parent is should receive "credit" for the outreach.
  "profileEid": "c-33129",

  "ctaId": 8, // the call to action they were contacted about
  "contactedMts": 1410169604253, // millisecond unix timestamp
  "answers": {
    "1": "Yes", // one of the options from the cta; could be null
    "2": null,
    "3": null
  },
  "notes": "They said they're going to vote" // could be null
}
```

### Shape of `region` object

A region is not necessarily geographic; it's however an organization has decided to divide up its people for targeting purposes.

```javascript
{
  "id": 1154, // a unique numeric identifer
  "name": "North Side",
  "inviteCode": "norside", // could be null
  "ctaId": 597, // the id of the cta currently active for this region
  "organizationId": 4, // can ignore
  "description": null // a description of the region; can be null
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
