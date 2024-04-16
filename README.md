# Restful

This is a collection of some of the ideas I keep in mind while designing APIs and API endpoints.

## One API, In The Host

For a given company; we should have an API hosted on e.g. `api.mycompany.com`. This is the only token in the URL which is needed to designate that the given host is an API; put another way, a URL root like `https://api.mycompany.com/api/...` is unnecessary and stutters.

## Domain Objects In The Path

Every API for a given company should be hosted as paths underneath this host. For example, a single company may have both a Users-centric and Tenant-centric API, which may on the backend be served by two different services. These should both be on the `api.mycompany.com` host, like: `api.mycompany.com/tenants` and `api.mycompany.com/users`, rather than e.g. `api-users.mycompany.com`. 

## Environments and URLs

Less to do with APIs and more to do with setting up environments in general, but: If you have both a production and staging environment, the best way to configure your staging environment is via the registration of an entirely new domain name, such as having your website on `mycompany.com` and `mycompany-staging.com`, and API on `api.mycompany.com` and `api.mycompany-staging.com`, for production and staging respectively.

The second best way is to leverage the hierarchy intrinsic to DNS; `mycompany.com` hosts your website on `mycompany.com` and API on `api.mycompany.com`, while the staging environment lives on `staging.mycompany.com` and `api.staging.mycompany.com`.

These two methods are preferrable to a system that looks like e.g. `api.mycompany.com` and `api-staging.mycompany.com`.

## Versioning

TODO

## Methods and Meanings

There are five HTTP methods which your endpoints should use.

| Method   | Description                                          |
|----------|------------------------------------------------------|
| `GET`    | Retrieve either one resource, or a list of resources |   
| `POST`   | Create a resource                                    |
| `PUT`    | Replace the content of a resource                    |
| `PATCH`  | Partially update the content of a resource           |
| `DELETE` | Delete or soft-delete a resource                     |

## Alternate Nouns and IDs

To query for a list of users, you may leverage a method+path like: `GET /users`.

To query for a single user, you may leverage a method+path like `GET /users/:id`.

To query for a single user's roles, you may leverage a method+path like `GET /users/:id/roles`.

Paths should always be designed around a series of alternating nouns and IDs; a noun like 'users', then a user's ID, then attributes on that user, then IDs on those attributes, onward and onward.

## No Verbs

TODO

## No Uppercase

None of the nouns in your paths should contain uppercase characters; they should instead match the regex `[a-z0-9_]+`; lowercase characters, underscores, and I suppose if you really need them, digits. IDs may naturally contain uppercase and lowercase characters, which is reasonable and fine.

## Prefer Pluralization

The nouns in your paths should generally be pluralized.

- Good: `GET /users` and `GET /users/123`
- Bad: `GET /user` and `GET /user/123`
- Bad: `GET /users` and `GET /user/123`

The exception to this rule is when writing terminal path fragments which may refer to single fields on a resource. For example:

- `GET /users/123/roles` is entirely plural and makes sense, because a user can have multiple roles.
- `GET /users/123/birthday` mixes singular and plural, but still makes sense because a user may only have one birthday.

This example is contrived, and wouldn't exist in a real API, because most fields which make sense to be singular would make the most sense to simply be returned on the e.g. `GET /users/123` response object. But, it can happen (for example: imagine if the `birthday` field, in this example, often contains an extremely large amount of data). 

## Listing versus Fetching

When using a `GET` request, whether that request returns one resource or a list of resources depends both on where you're querying in the alternation between nouns and IDs, and what you're querying for.

- `GET /users` -> returns a list of users
- `GET /users/123` -> returns a single user
- `GET /users/123/roles` -> returns a list of roles
- `GET /users/123/birthday` -> returns a single birthday

## Querying and Filtering

The URL path itself should only contain globally unique IDs which uniquely select one resource from the list of all resources of that noun. For example:

- Good: `GET /users/123123` (assuming `123123` is a globally unique ID)
- Bad: `GET /users/enabled` (listing all enabled users? bad)

Any selector on a noun which is not globally unique is best thought of as a filter; and should be placed in the route's query parameters: for example, `GET /users?status=enabled`.

Filter query parameters should prefer matching keys to the response object of each resource; for example, if each user has a `{ "status": "enabled" }` field, the query parameter should read `?status=enabled`, not `?enabled=true`.

## Pagination

TODO

## Put versus Patch

TODO

## Status Codes

TODO

## Response Envelopes

asdf
