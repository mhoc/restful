# Restful

This is a collection of some of the ideas I keep in mind while designing APIs and API endpoints.

## One API, In The Host

For a given company; we should have an API hosted on e.g. `api.mycompany.com`. This is the only token in the URL which is needed to designate that the given host is an API; put another way, a URL root like `https://api.mycompany.com/api/...` is unnecessary and stutters.

## Domain Objects In The Path

Every API for a given company should be hosted as paths underneath this host. For example, a single company may have both a Users-centric and Tenant-centric API, which may on the backend be served by two different services. These should both be on the `api.mycompany.com` host, like: `api.mycompany.com/tenants` and `api.mycompany.com/users`, rather than e.g. `api-users.mycompany.com`. 

## Environments and URLs

Less to do with APIs and more to do with setting up environments in general, but: If you have both a production and staging environment, the best way to configure your staging environment is via the registration of an entirely new domain name, such as having your website on `mycompany.com` and `mycompany-staging.com`, and API on `api.mycompany.com` and `api.mycompany-staging.com`, for production and staging respectively.

The second best way is to fully leverage the hierarchy intrinsic to DNS; `mycompany.com` hosts your website on `mycompany.com` and API on `api.mycompany.com`, while the staging environment lives on `staging.mycompany.com` and `api.staging.mycompany.com`.

## Response Envelopes

