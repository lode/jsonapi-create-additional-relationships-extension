# JSON:API create additional relationships extension

This extension for [JSON:API](https://jsonapi.org/) allows to create related resources when creating (or updating) a primary resource.

You can use this extension with the namespace `createAdditional`,
and the URI `https://github.com/lode/jsonapi-create-additional-relationships-extension`.

- [Why?](#why)
- [Examples](#examples)
- [Rules](#rules)
- [Related](#related)


## Why?

In the JSON:API base specification you can only create a single resource at the time.
Any relationships to the resource being created already need to exist.
When you need to create both primary resource and relationship resource, you need to make extra API calls.
When both resources require each other to validate, this is impossible.


## Examples

### Creating

Here's how to create a primary resource and a related resource in one call.

`POST /persons`:

```json
{
	"data": {
		"type": "Person",
		"attributes": {
			"name": "Zaphod Beeblebrox"
		},
		"createAdditional:relationships": {
			"starship": {
				"data": {
					"type": "Starship",
					"attributes": {
						"name": "Heart of Gold"
					}
				}
			}
		}
	}
}
```

### Updating

Similarly, you can also create related resources when updating a primary resource in one call.

`PATCH /persons/1`:

```json
{
	"data": {
		"type": "Person",
		"id": "1",
		"createAdditional:relationships": {
			"semiHalfCousin": {
				"data": {
					"type": "Person",
					"attributes": {
						"name": "Ford Perfect"
					}
				}
			}
		}
	}
}
```

### One-to-many relationships

You can also create (some) resources in one-to-many relationships.
Here one related resource (Head 1) is already existing, the other needs to be created.

`POST /persons`:

```json
{
	"data": {
		"type": "Person",
		"attributes": {
			"name": "Zaphod Beeblebrox"
		},
		"createAdditional:relationships": {
			"heads": [
				{
					"data": {
						"type": "Head",
						"id": "1"
					}
				},
				{
					"data": {
						"type": "Head",
						"attributes": {
							"position": "below"
						}
					}
				},
			]
		}
	}
}
```

> Note: if any other resources were currently related as `heads` to this primary resource,
they would be removed, since that's how [regular PATCH calls](https://jsonapi.org/format/#crud-updating-resource-relationships) work.
If you don't want that, you can use [regular calls to update a one-to-many relationship](https://jsonapi.org/format/#crud-updating-to-many-relationships).


## Rules

### Client requests

A client applying this extension **MUST** use `createAdditional:relationships` as a member of the primary data.

A client **MUST**, for the values in `createAdditional:relationships`, follow the rules for
[`relationships` in the JSON:API base specification](https://jsonapi.org/format/#document-resource-object-relationships).
With one exception, next to resource identifier objects it can use resource objects for related resources that don't exist yet.

A client **MUST** use [resource objects](https://jsonapi.org/format/#document-resource-objects)
as value for relationships that don't exist yet.

For these resource objects, a client **MUST NOT** use an `id` which is known to the server, but **MAY** use a client-generated `id`.
A client **MAY** use a local-id (`lid`) if that is needed for other extensions or profiles, however it is not used by this extension.

A client **MAY** combine resource identifier objects and resource objects in one document or one one-to-many relationship.

### Server processing

A server accepting this extension **MUST** process `createAdditional:relationships` in the same way as the regular `relationships`.

A server **MUST** assume that relationships inside `createAdditional:relationships` without an `id`,
or with an `id` unknown to the server (e.g. an client-generated `id`),
are resources that don't exist yet and create new resources for those.

A server **MUST** assume that relationships inside `createAdditional:relationships` with an `id` known to the server,
are existing resources and not create new resources for those.

A server **MUST NOT** create any resource if one or more of the resources that need to be created can't be created.

> Note: a server may decide to create these resources in any order logical to the server.

### Server responses

A server **MUST** return a `201 Created` also if it makes changes to any of the additional related resources, e.g. assigning an `id`.

A server **SHOULD** return the additionally created resources in the `included` data, unless an `include` query parameter prevents this.

A server **MAY** return a `403 Forbidden` if it doesn't support creating additional resources for the requested primary data or relationship.

### Everything else

This extension uses the rules of the JSON:API base specification as much as possible.
This allows for better compatibility with other extensions and profiles and makes it easier to implement.

For all rules unspecified, the JSON:API base specification **MUST** be followed. Thus:

- A client **MUST** use an `id` known to the server for resource identifier objects inside `createAdditional:relationships`.
- A client **MAY** also use the regular `relationships` member and a server **MAY** decide to not support that.
- A server **MUST** connect all relationships inside `createAdditional:relationships` to the resource in the primary data.
- A server **MUST** respond to these requests as specified in the JSON:API base specification.


## Related

- The bulk create extension: https://github.com/jelhan/json-api-bulk-create-extension
- The atomic operations extension: https://jsonapi.org/ext/atomic/
- A php server implementation for jsonapi: https://github.com/lode/jsonapi
