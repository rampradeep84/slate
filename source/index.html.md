---
title: FreeAgent CRM API Reference

includes:
  - errors

search: true
---

# API Introduction
 
FreeAgent CRM platform exposes a graphql API endpoint, essentially any data/operation you can access via FreeAgent web interface is available via APIs.

This includes typical CRUD operations on CRM objects like leads, deals, accounts or any custom object you may have configured in your account along with ability to create Next Steps or post activities in the system.

We have documented common APIs typically needed for most integration needs, in case you need access to specific data or operation not included in this documentation, please reach out to support for assistance on the same.
 
More info on graphql and comparison with REST can be found here: [GraphQL vs REST](https://www.howtographql.com/basics/1-graphql-is-the-better-rest/)

# Authentication

API authentication is via standard and secure OAuth2 protocol, using client credential grants.

Below are sample requests from curl to try out easily from command line, you can use any language/library of choice for actual integration.

Reach out to support to get access to your accounts client_id and secret, make sure to secure the api credentials appropriately.

## Fetching OAuth Access token 

FreeAgent CRM supports client credential Oauth Authentication which should suffice for most backend integrations.

Here is a sample curl request to fetch access token, access tokens are short lived and typically expires within an hour, so make sure your integration re-fetches the access token on Auth error due to expiry while invoking the APIs.
 

```shell
    curl --request POST \
  --url 'https://freeagent.network/oauth/token' \
  --header 'content-type: application/json' \
  --data '{"grant_type":"client_credentials","client_id": "{client_id}","client_secret": "{fa_secret}"}'
```
> Response 

```json
{
    "access_token": "31N2hEzxToeLDi2J...",
    "expires_in": 3600,
    "token_type": "Bearer"
}
```

## Using access token

After getting access token, include it in the Authorization header as Bearer token, shown in the CURL example below showing fetching of list of contacts from your Freeagent CRM account. 

### GraphQL API endpoint

All API requests (queries and mutations) are via the following graphQL API endpoint.
 
`POST https://freeagent.network/api/graphql`


```shell
  curl --request POST \
    --url 'https://freeagent.network/api/graphql' \
    --header "Content-Type: application/json" \
    --header "Authorization: Bearer {access_token}"\
    --data '{"query": "query {getLeadsWithCount(limit: 2) {count  leads { full_name }  } }"}'
```
>Response

```json
{
    "data": {
        "getLeadsWithCount": {
            "count": 5000,
            "leads": [
                {
                    "full_name": "Sample Contact1"
                },
                {
                    "full_name": "Sample Contact2"
                },
            ]
        }
    }
}
```

# Queries

## Contacts List

```
getLeadsWithCount(
limit: Int,
offset: Int,
order: [[String]],
pattern: String,
filters: [Filter]) 
{
   leads: [Contact]
   count: Int
   error: String
}
```

> Sample Response

```json
{
  "data": {
    "getLeadsWithCount": {
      "count": 5000,
      "leads": [
        {
          "id": "09872f5b-7ef9-45ec-ae3b-d51fe32f456f",
          "first_name": "Test",
          "last_name": "Lead1",
          "work_email": "lead1@test.com"
        },
        {
          "id": "bc2ad479-da23-4e9c-9603-4ca6226cdeea",
          "first_name": "Test",
          "last_name": "Lead2",
          "work_email": "lead2@test.com"
        }
      ]
    }
  }
}
```

This query retrieves a list of contacts available in your account.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
limit | Int | Limit the number of results returned by the query.
offset | Int | Page offset of the results returned, used along with limit to paginate results.
order | [[String]] | Sort order of list query by a given field and sorting function("ASC" or "DESC"), eg: [["full_name", "ASC"]] sorts by full_name in ascending order
pattern | String | Search pattern, to use to filter contacts.
filters | [{ field_name: String, values: [String] }] | Filter the list by a set of fields, if multiple fields are specified will be or'ed between the fields. Note for filter values use the id's of choice or reference fields.


### Response Fields

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
leads | [Contact] | Array of Contact Objects, request exact shape of contact fields needed for your integration.   
count | Int | The total count of contacts satisfied by the query ignoring the limit specified.

## Contact by ID 

```
getLead(id: String!) 
Contact
```

> Sample Response

```json
{
  "data": {
    "getLead": {
          "id": "09872f5b-7ef9-45ec-ae3b-d51fe32f456f",
          "first_name": "Test",
          "last_name": "Lead1",
          "work_email": "lead1@test.com"
    }
  }
}
```

This query retrieves a specific contact by Id.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
id | String! | Required parameter, specifying the id of contact to lookup.


## Deals List

```
getDealsWithCount(
limit: Int,
offset: Int,
order: [[String]],
pattern: String,
filters: [Filter]) 
{
   deals: [Deal]
   count: Int
}
```

> Sample Response

```json
{
  "data": {
    "getDealsWithCount": {
      "count": 24,
      "deals": [
        {
          "id": "5077937b-54fb-4360-9cc9-a57e9a3e4794",
          "name": "Sample Deal 1",
          "amount": 100000
        },
        {
          "id": "6ef130bf-7bbb-4d6e-96eb-d16c7fc66f81",
          "name": "Sample Deal 2",
          "amount": 300000
        }
      ]
    }
  }
}
```

This query retrieves a list of deals available in your account.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
limit | Int | Limit the number of results returned by the query.
offset | Int | Page offset of the results returned, used along with limit to paginate results.
order | [[String]] | Sort order of list query by a given field and sorting function("ASC" or "DESC"), eg: [["name", "ASC"]] sorts by full_name in ascending order
pattern | String | Search pattern, to use to filter deals.
filters | [{ field_name: String, values: [String] }] | Filter the list by a set of fields, if multiple fields are specified will be or'ed between the fields. Note for filter values use the id's of choice or reference fields.


### Response Fields

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
deals | [Deal] | Array of Deal Objects, request exact shape of deal fields needed for your integration.   
count | Int | The total count of contacts satisfied by the query ignoring the limit specified. 


## Deal by ID

```
getDeal(id: String!) 
Deal
```

> Sample Response

```json
{
  "data": {
    "getDeal": {
        "id": "5077937b-54fb-4360-9cc9-a57e9a3e4794",
        "name": "Sample Deal 1",
        "amount": 100000
    }
  }
}
```

This query retrieves a specific deal by Id.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
id | String! | Required parameter, specifying the id of deal to lookup.


## Accounts List

```
logosWithCount(
limit: String,
offset: String,
order: [[String]],
pattern: String,
filters: [Filter]) 
{
   logos: [Logo]
   count: Int
}
```

> Sample Response

```json
{
  "data": {
    "logosWithCount": {
      "count": 3000,
      "logos": [
        {
          "id": "01250b55-4a0c-49c8-8d88-94185e854e42",
          "name": "Google",
          "industry": "Technology"
        },
        {
          "id": "72975c3c-79f7-4ded-9bb8-7509131ea8af",
          "name": "Bank Of America",
          "industry": "Banking"
        }
      ]
    }
  }
}
```

This query retrieves a list of accounts available.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
limit | String | Limit the number of results returned by the query.
offset | String | Page offset of the results returned, used along with limit to paginate results.
order | [[String]] | Sort order of list query by a given field and sorting function("ASC" or "DESC"), eg: [["name", "ASC"]] sorts by name in ascending order
pattern | String | Search pattern, to use to filter accounts.
filters | [{ field_name: String, values: [String] }] | Filter the list by a set of fields, if multiple fields are specified will be or'ed between the fields. Note for filter values use the id's of choice or reference fields.


### Response Fields

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
logos | [Logo] | Array of Account Objects, request exact shape of contact fields needed for your integration.   
count | Int | The total count of accounts satisfied by the query ignoring the limit specified. 

## Account by ID 

```
logo(id: String!) 
Logo
```

> Sample Response

```json
{
  "data": {
    "logo": {
       "id": "01250b55-4a0c-49c8-8d88-94185e854e42",
       "name": "Google",
       "industry": "Technology"
    }
  }
}
```

This query retrieves a specific account by Id.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
id | String! | Required parameter, specifying the id of account to lookup.

## Custom Object List

```
getEntityValuesWithCount(
entity: String!,
limit: String,
offset: String,
order: [[String]],
pattern: String,
filters: [Filter]) 
{
   values: [faEntityValue]
   count: Int
}
```

> Sample Response

```json
{
  "data": {
    "getEntityValuesWithCount": {
      "count": 400,
      "values": [
        {
          "id": "25dbe1c9-503f-4594-af1c-026b2a7f255c",
          "seq_id": "QUO100005",
          "custom_fields": [
            {
              "field_name": "quote_field0",
              "value": "sample quote1"
            },
            {
              "field_name": "quote_field1",
              "value": "d30b32d7-089c-4426-b580-305094e31bb4"
            }
          ]
        },
        {
          "custom_fields": [
            {
              "field_name": "quote_field0",
              "value": "sample quote2"
            },
            {
              "field_name": "quote_field1",
              "value": "4a251647-cd6a-4ce6-a667-52b3efb30ea5"
            }
        }
      ]
    }
  }
}
```

This query retrieves a list of custom object records for a given custom entity.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
entity | String! | Name of the custom object, this is a required parameter, get the name of the custom object from Settings->Tabs under object column in the Tabs listing. 
limit | String | Limit the number of results returned by the query.
offset | String | Page offset of the results returned, used along with limit to paginate results.
order | [[String]] | Sort order of list query by a given field and sorting function("ASC" or "DESC"), eg: [["seq_id", "ASC"]] sorts by name in ascending order
pattern | String | Search pattern, to use to filter accounts.
filters | [{ field_name: String, values: [String] }] | Filter the list by a set of fields, if multiple fields are specified will be or'ed between the fields. Note for filter values use the id's of choice or reference fields.


### Response Fields

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
values | [faEntityValue] | Array of custom Objects data.   
count | Int | The total count of accounts satisfied by the query ignoring the limit specified. 

## Custom Object by ID 

```
getEntityValuesWithCount(entity: String!, id: String) 
{
   values: [faEntityValue]
}
```

> Sample Response

```json
{
  "data": {
    "getEntityValuesWithCount": {
      "values": [
        {
          "id": "25dbe1c9-503f-4594-af1c-026b2a7f255c",
          "seq_id": "QUO100005",
          "custom_fields": [
            {
              "field_name": "quote_field0",
              "value": "sample quote1"
            },
            {
              "field_name": "quote_field1",
              "value": "d30b32d7-089c-4426-b580-305094e31bb4"
            }
          ]
        }
      ]
    }
  }
}
```

This query retrieves a specific custom object record by Id.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
entity | String! | Name of the custom object, this is a required parameter, get the name of the custom object from Settings->Tabs under object column in the Tabs listing.
id | String! | Required parameter, specifying the id of account to lookup.


## Entities List

``` 
getEntities(show_hidden: Boolean) 
{
    id: String
    entity_id: String
    name: String
    label: String
    label_plural: String
    is_visible: Boolean
    is_custom: Boolean
}
```

> Sample Response

```json
{
  "data": {
    "getEntities": [
      {
        "id": "c42060c7-70fb-48ee-9c34-a3383dd64a00",
        "entity_id": "ac12096d-027b-57f5-b389-93c1920222a3",
        "name": "contact",
        "label": "Person",
        "label_plural": "People",
        "is_visible": true,
        "is_custom": false
      },
      {
        "id": "dca49832-c7b8-48ec-889c-e212414c6d29",
        "entity_id": "d72a990d-7bfa-55e7-9651-0b2b3889c311",
        "name": "logo",
        "label": "Account",
        "label_plural": "Accounts",
        "is_visible": true,
        "is_custom": false
      },
      {
        "id": "42851e2d-4c8d-446c-92ca-b8dc25fed39b",
        "entity_id": "9227e298-9074-5315-931f-c79e0126dc58",
        "name": "deal",
        "label": "Deal",
        "label_plural": "Deals",
        "is_visible": true,
        "is_custom": false
      },
      {
        "id": "820bb9d7-4226-4150-a59c-a78904491099",
        "entity_id": "b9c95a9b-b126-4e19-aca8-cb44a8b8387e",
        "name": "quote",
        "label": "Quote",
        "label_plural": "Quotes",
        "is_visible": true,
        "is_custom": true
      },
    ]
  }
}
```

This query retrieves a list of entities/objects configured, typically used to get the entity_id associated with Object.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
show_hidden | Boolean | To filter out hidden objects in the system or not, default is to list only active entities.


### Response Fields

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
id | String | Id of entity config.   
entity_id | String | Entity Id, note this is the ID to be used while creating Next Steps or posting activity on Objects. 
name | String | system friendly name, uses underscore format.
label | String | User configured label to indicate a single Object record.
label_plural | String |  User configured label to indicate a list of Object records.  
is_visible | Boolean | Indicates if the Object is visible or hidden in the system.
is_custom | Boolean | Indicates if this a user setup custom field or a system field.


## Entity Fields List

``` 
getFields(entity: String) 
{
    id: String
    name: String
    name_label: String
    render_type: String
    is_required: Boolean
    is_visible: Boolean
    is_custom: Boolean
}
```

> Sample Response

```json
{
  {
    "data": {
      "getFields": [
            {
              "id": "bb6b14f3-aa99-4253-9f4f-797e2d4ac697",
              "name": "full_name",
              "name_label": "Customized Name",
              "render_type": "profile",
              "is_required": null,
              "is_visible": true, 
              "is_custom": false
            },
            {
              "id": "094dd440-41a2-4643-93fe-f53fa1475000",
              "name": "first_name",
              "name_label": "contact.first_name",
              "render_type": "text",
              "is_required": true,
              "is_visible": true,
              "is_custom": false
            },
            {
              "id": "6c0d62a4-1f7e-4e5a-8e5d-3c22c7b649dc",
              "name": "contact_field7",
              "name_label": "Custom",
              "render_type": "reference",
              "is_required": null,
              "is_visible": true,
              "is_custom": true
            }
          ]
        }
      ]
    }
  }
}
```

This query retrieves a list of fields associated with a object, typically used to get field id or custom field name used for filtering or creating/updating the fields.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
entity | String | Name of the object(eg: contact/deal/logo etc), this is a required parameter, get the name of the object from Settings->Tabs under object column in the Tabs listing.


### Response Fields

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
id | String | Field config id use this to lookup choice values for a given field.   
name | String | field name given by the system, not changeable by users. 
name_label | String | User customized name label.   
render_type | String | The data type of the field, like text, date, boolean etc 
is_required | Boolean | Indicates if this is a required field  
is_custom | Boolean | Indicates if this a user setup custom field or a system field. 



## Field Choice List Values

```
getFieldItems(
  fa_field_config_id: String!,
  pattern: String,
  limit: Int
) 
{
  id: String
  name: String
}
```

> Sample Response

```json
{
  "data": {
    "getFieldItems": [
      {
        "id": "01250b55-4a0c-49c8-8d88-94185e854e42",
        "name": "Choice Value 1"
      },
      {
        "id": "72975c3c-79f7-4ded-9bb8-7509131ea8af",
        "name": "Choice Value 2"
      },
      {
        "id": "49a9a02b-94d9-46ca-b1af-e9b457914673",
        "name": "Choice Value 3"
      }
    ]
  }
}
```

This query retrieves choice list values for a given field config id.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
fa_field_config_id | String! | Id of the field, you can use getFields API to fetch the id of the field.
pattern | String | Search pattern, to use to filter choice list values.
limit | String | Limit the number of results returned by the query.


## Agents List

``` 
getTeamMembers(active: Boolean) 
{
      id: String,
      first_name: String,
      last_name: String,
      full_name: String,
      email_address: String
}
```

> Sample Response

```json
{
  "data": {
    "getTeamMembers": {
      "agents": [
        {
          "id": "67fabfb7-e774-440b-828b-d160f07012cc",
          "first_name": "Test",
          "last_name": "User1",
          "full_name": "Test User1",
          "email_address": "test1@demo.com",
          "status": "Active"
        },
        {
          "id": "c9b9b7d3-d6be-4555-a251-8c3d38915409",
          "first_name": "Test",
          "last_name": "User2",
          "full_name": "Test User2",
          "email_address": "test2@demo.com",
          "status": "Active"
        },
        {
          "id": "5168bfef-abd9-4d87-b683-b65209b4b7dd",
          "first_name": "Test",
          "last_name": "User3",
          "full_name": "Test User3",
          "email_address": "test3@demo.com",
          "status": "Suspended"
        }
      ]
    }
  }
}
```

This query retrieves a list of agents available in your Freeagent Account.

### GraphQL Parameters

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
active | Boolean | Retrieve only "Active" agents in the system if true, else returns all including inactive users.


### Response Fields

Parameter | GraphQL Type | Description
--------- | ------------ | -----------
id | String | Agent Id of the team member.   
first_name | String | First name of the agent. 
last_name | String | Last name of the agent.   
full_name | String | First and last name combined. 
email_address | String | Login email for the user.  
status | String | Status of the user "Active"/"Suspended" 

# Mutations


## Create Contact

```
addLead(
first_name: String,
last_name: String,
buying_power: Int,
normalised_title_id: String,
current_position: String,
logo_id: String,
location_id: String,
location_name: String,
birth_date: String,
portrait_url: String,
work_phone: String,
home_phone: String,
mobile_phone: String,
personal_email: String,
work_email: String,
alternate_email_1: String,
alternate_phone_1: String,
home_address: String,
work_address: String,
alternate_address_1: String,
linkedIn: String,
facebook: String,
twitter: String,
logo_name: String,
lead_score: Int,
lead_status_id: String,
lead_source_id: String,
override_buying_power: Boolean,
override_buying_center: Boolean,
buying_center_id: String,
lead_owner_id: String,
business_card_url: String,
industry_id: String,
custom_fields: [[String]],
channelId: String
): {
   lead: Contact
   error: String
}
```

## Update/Delete Contact

```
updateLeadFields(
recordId: String!,
values: [{
   field_name: String
   value: String
 }]!
): Contact
```

## Create Deal

```
addDeal(
name: String,
location_place_id: String,
location_name: String,
logoname: String,
logo_id: String,
win_probability: Int,
amount: Float,
close_date: Date,
sales_stage_id: String,
type_id: String,
lead_source_id: String,
forecast_category_id: String,
lead_source_detail: String,
owner_id: String,
custom_fields: [[String]],
status_id: String,
contactId: String
): Deal
```


## Update/Delete Deal

```
updateDealFields(
recordId: String!,
values: [{
  field_name: String
  value: String
}]!
): Deal
```

## Create Account

```
addLogo(
logo_src: String,
name: String,
hq_location: String,
industry_catalog_id: String,
website: String,
num_employees: String,
revenue: String,
summary: String,
business_phone: String,
owner_id: String,
custom_fields: [[String]]
): Logo
```

## Update/Delete Account

```
updateLogoFields(
  recordId: String!,
  values: [{
    field_name: String
    value: String
  }]!
): Logo
```


## Create Custom Object Record

```
addEntityValue(
entity: String!,
custom_fields: [[String]]
): faEntityValue
```

## Update/Delete Custom Object Record

```
updateEntityValueFields(
recordId: String!,
values: [{
  field_name: String
  value: String
}]!
): faEntityValue
```

## Create NextStep

```
addTask(
parent_entity_id: String,
parent_reference_id: String,
type: "Note",
description: String,
source_name: String,
): Task
```

## Post Activity

```
addTask(
parent_entity_id: String,
parent_reference_id: String,
type: "Note",
description: String,
source_name: String,
): Task
```

# GraphQL Objects

## Contact

```
type Contact {
id: String
first_name: String
last_name: String
full_name: String
portrait_url: String
current_position: String
birth_date: String
percent_completed: String
buying_power: Int
buying_center: String
buying_center_id: String
location: Location
location_id: String
logo_id: String
agent: agent
logo: Logo
source: String
mobile_phone: String
home_phone: String
work_phone: String
personal_email: String
work_email: String
alternate_email_1: String
alternate_phone_1: String
location_name: String
home_address: String
work_address: String
alternate_address_1: String
linkedIn: String
facebook: String
twitter: String
sources: [SyncAccountContact]
lead_status: String
lead_source: String
lead_status_catalog: catalog
lead_source_catalog: catalog
industry: catalog
lead_status_id: String
lead_source_id: String
lead_owner_id: String
industry_id: String
lead_score: Int
owner_id: String
lead_owner: agent
deal_role: String
deal_highlighted: Boolean
created_at: Date
updated_at: Date
tasks: [Task]
oldestDueTask: Task
custom_fields: [CustomFields]
attempts: Int
last_attempts: Date
last_activity: Date
outbound_attempts: Int
last_outbound_attempts: Date
email_addresses: [String]
contact_items: [contactItems]
}
```

## Deal

```
type Deal {
id: String
name: String
close_date: String
created_at: Date
updated_at: Date
amount: Float
deal_state: String
win_probability: Int
channel: Channel
owner_id: String
owner: agent
logos: [Logo]
logo: Logo
logo_id: String
location: Location
attachments: [Attachment]
tasks: [Task]
created_by: Profile
modified_id: String
sales_stage_id: String
sales_stage: salesStageCatalogQL
stage_changed_at: Date
weighted_forecast: Float
sales_cycle: Int
type_id: String
dtype_id: String
dtype: catalog
lead_source_id: String
lead_source: catalog
forecast_category_id: String
forecast_category: catalog
custom_fields: [CustomFields]
fiscal_period: String
oldestDueTask: Task
last_activity: Date
status: String
status_id: String
status_catalog: catalog
}
```

## Logo

```
type Logo {
id: String
name: String
logo_src: String
company_type: String
traded_as: String
predecessor: String
successor: String
founded: String
founder: String
defunct: String
hq_location: String
hq_location_country: String
area_served: String
key_people: String
industry: String
industry_catalog_id: String
industry_catalog: catalog
owner: agent
products: String
brands: String
revenue: String
operating_income: String
net_income: String
num_employees: String
subsid: String
rating: String
website: String
pageid: Int
revision: Int
summary: String
coordinates: String
logo_url: String
wikipediaurl: String
services: String
alias: String
issubsidiary: Boolean
parentid: String
parent: String
founded_date: String
type: String
chancellor: String
nickname: String
numstudents: String
endowment: String
universitytype: String
president: String
provost: String
undergraduates: String
description: String
postgraduates: String
contact_count: Int
deals_count: Int
follow: Boolean
favorite: Boolean
install_base: Boolean
territory: Boolean
agent_id: String
owner_id: String
team_id: String
status: String
g2krankings: [G2kranking]
contacts: [Contact]
locations: [Location]
deals: [Deal]
# Arguments
# order: [Not documented]
# limit: [Not documented]
# offset: [Not documented]
contacts_by_agent(order: [[String]], limit: String, offset: String): [Contact]
custom_fields: [CustomFields]
last_activity: Date
business_phone: String
oldestDueTask: Task
created_at: Date
}
```

## faEntityValue

```
type faEntityValue {
id: String
faEntity: faEntity
custom_fields: [CustomFields]
created_at: Date
updated_at: Date
created_by: Profile
updated_by: Profile
seq_id: String
last_activity: Date
oldestDueTask: Task
}
```

# Rate Limits

Integrations API is subject to the limit of 100 requests per 10 seconds.
Integrations exceeding either of those limits will receive error responses with a 429 response code.
