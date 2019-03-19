# Milkyway API

Help businesses and entities manage skills, suggestions and matches in one place. Milkyway API's can help manage multiple skills on multiple environments. You can also automate creating entities and link them to the Milkyway skills space enabling them to be used for matching.

In Milkyway you connect entities with skills and perform actions like:
  - Insert/Update an entity into Milkyway (link entity to skills space) | Once inserted, entity will be used for matching algorithm.
  - Get suggestions of related skills for speciefied skills | Using complex network.
  - Get autcomplete list of skills for specified prefix.
  - Get matching entities for specified skills | Using complex network.

## Definition

The goal of this page is to define the high level architecture of the Milkyway API that you can interact with.

## Prerequisites

To be defined...

<!-- POPULATE ENDPOINT DOC -->
For example, insert or update a new Entity with skills on a specific environment with a PUT:
## **Populate endpoint**


<aside class="warning">
Bearer token needed for authentification!
</aside>

>Shell command:

```shell
curl --header "Content-Type: application/json" \
 --header "Authorization: Bearer <ACCESS_TOKEN>"\
 --request PUT \
 --data '{"skills": ["<SKILL_1>", "<SKILL_2>", "<SKILL_3>", "<SKILL_4>", "<SKILL_5>", "<SKILL_6>", "<SKILL_7>"]}' \
 https://milkyway.sortlist.com/v1/<ENVIRONMENT>/entities/<ENTITY_ID>
```
>The above command returns JSON structured like this:

```shell
[
  {
   "id": "entity-id",
   "skills": [
       "<SKILL_1>",
       "<SKILL_2>",
       "<SKILL_3>",
       "<SKILL_4>",
       "<SKILL_5>",
       "<SKILL_6>",
       "<SKILL_7>"
    ]
  }
]
```
All the skills that do not have a value or whose length is larger than 100 characters will be ignored.

### Http Request

`PUT http://milkyway.sortlist.com/v1/`**`environment`**`/entities/`**`entity_id`**

### Path Parameters

Parameter | Necessary | Type | Description
--------- | -------- | ------- | -----------
**`environment`** | `yes` | `string` | `Environment in which you want to store data staging/prod`
**`entity_id`** | `yes` | `string` | `ID (slug) of the entity`

### Query Parameters

**`This endpoint does not use query parameters`**

### Body Arguments

Argument | Necessary | Type | Description
--------- | -------- | ------- | -----------
**`id`** | `no` | `string` | `ID (slug) of the entity`
**`skills`** | `yes` | `list(string)` | `List of entity's skills`
### Description
Endpoint specialized in adding a new agency (entity) in the Milkyway API's database, which will make the entity and the skills available for matching, auto-complete and auto-suggestion.
For each skill (input skill) from the list given in the request's body:

* Create a new entry `(entity_id, environment, entity_skill)` in table `mw_entity_skills`:
  1. **entity_id** : slug of the entity
  2. **environment** : staging/prod/dev
  3. **entity_skill** : is computed by replacing the whitespaces of an input skill with dash (as an internal normalization)
      * eg : input skill : `engine optimization` --> entity_skill : `engine_optimization`

This entry represents the **link between an entity and its entity skill**.

* Create a new entry (entity_skill, milkyway_skill) (called translation) in table `mw_entity_skill_translation`:
  1. **entity_skill** : agency skill slug (input skill)
  2. **milkyway_skill** : is the best match of the input skill in the MW skills space. This is done using fuzzy matching.
      * `entity skill`		<=>	 	`milkyway_skill`
      * (database relation :  many (*) to one (1) )


This entry represents the **link between a entity skill and a milkyway skill**.

<!-- MATCHING ENDPOINT DOC -->
## **Matching endpoint**

<aside class="warning">
Bearer token needed for authentification!
</aside>

>Shell command:

```shell
curl --header "Content-Type: application/json" \
 --header "Authorization: Bearer <ACCESS_TOKEN>"\
 --request GET \
 https://milkyway.sortlist.com/v1/<ENVIRONMENT>/matches?skills[]=<SKILL_1>&skills[]=<SKILL_2>
```
>The above command returns JSON structured like this:

```shell
[
  {
      "entity_id": "echoua",
      "score": 0.32
  },
  {
      "entity_id": "mtd-creative-designs",
      "score": 0.16
  }
]
```

### Http Request

`GET http://milkyway.sortlist.com/v1/`**`environment`**`/matches?`**`skills[]`**`=<skill_1>&`**`skills[]`**`=<skill_2>`

### Path Parameters

Parameter | Necessary | Type | Description
--------- | -------- | ------- | -----------
**`environment`** | `yes` | `string` | `Environment in which you want to store data staging/prod`

### Query Parameters

Query parameter | Necessary | Type | Description
--------- | -------- | ------- | -----------
**`skills[]`** | `yes` | `string` | `Input skills`


