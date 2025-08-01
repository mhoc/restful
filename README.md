This is a collection of some of the ideas I keep in mind while designing APIs and API endpoints.

## What Is REST?

Nearly zero APIs which purport to be "REST" APIs or "RESTful" do a remotely adequate job of conforming to the original design ideals of REST as outlined by Roy Fielding in the early '90s. This document is no different. This document is aligned with the industrialized perversion of the idea of REST APIs more-so than any idealized concept few have actually built. 

## One API, Preferably In The Host

The word "API" should appear once in any URL your server hosts.

- `api.mycompany.com` is the most preferrable and common; a subdomain on the overall site.
- `mycompany.com/api` as a path prefix also makes sense, in situations where we want the API to share the same root domain as e.g. the website.
- `api.mycompany.com/api` unnecessarily stutters and should be avoided.

## Abstract Implementation

Every API for a given company should be hosted as paths underneath this host, regardless of how the APIs are implemented.

For example, a single company may have both User-centric and Tenant-centric APIs, which may be served by two different logical services. These should both be on the `api.mycompany.com` host, like: `api.mycompany.com/tenants` and `api.mycompany.com/users`, rather than e.g. `api-users.mycompany.com`.

## Environments and URLs

Less to do with APIs and more to do with setting up environments in general, but: If you have both a production and staging environment, the best way to configure your staging environment is via the registration of an entirely new domain name, such as having your website on `mycompany.com` and `mycompany-staging.com`, and API on `api.mycompany.com` and `api.mycompany-staging.com`, for production and staging respectively.

The second best way is to leverage the hierarchy intrinsic to DNS; `mycompany.com` hosts your website on `mycompany.com` and API on `api.mycompany.com`, while the staging environment lives on `staging.mycompany.com` and `api.staging.mycompany.com`.

## Versioning

Its become something of a meme in the industry that every new API you write should start with `/v1`.

My take is: Don't do this. URLs are hierarchical, and prefixing the entire API with a `/v1` communicates that when a `/v2` happens every API will be replaced. The issue is, we don't usually break an entire API all at once; we break and are forced to version individual endpoints. You don't want an API surface that looks like:

- `GET /v1/users`
- `GET /v2/users/123/roles`

This communicates to your users that they should use the v1 endpoint to list users, but the v2 endpoint to list a user's roles. This is weird, and does not read well.

One situation where putting the API version in the URL may make sense is when writing an RPC-style API. For example: `POST /list_users/v2` is somewhat reasonable. But, this document covers restful APIs, not RPC-style APIs. Additionally; some developer tooling, such as the Chrome dev console, will display the "Name" of a network request as the last path segment; and having a bunch of `v2`s appear there isn't very productive. Ultimately, I wouldn't even do this.

Instead, consider implementing versioning as an HTTP header. Something simple like requiring a `X-VERSION: 1` header can make sense, or assuming `X-VERSION: 1` when a version header is not provided. Some popular APIs implement this as a date field, so the server can deduce which version of the API to serve based upon the date the client was written; e.g. `X-VERSION: 2024-04-04` may return the "v1" API, but `X-VERSION: 2024-05-05` may return the "v2" API.

## Methods and Meanings

There are six HTTP methods which your endpoints should use.

| Method   | Description                                          |
| -------- | ---------------------------------------------------- |
| `HEAD`   | Return only whether a given resource exists or not.  |
| `GET`    | Retrieve either one resource, or a list of resources |
| `POST`   | Create a resource                                    |
| `PUT`    | Replace the content of a resource                    |
| `PATCH`  | Partially update the content of a resource           |
| `DELETE` | Delete or soft-delete a resource                     |

## Alternate Nouns and IDs

To query for a list of users, you may leverage a method+path like: `GET /users`.

To query for a single user, you may leverage a method+path like `GET /users/:id`.

To query for a single user's roles, you may leverage a method+path like `GET /users/:id/roles`.

Paths should always be designed around a series of alternating nouns and IDs; a noun like 'users', then a user's ID, then attributes on that user, then IDs on those attributes, onward and onward.

