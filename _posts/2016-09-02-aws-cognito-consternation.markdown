---
layout: post
title: aws cognito consternation
---

# demistifying AWS Cognito

## what... is "AWS Cognito"?

the official [front page](https://aws.amazon.com/cognito/) says:

> Add user sign-up, sign-in, and data synchronization to your apps

but when you login to your AWS console and [click on Cognito](https://console.aws.amazon.com/cognito/home):

![aws console link to cognito](/assets/cognito-console-link.png)

you're faced with the following crisis... does one:

> Manage your User Pools

or

> Manage Federated Identities

![cognito home page](/assets/cognito-home-page.png)

what is a "user pool" and wtf is a "federated identity"? all i want to do is add _sign-up, sign-in and data synchronization_ to my Javascript app and now i'm dealing with the cognitive dissonance of trying to understand why there are 2 services: one using the word _user_ and one using the word _identity_.

my (unrealistic) expectation was a **tiny** javascript snippet to drop in that would yield auth forms and a session **(spoiler: the actual javascript involved is over 1 megabyte (minified), there's nothing _drop in_ about it and it's slow AF)**

by the way, take heed to note that "user pool" are "federated identities" could be renamed "my personal database of users hosted by amazon" and "ids that represent the same user from different authentication providers like google and facebook"

### cognito is actually 3 separate things

1. user pools
  - literally a database of users with attributes (think, a ruby on rails app with devise installed)
  - almost useless on its own
2. federated identities
  - literally another database of users... except it abstracts various identity providers (AWS cognito user pool, facebook, google, etc...)
3. cognito sync
  - merely a key/value store (1mb max) for individual users authenticated with a federated identity (cognito sync doesn't work with user pools alone, you need the federated identity)

### so how do i setup and use these 3 cognito things?

#### setup

1. first create the _federated identity_ (unless you don't want to authorize any AWS services)
2. then optionally create an aws cognito _user pool_, otherwise you'll have to create a google app, facebook app etc...
3. then link the _federated identity_ you created in step 1 with whatever you created in step 2 (from the federated identities page)

#### usage

thus far, there's no example apps or UI - just APIs for creating and authenticating users. this means there's still a lot of UI code to write (and the UI code for signIn/signUp/confirm/forgot is substantial)

if you want your own user database (user pool) the high level workflow looks like:

1. authenticate with a provider (AWS cognito user pool, facebook, google, etc...)
  - at this point you'll have a session but no ability to authenticate with any AWS services - you could, in theory stop here if you didn't want to use any AWS services with your session
2. set the `AWS SDK`'s credentials object to a `CognitoIdentityCredentials` object (where that object has the JWT from step 1)
3. now happily use any services in the `AWS SDK` (anything that's authorized via the _federated identity_ policies - by default you get `Cognito Sync` and `Mobile Analytics`)

if you don't want your own user database (user pool), **then skip step 1** (but you'll have to use google, facebook etc...) - step 1 being optional is more obvious when you learn user pools were a [feature added late (apr 2016) in cognito's existence](https://aws.amazon.com/blogs/aws/new-user-pools-for-amazon-cognito/)





## cons of aws cognito (sep 2016)
* yuge javascript payload
* poor organization/documentation/examples
* callback hell javascript library (**no usage of promises**, and inconsistent use of nodebacks and `onSuccess`/`onFailure` handlers)
* **hilariously slow**
  * 500ms ttfb
  * 2.5s (seriously) to authenticate
* little or no community code/plugins

## pros of aws cognito (sep 2016)
* best practice security
* scalable and cheap
* very decent UI within the AWS Console

if you care about performance and getting shit done, then [https://auth0.com/](https://auth0.com/) is currently a better option

### reference
* aws pages for the 3 cognito services
  * aws cognito (the parent/umbrella name) - [https://console.aws.amazon.com/cognito/home](https://console.aws.amazon.com/cognito/home)
    * **federated identity** - [https://console.aws.amazon.com/cognito/federated/](https://console.aws.amazon.com/cognito/federated/)
      * **sync** (find it on the federated identities page) - [https://console.aws.amazon.com/cognito/identities/](https://console.aws.amazon.com/cognito/identities/)
      - browse the key/value store for any user
    * **user pools** - [https://console.aws.amazon.com/cognito/users/](https://console.aws.amazon.com/cognito/users/)
* javascript sdks (the github project names are confusing)
  * **user pools** - [https://github.com/aws/amazon-cognito-identity-js](https://github.com/aws/amazon-cognito-identity-js)
  * **sync** - [https://github.com/aws/amazon-cognito-js](https://github.com/aws/amazon-cognito-js)
  * full blown aws sdk - [https://github.com/aws/aws-sdk-js](https://github.com/aws/aws-sdk-js)