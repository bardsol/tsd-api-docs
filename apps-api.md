
# Using the apps API - an example

## Use case

In this example there are three different roles, each with different levels of data access:

1. doctors - read and write on individual and aggregate data
2. patients - read and write on their own individual data
3. researchers - read on aggregate data

## Access tokens

The following token types are defined to cover these roles:

* `app-processor`, requested by doctors
* `app-subject`, requested by patients
* `app-generic`, requested by researchers

## Identity management requirements

The following groups should be created:
* `p[0-9]+-{app}-processor-group`
* `p[0-9]+-{app}-subject-group`

In the use case above, the following would apply:

* doctors are members of the `p[0-9]+-{app}-processor-group`
  * a TSD project member requests group creation (a person group)
  * doctors apply for TSD project membership
  * TSD project admins approve them as associated members
* patients are members of the `p[0-9]+-{app}-subject-group`
  * a TSD project member requests group creation (a person group)
  * patient person objects are created
  * patients are added to groups by processors
* researchers do not need to be a member of any group

## Using the API

Create a person object for a new patient:
```txt
POST /v1/{pnum}/apps/{app}/iam/persons
Authorization: Bearer $app-processor-token
Content-Type: application/json

{
    "full_name": "First Last",
    "birth_date": "2000-01-01",
    "identifiers": {
        "id": "00010112345",
        "type": "id_number",
        "issuer": {"country": "NO"},
    }
    "email": "my@mail.no"
}
```

If a person with the same `identifiers` has already been created, then the above request will return the person ID, without modifying the rest of the attributes. If a new person is created, the same response will be returned:
```json
{
    "person_id": "some-uuid-value"
}
```

Add them to the subject group:
```
PUT /v1/{pnum}/apps/{app}/iam/groups/{pnum}-{app}-{subject}-group/members
Authorization: Bearer $app-processor-token
Content-Type: application/json

{
    "member": "person_id"
}
```

A subject adds data about themselves to a hypothetical survey named `study1`, or a processor does so on their behalf:
```
PUT /v1/{pnum}/apps/{app}/tables/persons/study1?where.id=eq.{person_id}
Authorization: Bearer $app-processor-token|$app-subject-token
Content-Type: application/json

{
    "id": "person-id",
    "data": {...}
}
```

A subject gets their own data, or a processor gets data about a specific subject:
```
GET /v1/{pnum}/apps/{app}/tables/persons/study1?where.id=eq.{person_id}
Authorization: Bearer $app-processor-token|$app-subject-token
```

A processor gets data about all subjects:
```
GET /v1/{pnum}/apps/{app}/tables/persons/study1
Authorization: Bearer $app-processor-token
```

Any token can be used to add data to the generic endpoints, for example summary statistics about `study1`:
```
PUT /v1/{pnum}/apps/{app}/tables/study1-summary
Authorization: Bearer $app-processor-token|$app-subject-token|$app-generic-token
```

And to read from it:
```
PUT /v1/{pnum}/apps/{app}/tables/study1-summary
Authorization: Bearer $app-processor-token|$app-subject-token|$app-generic-token
```

## Other API calls

List the members of groups:
```
GET /v1/{pnum}/apps/{app}/iam/groups/{pnum}-{app}-{subject|processor}-group/members
Authorization: Bearer $app-processor-token
```
