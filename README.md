# spectral-dedup-issue

This repo contains a testcase that demonstrates a problem in
the de-duplication logic of spectral-core.

## Contents
- .spectral.yml - defines a custom rule with id `schema-description` that
verifies that each schema contains a description.
- api.yaml - An OpenAPI document that defines a couple of operations, along
with one schema that lacks a description.
- functions/schemaDescription.js - a custom function that implements the rule; this
is a bit non-trivial as the function will traverse all schemas within the document
that are reachable from the paths specified in the rule definition.  So the rule's
`given` field is the set of starting points for these traversals.

## To reproduce the issue:
- clone this repo and install spectral (`npm install -g @stoplight/spectral-cli`)
- cd to project root directory
- `spectral lint api.yaml`

## Results:
You should the following output:
```
$ spectral lint api.yaml
Visiting path: paths./v1/drinks.post.requestBody.content.application/json.schema
Visiting path: paths./v1/drinks.post.requestBody.content.application/json.schema.oneOf.0
Visiting path: paths./v1/drinks.post.requestBody.content.application/json.schema.oneOf.0.properties.type
Visiting path: paths./v1/drinks.post.requestBody.content.application/json.schema.oneOf.0.properties.fruit
Visiting path: paths./v1/drinks.post.requestBody.content.application/json.schema.oneOf.1
>> Violation at: paths./v1/drinks.post.requestBody.content.application/json.schema.oneOf.1
Visiting path: paths./v1/drinks.post.requestBody.content.application/json.schema.oneOf.1.properties.type
Visiting path: paths./v1/drinks.post.requestBody.content.application/json.schema.oneOf.1.properties.name
Visiting path: paths./v1/drinks.post.responses.201.content.application/json.schema
Visiting path: paths./v1/drinks.post.responses.201.content.application/json.schema.oneOf.0
Visiting path: paths./v1/drinks.post.responses.201.content.application/json.schema.oneOf.0.properties.type
Visiting path: paths./v1/drinks.post.responses.201.content.application/json.schema.oneOf.0.properties.fruit
Visiting path: paths./v1/drinks.post.responses.201.content.application/json.schema.oneOf.1
>> Violation at: paths./v1/drinks.post.responses.201.content.application/json.schema.oneOf.1
Visiting path: paths./v1/drinks.post.responses.201.content.application/json.schema.oneOf.1.properties.type
Visiting path: paths./v1/drinks.post.responses.201.content.application/json.schema.oneOf.1.properties.name
Visiting path: paths./v1/drinks.get.responses.201.content.application/json.schema
Visiting path: paths./v1/drinks.get.responses.201.content.application/json.schema.properties.drinks
Visiting path: paths./v1/drinks.get.responses.201.content.application/json.schema.properties.drinks.items
Visiting path: paths./v1/drinks.get.responses.201.content.application/json.schema.properties.drinks.items.oneOf.0
Visiting path: paths./v1/drinks.get.responses.201.content.application/json.schema.properties.drinks.items.oneOf.0.properties.type
Visiting path: paths./v1/drinks.get.responses.201.content.application/json.schema.properties.drinks.items.oneOf.0.properties.fruit
Visiting path: paths./v1/drinks.get.responses.201.content.application/json.schema.properties.drinks.items.oneOf.1
>> Violation at: paths./v1/drinks.get.responses.201.content.application/json.schema.properties.drinks.items.oneOf.1
Visiting path: paths./v1/drinks.get.responses.201.content.application/json.schema.properties.drinks.items.oneOf.1.properties.type
Visiting path: paths./v1/drinks.get.responses.201.content.application/json.schema.properties.drinks.items.oneOf.1.properties.name

/home/padams/work/test/spectral-dedup-issue/api.yaml
 62:11  error  schema-description  Schema must have a non-empty description.  components.schemas.Drink.oneOf[1]
 75:17  error  schema-description  Schema must have a non-empty description.  components.schemas.Drinks.properties.drinks.items

✖ 2 problems (2 errors, 0 warnings, 0 infos, 0 hints)
```

The custom rule function will log each individual path that it is visiting and will also
log the path associated with each specific violation that it finds.
Based on the above output, you can see three distinct violations, where all of these violations
should be mapped back to the path `components.schemas.Soda`, since that is the schema that is
lacking the description.
It seems that the de-duplication logic is tripping over the fact that the `Soda` schema
is referenced within a `oneOf` list.
