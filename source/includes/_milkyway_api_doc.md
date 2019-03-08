# Milkyway API

## Definition

The goal of this page is to define the high level architecture of the Milkyway API that will allow matching agencies (called here entities) using Milkyway network.


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

**`PUT http://milkyway.sortlist.com/v1/<ENVIRONMENT>/entities/<ENTITY_ID>`**

### Path Parameters

Parameter | Necessary | Type | Description
--------- | -------- | ------- | -----------
**`<ENTITY_ID>`** | `yes` | `string` | `ID (slug) of the entity`
**`<ENVIRONMENT>`** | `yes` | `string` | `Environment in which you want to store data staging/prod`

### Query Parameters

**`This endpoint does not use query parameters`**

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

## Errors
The Milkyway API uses the following error codes:

Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is invalid.
401 | Unauthorized -- Your API key is wrong.
403 | Forbidden -- The kitten requested is hidden for administrators only.
404 | Not Found -- The specified kitten could not be found.
405 | Method Not Allowed -- You tried to access a kitten with an invalid method.
406 | Not Acceptable -- You requested a format that isn't json.
410 | Gone -- The kitten requested has been removed from our servers.
418 | I'm a teapot.
429 | Too Many Requests -- You're requesting too many kittens! Slow down!
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.
