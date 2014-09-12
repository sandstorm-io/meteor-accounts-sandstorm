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

