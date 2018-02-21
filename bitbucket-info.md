# Bitbucket [Cloud Integration](https://developer.atlassian.com/cloud/bitbucket/)

## What's a Connect app ?

We must create a Connect app to integrate on Bitbucket Cloud. It's a web application that is hosted by us and will be embeded in deferent extension points of the bitbucket page inside iFrames. It means we can do anything we want with our app, the only requirement is to embed the [Atlassian Connect JavaScript client library](https://developer.atlassian.com/cloud/bitbucket/about-the-javascript-api/) in the pages that will be displayed inside the extension points. This library also allow to comunicate with the bitbucket api, open dialogs, display messages, react to events, etc.

## Auth

### Atlassian Connect

Way to go for Connect apps, based on JWT.

Our `Log in with Bitbucket` button would trigger the installation of the Connect app into Bitbucket, linking the Bitbucket account to SonarCloud.

Once the user approve the installation of the app he is redirected to the SonarCloud page.
In the meantime an installation url is called by Bitbucket (the `lifecycle:installed` property in the app descriptor file) with the security context, that contains the account id and a shared secret to sign and verify future requests. More details [here](https://developer.atlassian.com/cloud/bitbucket/authentication-for-apps/).

It means we would have to store those client ids and related shared secrets in SonarCloud for later auth/usage.

Later, when a user access a page of our plugin, a JWT of the user logged on Bitbucket will be added to the url (`?jwt=token`) so that we can identify/authorize access to the data (quality gate of a private project for exemple).

_I'm not sure yet how the authentication of user on SonarCloud is gonna work. Because during my tests I noticed that I have to grant permission to the app everytime I use the `Log in with Bitbucket` button, like the first time.
It also seems that the client key returned when the user grant permission is always the one from the team. So we still need to dig more into this._

### OAuth 2.0

Is the way we currently authenticate Bitbucket users. We can't use it with a Connect app but maybe there is a way to keep parts of it since there is a custom Bitbucket flow for exchanging JWT tokens for access tokens. If it can be of any use, check : [5. Bitbucket Cloud JWT Grant (urn:bitbucket:oauth2:jwt)](https://developer.atlassian.com/cloud/bitbucket/oauth-2/)

## Application types (context)

### Account

Limit the app to the account or team who installed the app. The app will be visible to anyone with access to the repository.

When connecting the app the user can choose where to install it, between his private account or all teams he is admin of. If installing to a team, all members of the team will see the app and only admin will be able to manage it. And it will be available regardless of the right's of the user on the repos.

### Personal

Only the user who installed the app will see the app on all it's repo.

### Pipeline integration ?

It seems like every other applications listed in the documentation under [Pipelines integrations](https://confluence.atlassian.com/bitbucket/bitbucket-pipelines-integrations-826868162.html) don't have available application for this (like the VSTS task). They all just give some example of how to integrate their tool in the pipeline: what to write in `bitbucket-pipelines.yml`, what files should be added to the project, etc...

## Extension Points

We can define different types of modules that can be pluged into extension points.
Here is a list of what's the most interesting for us :

* `webPanel`: a section inserted inside a page of Bitbucket to show related information

  * `org.bitbucket.repository.overview.informationPanel`: plug into the overview page of a repository (perfect to display the quality gate)
  * `org.bitbucket.pullrequest.overview.informationPanel`: plug into the pull request page (perfect to show result of PR analysis)

* `webItem`: a link inserted in some standard place in bitbucket to open an external page or a dialog

  * `org.bitbucket.branch.summary.info`: link on the right of the branch overview
  * `org.bitbucket.pullrequest.summary.info`: link on the right of the pull requet overview

* `fileViews`: a custom file viewer, could be our code viewer to show issues

* `adminPage`: a full page in the repo settings, might be usefull to configure the project matching this repo on SonarCloud

* `configurePage`: a full page to globally manage the connect app

## Expected implementation

Creation of a SonarCloud plugin for Bitbucket, or update of the existing one.
This plugin will implement the following behaviors :

* Authentication and creation of accounts for Bitbucket users
  * It is still unclear how we are gonna be able to do it only the Atlassian Connect mechanisme.
* Installation of the Connect app on Bitbucket users account/teams
  * It's possible to install the app from a button that will ask the user to install and grant access to our application on his account or team account.
  * Once the app is installed a webhook is called from bitbucket to give use the client id and shared secret.
  * It's still unclear if we will need to save that data, but probably to authorize access to private projects information on SonarCloud. Because all the calls to our app pages will be authenticated with a JWT in the query parameters of the url.
* Host a `adminPage` to allow the user to link his repo to a SonarCloud project
  * This will probably require us to store the Bitbucket repo id on SonarCloud side to do the mapping with the project key
* Host a `webPanel` to display the project analysis result (quality gate, ratings,...) in the repo overview
  * Calls to that hosted page will need to be authorized by SonarCloud in case of private projects, the authorization step will probably be done with the JWT and shared secret reveived at installation time.

## Upgrade and versioning

All changes done inside the web app will be available to the end user as soon as we deploy the changes on our server.

When the app is on the marketplace every changes done to the app descriptor are listened and when a change is detected the app is update to the new version. Versioning is done automatically by the marketplace.

And the changes are automatically deployed to customers within 10 hours unless it needs manual approval. It means the updates should be backward compatible or support old and new version for a limited time.

More details [here](https://developer.atlassian.com/platform/marketplace/upgrading-and-versioning-cloud-apps/).

## List the app in the marketplace

It's possible to have the application listed in the Atlassian Marketplace, more details [here](https://developer.atlassian.com/platform/marketplace/installing-cloud-apps/).

# Bitbucket [Server Integration](https://developer.atlassian.com/server/bitbucket/)

The documentation clearly state that it is not the same to develop an app for the Cloud or for the Server version.
The doc of the Server Integration seems more up to date than the Cloud version

## Server app package

* Written in Java
* Built with Maven
* App descriptor in XML with [Web Fragments](https://developer.atlassian.com/server/bitbucket/reference/plugin-module-types/client-web-fragment/)
* Possiblity to have less, css, javascript

Some example of apps : https://atlassian.bitbucket.io/#bitbucket
