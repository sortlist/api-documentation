# Milkyway API

## Definition

The goal of this page is to define the high level architecture of the Milkyway API that will allow matching agencies (called here entities) using Milkyway network.

<!-- POPULATE ENDPOINT DOC -->
## **Populate endpoint** 
**Create new entity with skills on a specific environment**

<aside class="warning">
Bearer token needed for authentification!
</aside>

>Shell command:

```shell
curl --header "Content-Type: application/json" \
 --header "Authorization: Bearer <ACCESS_TOKEN>"\
 --request PUT \
 --data '{"skills":["design", "frontend development", "angularJS", "reactJS", "html", "css", "web-development"]}' \
 http://milkyway.sortlist.com/v1/<environment>/entities/<entity-id>
```
>The above command returns JSON structured like this:

```shell
[
  {
   "id": "entity-id",
   "skills": [
       "design",
       "frontend-development",
       "angularJS",
       "reactJS",
       "html",
       "css",
       "web-development"
    ]
  }
]
```

### Http Request

`PUT http://milkyway.sortlist.com/v1/`**`environment`**`/entities/`**`entity_id`**

### Path Parameters

Parameter | Necessary | Type | Description
--------- | -------- | ------- | -----------
**`environment`** | `yes` | `string` | `Environment in which you want to store data staging/prod`
**`entity_id`** | `yes` | `string` | `ID (slug) of the entity`

### Query Parameters

**`This endpoint does not use query parameters`**

### Arguments

Argument | Necessary | Type | Description
--------- | -------- | ------- | -----------
**`id`** | `no` | `string` | `ID (slug) of the entity`
**`skills`** | `yes` | `list(string)` | `List of entity's skills`
### Description
Endpoint specialized in adding a new agency (entity) in the Milkyway API's database, which will make the entity and the skills available for matching, auto-complete and auto-suggestion.  
For each skill (input skill) from the list given in the request's body:

* Create a new entry (entry_id, environment, entity_skill) in table `mw_entity_skills`:
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
 http://milkyway.sortlist.com/v1/<environment>/matches?skills[]=<skill_1>&skills[]=<skill_2>
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

<!-- * Retrieve network skeleton from MW using list of N input skills (query params)  

* Fulfil the network skeleton with entities.Considering the list of N input skills, MW will return a skeleton:

* For each entity skill returns the 
  1. **entity_skill** : agency skill slug (input skill)
  2. **milkyway_skill** : is the best match of the input skill in the MW skills space. This is done using fuzzy matching.
      * `entity skill`		<=>	 	`milkyway_skill`
      * (database relation :  many (*) to one (1) )


This entry represents the **link between a entity skill and a milkyway skill**. -->