## Verbs and Edge Cases

Every word typed into the path of your URL should be a noun.

You'll naturally come across examples with every API where the thing you're trying to do just doesn't make sense as a noun. Here's one example: Your system has some kind of synchronization procedure, and you want to kick off a manual invocation of this sync. When thinking of this in terms of RPCs, the call might be: `invokeSync(syncId: string)`. In a restful API, this might be: `POST /syncs/:sync_id/invocations`. We're "creating an invocation"; even if that doesn't create a persistent _thing_ in the database, it still fits into the restful world as a noun.

Now, let's say you need an API to cancel this manual invocation. Assumedly, that `POST` call actually did create something in a database, with a primary key; and you could now call `DELETE /syncs/:sync_id/invocations/:invocation_id` to cancel it.

What if your API needs to support _both_ the actions of cancelling an in-progress sync, and deleting the existence of this sync invocation? How about something like:

- Cancel in in-progress sync: `POST /syncs/:sync_id/invocations/:invocation_id/cancellations`
- Delete the sync invocation entirely: `DELETE /syncs/:sync_id/invocations/:invocation_id`

My point in going several steps down this path is: There is always a solution to these "my thing kinda looks like a verb" problems. Sometimes those solutions stretch the meaning of a "noun", or require some creativity. It helps if you think through the design of the entire system always keeping the nouns in mind; for example, maybe you aren't "cancelling a sync invocation", you're "creating a cancellation request for a sync invocation". Design the entire system under the impression that the only verbs you have in your lexicon are "create" "read" "update" and "delete".

## No Uppercase

None of the nouns in your paths should contain uppercase characters; they should match the regex `[a-z0-9_]+`; lowercase characters, underscores, and I suppose if you really need them, digits. IDs may naturally contain uppercase and lowercase characters, which is reasonable and fine.

The reason for this is mostly just consistency. It was once stated to me that some older devices which may connect to websites or APIs struggle with case sensitivity, so all-lowercase has wider support; this feels like a rare edge case, but given it doesn't _really_ matter and consistency is good, I follow all-lowercase.

This rule applies equally to query parameters.

## Prefer Pluralization

The nouns in your paths should generally be pluralized.

- Good: `GET /users` and `GET /users/123`
- Bad: `GET /user` and `GET /user/123`
- Bad: `GET /users` and `GET /user/123`

The exception to this rule is for terminal path fragments, which may refer to single fields on a resource. For example:

- `GET /users/123/roles` makes sense, because a user can have multiple roles.
- `GET /users/123/birthday` terminates with a singular noun, but still makes sense because a user may only have one birthday.

This example is contrived, and wouldn't exist in a real API, because most fields which make sense to be singular would make the most sense to simply be returned on the e.g. `GET /users/123` response object. But, it can happen; for example: imagine if the `birthday` field, in this example, often contains an extremely large amount of data such that we don't want to bundle it into the overall `user` object for performance reasons.

## Listing versus Fetching

When using a `GET` request, whether that request returns one resource or a list of resources depends both on where you're querying in the alternation between nouns and IDs, and what you're querying for.

- `GET /users` -> returns a list of users
- `GET /users/123` -> returns a single user
- `GET /users/123/roles` -> returns a list of roles
- `GET /users/123/birthday` -> returns a single birthday

## Querying and Filtering

The URL path itself should only contain globally unique IDs which uniquely select one resource from the list of all resources of that noun. For example:

- Good: `GET /users/123123` (assuming `123123` is a globally unique ID)
- Bad: `GET /users/enabled` (listing all enabled users? bad)

Any selector on a noun which is not globally unique is best thought of as a filter; and should be placed in the route's query parameters: for example, `GET /users?status=enabled`. Filter parameters should never appear in a `GET` request's body, as GET requests should have no body.

Filter query parameters should prefer matching keys to the response object of each resource; for example, if each user has a `{ "status": "enabled" }` field, the query parameter should read `?status=enabled`, not `?enabled=true`. You should not prefix filterable fields with the word `filter`; for example, `status=enabled` is preferable to `filterStatus=enabled`.

