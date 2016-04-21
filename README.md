# Sandstorm.io login integration for Meteor.js

[Sandstorm](https://sandstorm.io) is a platform for personal clouds that makes
installing apps to your personal server as easy as installing apps to your
phone.

[Meteor](https://meteor.com) is a revolutionary web app framework. Sandstorm's
own UI is built using Meteor, and Meteor is also a great way to build Sandstorm
apps.

This package is meant to be used by Meteor apps built to run on Sandstorm.
It integrates with Sandstorm's built-in login system to log the user in
automatically when they open the app. The user's `profile.name` will be
populated from Sandstorm. When using this package, you should not use
`accounts-ui` at all; just let login happen automatically.

To use this package in your Meteor project, simply install it from the Meteor
package repository:

    meteor add kenton:accounts-sandstorm

To package a Meteor app for Sandstorm,
[use the `meteor-spk` tool](https://github.com/sandstorm-io/meteor-spk).

To enable it, you have to run Meteor with `SANDSTORM` environment variable set:

```
SANDSTORM=1 meteor
```

You can set it by adding the following value to `environ` of your command
in your `sandstorm-pkgdef.capnp`:

```
(key = "SANDSTORM", value = "1"),
```

## Usage

* On the client, call `Meteor.sandstormUser()`. (This is a reactive data source.)
* In a method or publish (on the server), call `this.connection.sandstormUser()`.

Either of these will return an object containing the following fields:

* `id`: From `X-Sandstorm-User-Id`; globally unique and stable
  identifier for this user. `null` if the user is not logged in.
* `name`: From "X-Sandstorm-Username`, the user's display name (e.g.
  `"Kenton Varda"`).
* `picture`: From `X-Sandstorm-User-Picture`, URL of the user's preferred
  avatar, or `null` if they don't have one.
* `permissions`: From `X-Sandstorm-Permissions` (but parsed to a list),
  the list of permissions the user has as determined by the Sandstorm
  sharing model. Apps can define their own permissions.
* `preferredHandle`: From `X-Sandstorm-Preferred-Handle`, the user's
  preferred handle ("username", in the unix sense). This is NOT
  guaranteed to be unique; it's just a different form of display name.
* `pronouns`: From `X-Sandstorm-User-Pronouns`, indicates the pronouns
  by which the user prefers to be referred.

See [the Sandstorm docs](https://docs.sandstorm.io/en/latest/developing/auth/#headers-that-an-app-receives) for more information about these fields.

Note that `sandstormUser()` may return `null` on the client side if the login
handshake has not completed yet (`Meteor.loggingIn()` returns `true` during
this time). It never returns `null` on the server, but it may throw an
exception if the client skipped the authentication handshake (which indicates
the client is not running accounts-sandstorm, which is rather suspicious!).

## Synchronization with Meteor Accounts

`accounts-sandstorm` does NOT require `accounts-base`. However, if you do
include `accounts-base` in your dependencies, then `accounts-sandstorm` will
integrate with it in order to store information about users seen previously.
In particular:

* A Meteor account will be automatically created for each logged-in Sandstorm user,
  the first time they visit the grain.
* In the `Meteor.users` table, `services.sandstorm` will contain the same data
  returned by `Meteor.sandstormUser()`.
* `Meteor.loggingIn()` will return `true` during the initial handshake (when
  `sandstormUser()` would return `null`).

Please note, however, that you should prefer `sandstormUser()` over
`Meteor.user().services.sandstorm` whenever possible, **especially** for enforcing
permissions, for a few reasons:

* Anonymous users do NOT get a table entry, therefore `Meteor.user()` will be
  `null` for them. However, anonymous users of a sharing link may have permissions!
* Moreover, in the future, anonymous users may additionally be able to assign
  themselves names, handles, avatars, etc. The only thing that makes them "anonymous"
  is that they have not provided the app with a unique identifier that could be used
  to recognize the same user when they visit again later.
* `services.sandstorm` is only updated when the user is online; it may be stale
  when they are not present. This implies that when a user's access is revoked,
  their user table entry will never be updated again, and will continue to
  indicate that they have permissions when they in fact no longer do.

## Development aids

`accounts-sandstorm` normally works its magic when running inside Sandstorm. However,
it's often a lot more convenient to develop Meteor apps using Meteor's normal dev tools
which currently cannot run inside Sandstorm.

Therefore, when *not* running inside Sansdtorm, you may use the following console
function to fake your user information:

    SandstormAccounts.setTestUserInfo({
      id: "12345",
      name: "Alice",
      // ... other parameters, as listed above ...
    });

This will cause `accounts-sandstorm` to spoof the `X-Sandstorm-*` headers with the
parameters you provided when it attempts to log in. When actually running inside
Sandstorm, such spoofing is blocked by Sandstorm, but when running outside it will
work and now you can test your app.

## Migrating from 0.1

In version 0.1.x of this puackage, there was no `sandstormUser()` function; the
only mode of operation was through Meteor accounts. This had problems with
permissions and anonymous users as described adove. Introducing `sandstormUser()`
is a huge update.

For almost all users, 0.2 should be a drop-in replacement for 0.1, only adding
new features. Please note, though, two possible issues:

* If you did not explicitly depend on `accounts-base` before, you must add this
  dependency, since it is no longer implied by `accounts-sansdtorm`.
* The `/.sandstorm-credentials` endpoint no longer exists. If you were directly
  fetching this undocumented endpoint before, you will need to switch your code
  to use `Meteor.sandstormUser()`.
