## Software-as-a-Service Skeleton

This repo aims to hold the infrastructure for a "generic" SaaS product for Cloud
deployment. Every product needs users and user management, which needs some kind
of storage; most use SQL for that unless they're truly huge (a la Google), so
every product needs a SQL interface.  Most need to send messages outbound and
to handle file-like artifacts (uploaded inputs, generated outputs, etc), so the
skeleton abstracts those too.

**The design constraint is that it should be possible to run at a "toy" scale
out of the box _for free_.**  

Since Amazon doesn't offer a free tier, this skews
towards Google Cloud, but is intended to be easy to migrate if that is not the
desired platform---the requirements for SQL and files and messaging and secrets
are constants, and the different clouds have fairly similar solutions to those
baseline needs.

There is some bias towards a "monolith" architecture, not because that is a
good end state but because it is a good _starting_ state; if you design your
modules correctly, i.e. thinking of them as services that happen to share a
process space, then teasing a monolith into smaller services is not trivial,
but also not insane, whereas premature separation is added friction during the
initial prototyping phase.

## Execution Models
Three distinct execution models are supported:
# Local: the appliction runs on a developer's desktop, with actual file and
  local database; messages are written to file for inspection.  This is the
  edit/run/debug loop for developers.
# Personal Cloud: the application runs in a GCP project, but is still intended
  to be
  a developer's execution for testing, at "toy" scale and well within the free
  tier. (Developers are expected to make their own projects, perhaps in a
  corporate email identity rather than their personal one.)
# Cloud: there are both "staging" and "production" environments expected, but
  these are much more "real" in scaling and complexity.

## Service Abstractions
The following ports have distinct interfaces for the local and cloud:

### File-like Storage
This is GCS in cloude, and local files in local run.  By default in stores in
webapp/runtime-data, but that can be changed.

Note that the `list()` method is recursive-by-default.  In a real file system,
this is unusual; however, in GCS, "directories" are not really a thing so all
files list by pattern-matching.  This makes recursive listing mildly unexpected
locally, but non-recursive listing becomes much more expensive in cloud.  So
the default behavior is recursive.

### Secrets storage
Google Secret Manager in cloud, local files (one per secret, by default in
`webapp/local-secrets`) when run locally.

### SQL

### Messaging

## Data Model Assumptions

### Users, Tenant Organizations, and Identity
We expect a multi-tenant architecture with federated user identity. This means
that there is a USERS construction outside of any particular tenant, mapping
from multiple identity providers (Google, Discored, Facebook, SSO, etc) to a
user identity for this application; it is possible for the same user to be
identified by multiple providers.

Users are members of perhaps multiple organizations, so this membership is also
tracked outside of the tenancy layer.  However, there is no exposed way to list
all organizations or users: a new visitor can create a new organization AND a
new user identity together, or can login as a known user.  An existing
organization can issue an invitation to an address, as the only way for a new
user to join an existing organization.

On login, a user is presented a list of the organizations to which they have
access, and can bind their login session to their choice of those organizations.
The login session therefore captures a user/org pair.

Within an organization, there are administrator users who can issue or revoke
invitations, as well as removing users and promoting or demothing their access
as aministrators or not.

The baseline schema tracks these entities and relationships, as well as the
last-seen timestamp for a user overall and for the user in each of their
organizations.  It also tracks a terms-of-service revision the user last agreed
to.

Anything _else_ crosses over into product features, not skeleton; we're aiming
to provide all the "everybody needs this" baseline that you build your product
on top of.