Filter parameters can very quickly become more complex than simple string-to-string equivalence; for example, how would you represent a parameter that answers "return all users who are not enabled" (`status != enabled`)? I don't feel its appropriate to answer this here, but suffice to say that its a problem common and weird enough that it has caused many alternatives to REST to be invented. One opinion I have in this domain is: You shouldn't prematurely optimize your entire system when faced with this reality. A solution that looks like, for example, `status_neq=enabled` is preferrable to going all in with, I don't know, supporting writing SQL in your query parameters.

## Dynamic Psuedo-IDs

Imagine an API that queries for, as a contrived example, the commits in a git repo; like `GET /repos/:id/commits`. The corresponding API to fetch information for a single commit would be: `GET /repos/:id/commits/:hash`. Let's say we wanted to extrend this API surface to allow clients to fetch commit information for the latest commit in a repository; how might we do this?

One agreeable way to think about this is to embed a concept of pseudo-IDs in this endpoint; in other words, a special value for `:hash` like `latest`. Clients may then query `GET /repos/:id/commits/latest`; `latest` acts like an ID in that it globally uniquely identifies one commit. Because commit hashes are usually encoded in hexadecimal, and the word `latest` is not a valid hex representation, there is no chance of collision between the word `latest` and a valid commit ID. Your system can easily distinguish between these two values, and it creates a more productive API for clients.

Another valid way to do this would be to simply encode this as a filter on the listing commits endpoint; e.g. `GET /repos/:id/commits?order_by=date&order_dir=desc&limit=1`. 

## Pagination

Every API endpoint which list resources and does not paginate will eventually break. Don't take shortcuts.

Every resource listing endpoint should accept a `limit=N` query parameter. The implementation on the backend should enforce this, with a sensible default and a sensible maximum value.

Beyond that, a more specific pagination schema is left to the implementation, but I generally prefer to keep it simple: `limit=N` and `offset=M`.

The APIs response should return metadata concerning at minimum the number of items in the total set. My preferred method of doing this is to respond with the array of objects as the entire response, but provide this metadata in the response headers e.g.:

```
200 OK
X-TOTAL: 598
[
  { "id": "12345", "name": "Mike" },
  { "id": "54321", "name": "Bob" }
]
```

You may also elect to use a response envelope, which is fine. More will be written on response envelopes later.

```js
{
  "users": [ /* ... */ ],
  "total": 598
}
```

## Put versus Patch

The difference between `PUT` and `PATCH` is subtle, but strict.

- `PUT` replaces all updatable fields in the target resource with the provided object.
- `PATCH` replaces the content of only the specified fields with those in the provided object.

One example: Given an existing object like `{ "frstName": "Cave" }` and an update object like `{}`:

- `PUT` would result in a final object `{}`
- `PATCH` would result in a final object `{ "firstName": "Cave" }`.

Second example: Given an existing object like `{"lastName": "Johnson" }` and an update object like `{ "age": 56, "active": false }`:

- `PUT` would result in a final object `{ "age": 56, "active": false }`.
- `PATCH` would result in a final object `{ "lastName": "Johnson", "age": 56, "active": false }`.

Observationally: While you may choose to invest time in building both, most APIs will get more mileage preferring `PATCH` over `PUT`.

## Array Fields

APIs should, generally, not publish array fields on resources.

- `GET /users/123` could return `{ "roles": ["admin"] }`, but this is not desirable.
- `GET /users/123/roles` is superior.

Unbounded arrays may grow to be very large relative to the overall response size; and there is no consistent and general solution to modifying the content of array fields.

- To append a new role to this field: `PATCH /users/123` with a body of `{ "roles": ["admin","superadmin"] }` would work, but it requires the caller to already know their current roles, which should not be necessary, and it introduces a race condition whereby other clients might have a different view of the user's pre-existing roles. `POST /users/123/roles` is more consistent.
    
- To update items in an array: Without each items having a primary key, the only solution is to `PATCH` the entire array. Thus, for the pre-stated reasons, the better solution is to ensure each item in the array has its own primary key, and thus treat each item as its own restful resource: `PATCH /users/123/roles/456`.
    
