# Milkyway API

## Definition

The goal of this page is to define the high level architecture of the Milkyway API that will allow matching agencies (called here entities) using Milkyway network.


## **Populate endpoint**

>Shell command:

```shell
curl --header "Content-Type: application/json" \
 --header "Authorization: Bearer <ACCESS_TOKEN>"\
 --request PUT \
 --data '{"skills":["<SKILL_1>", "<SKILL_2>", "<SKILL_3>", "<SKILL_4>", "<SKILL_5>"]}' \
 http://milkyway.sortlist.com/v1/<ENVIRONMENT>/entities/<ENTITY_ID>
```
>Example of request:

```json
{ 
  "id" : "cluj-napoca-studio",
  "skills" : ["design", "frontend development", "angularJS", "reactJS", "html", "css", "web-development",] 
}
```


>The above command returns JSON structured like this:

```json
[
  {
   "id": "cluj-napoca-studio",
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

### Description
Endpoint specialized in adding a new agency (entity) in the Milkyway API's database, which will make the entity and the skills available for matching, auto-complete and auto-suggestion.
For each skill (input skill) from the list given in the request's body:

* Create a new entry (entry_id,environment, entity_skill) in table `mw_entity_skills`:
  1. **entity_id** : slug of the entity
  2. **environment** : staging/prod/dev
  3. **entity_skill** : is computed by replacing the whitespaces of an input skill with dash (as an internal normalization)
      * eg : input skill : `engine optimization` --> entity_skill : `engine_optimization`

This entry represents the **link between an entity and a input skill**.

* Create a new entry (entity_skill, milkyway_skill) (called translation) in table `mw_entity_skill_translation`:
  1. **entity_skill** : agency skill slug (input skill)
  2. **milkyway_skill** : is the best match of the input skill in the MW skills space. This is done using fuzzy matching.
      * `input skill`		<=>	 	`milkyway_skill`
      * (database relation :  many (*) to one (1) )


This entry represents the **link between a input skill and a milkyway skill**.

### Http request

`PUT http://milkyway.sortlist.com/v1/<ENVIRONMENT>/entities/<ENTITY_ID>`

### Query Parameters

Parameter | Necessary | Type | Description
--------- | -------- | ------- | -----------
**`id`** | `no` | `string` | `ID (slug) of the entity`
**`skills`** | `yes` | `list` | `List of skills saved for entity`

<aside class="warning">
Braer token needed for authentification!
</aside>
