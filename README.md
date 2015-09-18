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

## Development aids

`accounts-sandstorm` normally only does anything when running inside Sandstorm. However, it's often a lot more convenient to develop Meteor apps using Meteor's normal dev tools which currently cannot run inside Sandstorm. This makes it hard to test your Sandstorm integration.

To solve this, try using the package [`jacksingleton:accounts-sandstorm-dev`](https://atmospherejs.com/jacksingleton/accounts-sandstorm-dev), which lets you fake Sandstorm parameters even when running in regular dev mode outside Sandstorm!