The exception to this rule enters when the order of items in the array field is material. When the ordering of items is material, then it becomes difficult to treat each item in the array as its own restful resource. You should ignore everything stated in this section and instead accept that any API you design will not be consistent nor convenient.

## Patching and Deleting Fields

Imagine you have a schema for an object that resembles the following: `type User = { name?: string, /* ... */ }`.

- An API like `PUT /users/:id` would allow for deleting the name, but would require the caller to know everything else about the user in order to delete just their name.
- An API like `PATCH /users/:id` removes this need, but its non-obvious how to represent "delete the user's name" in the body.

Javascript plays well with providing a body like `{ "name": null }`; but as a general rule this is somewhat risky. Other languages and frameworks may be unable to consistently disambiguate between `{}` and `{ "name": null }`.

A general, consistent solution is to instead write another API: `DELETE /users/:id/name`.

A second solution is to borrow a page from MongoDB or Firestore's book, and build a kind of DSL into your API like: `PATCH /users/:id` with a body `{ "name": { $delete: true } }`. This might be preferrable if you have a large number of fields which need to support deleting.

## Status Codes

You should leverage HTTP whenever possible, and thus leverage the status codes HTTP provides.

| Status Code | Description                                                                                                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `200`       | The classic.                                                                                                                                                                                                  |
| `201`       | "Created". You can use this with `POST` requests, but the only time it should matter is if you e.g. have an API which "upserts" resources on a `POST`; then it may `200` on an update, but `201` on a create. |
| `202+`      | Not worth thinking about.                                                                                                                                                                                     |
| `400`       | Classic. Malformed request.                                                                                                                                                                                   |
| `401`       | Authentication failed. No API key provided, bad API Key, etc.                                                                                                                                                 |
| `403`       | Authorization failed. API key was provided, we know who you are, but you don't have permission to do that.                                                                                                    |
| `404`       | Classic                                                                                                                                                                                                       |
| `500`       | Ruh roh                                                                                                                                                                                                       |

Your platform might get more specific; for example, oftentimes a `500` means "internal server failure" whereas a `502` means "the server didn't even respond. Totally cool.
## Response Envelopes
A response envelope might look like:

```json
{
  "data": {},
  "status": "success"
}
```

I am not generally a fan of response envelopes, and I don't feel that APIs need them. HTTP is already an envelope; and it contains many useful dials and knobs with which you can communicate everything you might otherwise need to communicate via an envelope.

- A field like `status` is better communicated via the HTTP status code.
- A field like `timestamp` is built-in to the HTTP spec.
- When listing items via an array, a field like `count` to count the items on this page does not make sense, because the client will have to parse the whole array anyway, so they could just do `.length` or their language equivalent.
- When listing items via an array, a field like `total` to count the total number of items regardless of pagination is useful, but could be moved into an HTTP repsonse header like `X-TOTAL`.

Use the platform.

## Errors

When the server responds with a status code `>=400`, this is communicating that the server has entered an error state. When this happens, the server should respond with an error message in the response body, in plaintext.

```
404 NOT FOUND

no user with the id 123123 was found
```

The HTTP status code should be used to communicate the general class of error. If the server has a more specific error code it wants to return for debugging purposes, this should be returned as an HTTP header:

```
404 NOT FOUND
X-ERROR-CODE: user_not_found

user_not_found: no user with the id 123123 was found
```

Some servers prefer returning a JSON response body with this information encoded into that body. This isn't unreasonable, but its arguable that anything is really gained by doing this, and simpler solutions are better. The vast majority of the time, clients don't implement special behavior depending on the kind of error returned; they display it, report it, etc. If they do implement special behavior depending on the kind of error, the vast majority of _those_ cases simply switch on the HTTP responses status code (e.g. oh that user doesn't exist? ok create them.) It is a definite minority of cases where a more specific error code, diagnostic information, etc is valuable; and thus complicating the response format or forcing clients to parse and traverse JSON only leads to more brittle error handling.