### Description
Endpoint specialized in matching input skills with entities that exists in Milkyway API™ DB. A list of entities will be returned containing entity’s id and a matching score.


**Steps:**

* Retrieve a skeleton from MW using list of N input skills (query params)

> SKELETON STRUCTURE

```shell
[
  <PARENT NODE NAME FOR SKILL_1>: {
    'level_1': [
      { 'name': <milkyway skill>
        'seachable_name': <milkyway skill>
        'agencies': []
      }
      ...
    ],
    'level_2': [
      { 'name': <milkyway skill>
        'seachable_name': <milkyway skill>
        'agencies': []
      }
      ...
    ]
  }

  ...

  <PARENT NODE NAME FOR SKILL_N>:{
    'level_1': [
      { 'name': <milkyway skill>
        'seachable_name': <milkyway skill>
        'agencies': []
      }
      ...
    ],
    'level_2': [
      { 'name': <milkyway skill>
        'seachable_name': <milkyway skill>
        'agencies': []
      }
      ...
    ]
  }
]
```

* The skeleton represents the relevant skills space for the input skills.
    1. On the first level (`level_1`), there are milkyway skills that could represent an exact match to the input skill.
    2. On the second level (`level_2`), there are closest milkyway skills in the skills space related to the input skill.
<br>
* As the network does not know the entities that have one skill or another, skeleton needs to be fulfilled with entities. For each milkyway skill in the skeleton, entities will be retrieved from milkyway database.
    1. With one milkyway skill, more entity skills will be retrieved using db translations. Then, for each entity skill, entities will be retrieved and added to the skeleton.

<!-- AUTOCOMPLETE ENDPOINT DOC -->
## **Autocomplete endpoint**

<aside class="warning">
Bearer token needed for authentification!
</aside>


>Shell command:

```shell
curl --header "Content-Type: application/json" \
 --header "Authorization: Bearer <ACCESS_TOKEN>"\
 --request GET \
 https://milkyway.sortlist.com/v1/<ENVIRONMENT>/skills?prefix=<PREFIX>&limit=<LIMIT>
```
>The above command returns JSON structured like this:

```shell
[
  {
   "skills": [
       "branding",
       "brand awareness",
       "branding identity",
       "brand design",
       "brand identity design",
       "brand guidelines",
       "brand development",
       "browser window",
       "breakpoint",
       "broadcast production"
   ]
  }
]
```

### Http Request

`GET https://milkyway.sortlist.com/v1/`**`environment`**`/skills?`**`prefix`**`=<prefix>&`**`limit`**`=<limit>`

### Path Parameters

Parameter | Necessary | Type | Description
--------- | -------- | ------- | -----------
**`environment`** | `yes` | `string` | `Environment in which you want to store data staging/prod`

### Query Parameters

Query parameter | Necessary | Default value | Type | Description |
--------- | -------- | ------------- | ------- | -----------
**`prefix`** | `yes` | `-` | `string` | `Prefix string for searched skills.`
**`limit`** | `no` | `10` | `int` | `Number of skills retrieved.`

### Description

Endpoint specialized in returning milkyway skills starting with a prefix.
All the skill nodes will be retrieved from MW and only those starting with the specified query param prefix will be returned.


<!-- AUTOSUGGESTENDPOINT DOC -->
## **Autosuggest endpoint**

<aside class="warning">
Bearer token needed for authentification!
</aside>


>Shell command:

```shell
curl --header "Content-Type: application/json" \
 --header "Authorization: Bearer <ACCESS_TOKEN>"\
 --request GET \
 https://milkyway.sortlist.com/v1/<ENVIRONMENT>/suggestions?skills[]=<SKILL_1>&skills[]=<SKILL_2>&limit=10
```
>The above command returns JSON structured like this:

```shell
[
  {
   "skills": [
       "graphic",
       "javascript",
       "typography",
       "framework",
       "architecture",
       "application programming interface",
       "design process",
       "cascading style sheets",
       "aesthetic",
       "client side"
   ]
  }
]
```

### Http Request

`GET https://milkyway.sortlist.com/v1/`**`environment`**`/suggestions?`**`skills[]`**`=<skill_1>&`**`skills[]`**`=<skill_2>`**`limit`**`=<limit>`

### Path Parameters

Parameter | Necessary | Type | Description
--------- | -------- | ------- | -----------
**`environment`** | `yes` | `string` | `Environment in which you want to store data staging/prod`

### Query Parameters

Query parameter | Necessary | Default value | Type | Description |
--------- | -------- | ------------- | ------- | -----------
**`skills[]`** | `yes` | `10` | `string` | `Input skills`
**`limit`** | `no` | `10` | `int` | `Prefix string for searched skills.`

### Description

Endpoint specialized in returning a list of suggested skills based on input skills.
Based on an input list of skills, a list of related skills (suggestions) will be retrieved from MW.
