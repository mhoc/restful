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

Its become something of a meme in the industry that every new API you write should start with `/v1`. 

My take is: Don't do this. URLs are hierarchical, and prefixing the entire API with a `/v1` communicates that, when a `/v2` happens (put another way: when you have to break an API), every API will be replaced. The far more common situation arises when you have to break a single API; and you _don't_ want an API surface that looks like:

- `GET /v1/users`
- `GET /v2/users/123/roles`

Put another way: "If you're listing users, go ahead and use the v1 endpoint, but if you're getting a user's roles use the v2 endpoint". That's weird.

The only situation where putting the API version in the URL makes sense is when writing an RPC style API. For example: `POST /list_users/v2` is a totally reasonable path. But, this document covers restful APIs, not RPC-style APIs, and thus I would assert that a version number should not appear in the path of your API.

Instead, consider implementing versioning as an HTTP header. Something simple like requiring an `X-VERSION: 1` header makes sense, or assuming `X-VERSION: 1` when a version header is not provided. Some popular APIs implement this as a date field, so the server can deduce which version of the API to serve based upon the date the client was written; e.g. `X-VERSION: 2024-04-04` may return the "v1" API, but `X-VERSION: 2024-05-05` may return the "v2" API. 

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

## Verbs and Edge Cases

Every word typed into the path of your URL should be a noun.

You'll naturally come across examples with every API where the thing you're trying to do just doesn't make sense as a noun. Here's one example: Your system has some kind of synchronization procedure, and you want to kick off a manual invocation of this sync. When thinking of this in terms of RPCs, the call might be: `invokeSync(syncId: string)`. In a restful API, this might be: `POST /syncs/:sync_id/invocations`. We're "creating an invocation"; even if that doesn't create a persistent _thing_ in the database, it still fits into the restful world as a noun. 

Now, let's say you need an API to cancel this manual invocation. Assumedly, that `POST` call actually did create something in a database, with a primary key; and you could now call `DELETE /syncs/:sync_id/invocations/:invocation_id` to cancel it. 

What if your API needs to support _both_ the actions of cancelling an in-progress sync, and deleting the existence of this sync invocation? How about something like:

- Cancel in in-progress sync: `POST /syncs/:sync_id/invocations/:invocation_id/cancellation`
- Delete the sync invocation entirely: `DELETE /syncs/:sync_id/invocations/:invocation_id`

My point in going several steps down this path is: There is always a solution to these "my thing kinda looks like a verb" problems. Sometimes those solutions stretch the meaning of a "noun", or require some creativity. It helps if you think through the design of the entire system always keeping the nouns in mind; for example, maybe you aren't "cancelling a sync invocation", you're "creating a cancellation request for a sync invocation". Design the entire system under the impression that the only verbs you have in your lexicon are "create" "read" "update" and "delete". 

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

Every API endpoint which list resources and does not paginate will break something, eventually. Don't take shortcuts.

Every resource listing endpoint should accept a `limit=N` query parameter. The implementation on the backend should enforce this, with a sensible default and a sensible maximum value. 

Beyond that, a more specific pagination schema is left to the implementation, but I generally prefer to keep it simple: `limit=N` and `offset=M`.

The APIs response should not be a bare array; it should return metadata concerning at minimum the number of items in the total set; for example:

```json
{
  "users": [],
  "total": 124
}
```

## Put versus Patch

The difference between `PUT` and `PATCH` is subtle, but strict.

- `PUT` replaces all updatable fields in the target resource with the provided object. E.g. `{ "name": "Mike" }` updating with `{}` becomes `{}`.
- `PATCH` replaces the content of only the specified fields with those in the provided object. E.g. `{ "name": "Mike" }` patching with `{}` does not change.

Observationally; most APIs need one or the other, and not both; and in fact, most APIs find more value in `PATCH` than `PUT`. But, as always, your mileage may vary.

## Array Fields

APIs should, generally, not publish array fields on resources.

- `GET /users/123` could return `{ "roles": ["admin"] }`, but this is not desirable.
- `GET /users/123/roles` is superior.

Unbounded arrays may grow to be very large relative to the overall response size; and there is no consistent and general solution to modifying the content of array fields.

- To append a new role to this field: `PATCH /users/123` with a body of `{ "roles": ["admin","superadmin"] }` would work, but it requires the caller to already know their current roles, which should not be necessary, and it introduces a race condition whereby other clients might have a different view of the user's pre-existing roles. `POST /users/123/roles` is more consistent.

- To update items in an array: Without each items having a primary key, the only solution is to `PATCH` the entire array. Thus, for the pre-stated reasons, the better solution is to ensure each item in the array has its own primary key, and thus treat each item as its own restful resource: `PATCH /users/123/roles/456`.

The exception to this rule enters when the order of items in the array field is material. When the ordering of items is material, then it becomes difficult to treat each item in the array as its own restful resource. You should ignore everything stated in this section and instead accept that any API you design will not be consistent nor convenient. 

## Patching and Deleting Fields

Imagine you have a schema for an object that resembles the following: `type User = { name?: string, /* ... */ }`.

- An API like `PUT /users/:id` would allow for deleting the name, but would require the caller to know everything else about the user in order to delete just their name.
- An API like `PATCH /users/:id` removes this need, but its non-obvious how to represent "delete the user's name" in the body.

Javascript plays well with providing a body like `{ "name": null }`; but as a general rule this is somewhat risky. Other languages and frameworks may be unable to consistently disambiguate between `{}` and `{ "name": null }`. 

A general, consistent solution is to instead write another API: `DELETE /users/:id/name`. 

A second solution is to borrow a page from MongoDB or Firestore's book, and build a kind of DSL into your API like: `PATCH /users/:id` with a body `{ "name": { $delete: true } }`. This might be preferrable if you have a large number of fields which need to support deleting.

## Status Codes

TODO

## Response Envelopes

asdf

## Errors

TODO
