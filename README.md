<div align="center">
  <a href="https://assertible.com">
    <img src="https://assertible.com/images/logo/logo-512x512.png" width="100" alt="Assertible logo" />
  </a>
</div>

<div align="center">
  <h1>Continuously test your web service</h1>
  <h3>Post-deployment testing with Assertible</h3>
</div>

<!-- NOTE: This badge is for the 'assertible/deployments' service and it should always be passing -->
[![Assertible status](https://assertible.com/apis/4b4e1f08-63db-4e48-a738-750731c2321a/status?api_token=8b55a286830323effb)](https://assertible.com/docs#test-badges)

Assertible **extends your CI pipeline** to provide **automated API
testing** after every deployment. Assertible will work with any
continuous integration system and we've outlined how to get started
with some of the most popular ones below.

[![Assertible status](https://s3-us-west-2.amazonaws.com/assertible/blog/assertible-github-status-check.png)](https://assertible.com/blog/github-status-checks)

You can get started with post-deployment testing by connecting your
Assertible account with GitHub, or simply using your web service's
Trigger URL. If you don't have an Assertible account yet, you
can [sign up free here](https://assertible.com/signup).

## Overview

This README describes how to connect **Assertible** tests to
**GitHub** deployment events using different providers. There are two
ways to connect your web service repository to GitHub deployment
events.

1. Using a providers built-in integration

    Some providers have an integration which works without the need to
    add any extra code. For example, [Heroku](#-heroku) will
    automatically post deployment events to GitHub (/we've primary
    tested with
    [Heroku Review Apps](https://devcenter.heroku.com/articles/github-integration-review-apps)/).

2. Custom script

    This repo provides two custom scripts for integrating your repo's
    CI pipeline w/ GitHub deployment events:

    The most basic way to use this integration is to use our limited
    two line script (copy it into your CI or manual deploy script):

    ```
    DEPLOY_ID=$(curl -XPOST --verbose "https://$GH_TOKEN@api.github.com/repos/$TRAVIS_REPO_SLUG/deployments" -H "Content-Type:application/json" --data '{"ref":"master", "auto_merge":false, "required_contexts": []}' | python -c "import json,sys;obj=json.load(sys.stdin);print obj['id'];")
    curl -XPOST "https://$GH_TOKEN@api.github.com/repos/$REPO/deployments/$DEPLOY_ID/statuses" --data '{"state":"success"}'
    ```

    For a more comprehensive set of features, you may wish to use the
    `github_deploy` script. It has several improvements over the short
    two-line script:

    - Support for Pending deployments

    - Support for `environment_url`

    Assertible uses the `environment_url` to dynamically identify the
    host your web app is being served from. Assertible will then run
    your tests against the service being deployed (as opposed to
    testing your production service).

    - Support for `auto_inactive`

    Using the `github_deploy` script is For example:

    ```
    $ export GH_TOKEN=lk34j234

    $ DEPLOY_ID=$(./github_deploy.sh $CIRCLE_SHA1 org/repo https://staging.url.com "pending")

      .. run your deploy steps ...

    $ ./github_deploy.sh $CIRCLE_SHA1 org/repo https://staging.url.com "success" $DEPLOY_ID
    ```

## Configurations

Find your continuous integration or deployments provider below to see
the recommended steps for setting up your Assertible integration:

- [Heroku](#-heroku) ([website](https://heroku.com))

- [Travis CI](#-travis-ci) ([website](https://travis-ci.org))

- [Circle CI](#-circle-ci) ([website](https://circleci.com))

- [Wercker](#-wercker) ([website](http://www.wercker.com/))

- [Additional resources](#additional-resources)

- [GitHub status checks](#github-status-checks)

- [Badges!](#status-badges)

- [Example Projects](#example-projects)

## <img src="https://s3-us-west-2.amazonaws.com/assertible/integrations/heroku-logo.png" width="50" alt="Heroku" style="margin-bottom:-10px" /> Heroku

If you're using Heroku and have the GitHub integration enabled, then
your Assertible integration will work without any further
configuration. On the 'Deployment' page of your Heroku app, you'll
want to see that your GitHub repository is connected:

<br/>
<div align="center">
  <img alt="Heroku Github integration" src="https://s3-us-west-2.amazonaws.com/assertible/integrations/heroku-github-connected.png" style="display:block;margin:auto" />
</div>
<br/>

You can read how to enable this for your Heroku account here:

- https://devcenter.heroku.com/articles/github-integration

## <img src="https://s3-us-west-2.amazonaws.com/assertible/integrations/TravisCI-Mascot-original.png" width="50" /> Travis CI

> Note that the examples below assume that you have a $GH_TOKEN
> environment variable defined in your Travis environment. See the [API
> token section](#creating-an-api-token).

If you deploy a website or API from Travis-CI (especially if you're
using the `deploy` or `after_success` steps), then it will be easy to
trigger a deployment event to run your Assertible tests. The sections
below describe the most common use-cases:

**Sections**

- [Example `.travis.yml`](#example-travis-config)
- [Using the `after_deploy` step](#deploy)
- [Using `after_script` or `after_success`](#after_success)
- [Creating an API token](#creating-an-api-token)

### Example Travis config

You can see a runnable `.travis.yml` in the repo here:

- https://github.com/assertible/deployments/blob/master/.travis.yml

_Note: You can just copy the two lines below into your existing
configuration, if you have one. Otherwise, continue reading to
determine which setup will work best._

### `deploy`

If you use the [`deploy`](https://docs.travis-ci.com/user/deployment)
step in your Travis configuration then you can create a GitHub
deployment event in the
[`after_deploy`](https://docs.travis-ci.com/user/customizing-the-build/#Deploying-your-Code)
step, like this:

```yaml
after_deploy:
  - |
    DEPLOY_ID=$(curl -XPOST --verbose "https://$GH_TOKEN@api.github.com/repos/$TRAVIS_REPO_SLUG/deployments" -H "Content-Type:application/json" --data '{"ref":"master", "auto_merge":false, "required_contexts": []}' | python -c "import json,sys;obj=json.load(sys.stdin);print obj['id'];")
    curl -XPOST "https://$GH_TOKEN@api.github.com/repos/$TRAVIS_REPO_SLUG/deployments/$DEPLOY_ID/statuses" --data '{"state":"success"}'
```

_Note: If the `deploy` command does not exit successfully then
`after_deploy` won't run._

Depending on which 'provider' you use for your deploy, you may not
need to do this step at all. For example, if you use the Heroku
provider and your Heroku account is
[linked with your GitHub](#-heroku) repository, then you will already
receive deployment events and the Assertible integration will work.

### `after_success`

If your `.travis.yml` runs a deployment during the
[`after_success`](https://docs.travis-ci.com/user/customizing-the-build/#The-Build-Lifecycle)
step, then you have two options:

- Add the following lines to the end of your existing `after_success`
  script, or
- Copy the following lines to the `after_script` step in your
  `.travis.yml`.

Example in the `after_script` step:

```yaml
after_script:
  - |
    DEPLOY_ID=$(curl -XPOST --verbose "https://$GH_TOKEN@api.github.com/repos/$TRAVIS_REPO_SLUG/deployments" -H "Content-Type:application/json" --data '{"ref":"master", "auto_merge":false, "required_contexts": []}' | python -c "import json,sys;obj=json.load(sys.stdin);print obj['id'];")
    curl -XPOST "https://$GH_TOKEN@api.github.com/repos/$TRAVIS_REPO_SLUG/deployments/$DEPLOY_ID/statuses" --data '{"state":"success"}'
```

Read more about `after_success` step here:
https://docs.travis-ci.com/user/customizing-the-build/

## <img src="https://s3-us-west-2.amazonaws.com/assertible/integrations/circleci-logo.png" width="50" /> Circle CI

> Note that the examples below assume that you have a $GH_TOKEN
> environment variable defined in your Circle CI environment. See the [API
> token section](#creating-an-api-token).

If you deploy a website or API from Circle CI (especially if you're
using the `deployment` step), then it will be easy to trigger a
deployment event to run your Assertible tests. The sections below
describe the most common use-cases:

**Sections**

- [Example `circle.yml`](#example-circleci-config)
- [Using the `deployment` step](#deployment)
- [Creating an API token](#creating-an-api-token)

### Example CircleCI config

You can see a runnable `circle.yml` in the repo here:

- https://github.com/assertible/deployments/blob/master/circle.yml

### `deployment`

If your `circle.yml` runs a
[`deployment`](https://circleci.com/docs/configuration/#deployment)
step, add the following lines to the end of the `commands` section:

```yaml
deployment:
  production:
    branch: master
    commands:
      # - ./deploy script here
      - |
          DEPLOY_ID=$(curl -XPOST --verbose "https://$GH_TOKEN@api.github.com/repos/$CIRCLE_PROJECT_USERNAME/$CIRCLE_PROJECT_REPONAME/deployments" -H "Content-Type:application/json" --data '{"ref":"master", "auto_merge":false, "required_contexts": []}' | python -c "import json,sys;obj=json.load(sys.stdin);print obj['id'];")
          curl -XPOST "https://$GH_TOKEN@api.github.com/repos/$CIRCLE_PROJECT_USERNAME/$CIRCLE_PROJECT_REPONAME/deployments/$DEPLOY_ID/statuses" --data '{"state":"success"}'
```

_Note: If you're deployment uses a built-in provider like Heroku or
AWS, deployment events may already be sent and this script would be
unecessary. If possible, determine if hooks are already being
delivered in your existing deployment steps before using this script._

Read more about `deployment` step here:
https://circleci.com/docs/configuration/#deployment

## <img src="https://s3-us-west-2.amazonaws.com/assertible/integrations/wercker-logo.png" width="50" /> Wercker

> Note that the examples below assume that you have a $GH_TOKEN
> environment variable defined in your Wercker project. See
> the [API token section](#creating-an-api-token).

If you deploy your API or website from Wercker (especially if you're
using the `deployment` step), then it will be easy to trigger a GitHub
deployment event that runs your Assertible tests. The sections below
describe the most common workflows:

**Sections**

- [Example `wercker.yml`](#example-wercker-config)
- [Using the `deployment` step](#deploy-step)

### Example Wercker config

You can see a runnable `wercker.yml` in the repo here:

- https://github.com/assertible/deployments/blob/master/wercker.yml

### `deploy` step

If your `wercker.yml` runs
a [`deployment`](http://old-devcenter.wercker.com/articles/deployment/)
step, add the following lines as the very last step in a `script`:

```yaml
deploy:
  steps:
    # This is where you would normally run your deployment. Right
    # after this, we will trigger a GitHub deployment event, which
    # tells Assertible to run your tests.
    - script:
      code:
        - |
          # These two lines will:
          #  1. Create a GitHub deployment, and save the ID
          #  2. Make the deployment 'successful'
          DEPLOY_ID=$(curl -XPOST "https://$GH_TOKEN@api.github.com/repos/$WERCKER_GIT_OWNER/$WERCKER_GIT_REPOSITORY/deployments" -H "Content-Type:application/json" --data '{"ref":"master", "auto_merge":false, "required_contexts": []}' | python -c "import json,sys;obj=json.load(sys.stdin);print obj['id'];")
          curl -XPOST "https://$GH_TOKEN@api.github.com/repos/$WERCKER_GIT_OWNER/$WERCKER_GIT_REPOSITORY/deployments/$DEPLOY_ID/statuses" --data '{"state":"success"}'
```

_Note: If you're deployment uses a built-in provider like Heroku or
AWS, deployment events may already be sent and this script would be
unecessary. If possible, determine if hooks are already being
delivered in your existing deployment steps before using this script._

Read more about `deployment` step here:
http://old-devcenter.wercker.com/articles/deployment/

## Creating an API Token

The configurations above assume that you have a `GH_TOKEN` environment
variable in your CI configuration. If you don't already have that, the
easiest way to set it up is described below:

- [Create a Peronal Access Token](https://github.com/settings/tokens)
  in your GitHub settings. Make sure to give it 'repo' access.

### Travis CI

- Add that token as an environment variable in your
[Travis-CI repository settings](https://docs.travis-ci.com/user/environment-variables/#Defining-Variables-in-Repository-Settings)
named `GH_TOKEN`.

### CircleCI CI

- Add that token as an environment variable in your
  [Circle CI repository settings](https://circleci.com/docs/environment-variables/)
  named `GH_TOKEN`.

### Wercker

- Add that token as an environment variable in your [Wercker project settings](http://devcenter.wercker.com/docs/environment-variables/creating-env-vars) named `GH_TOKEN`.


## Additional resources

These links provide more information on the underlying technology and
services that make this work:

- [Assertible - Getting Started](https://assertible.com/docs)
- [Setting up Assertible and GitHub Deployments](https://assertible.com/docs#github-deployments)
- [GitHub Deployments API](https://developer.github.com/v3/repos/deployments/)

**Alternatives**

Sometimes you don't use GitHub, or sending deployment events isn't
always possible. Assertible also supports a standalong Trigger URL
that you can to run your tests outside of the Assertible dashboard any
time. For more information, see
the [documentation](https://assertible.com/docs#trigger-url)

## GitHub status checks

Enabling this integration will also gives you
[GitHub status checks](https://github.com/blog/1935-see-results-from-all-pull-request-status-checks)
on your GitHub PR's and deployments. You can test and verify that your
staging and production environments are working correctly with
end-to-end tests every time you push new code.

Once you set up the deployment scripts from this repo, and have
[created an Assertible account](https://assertible.com/signup), you
should be ready to create a PR and check out the continuous testing
status checks in your repo.

![Assertible GitHub Status Check](https://s3-us-west-2.amazonaws.com/assertible/blog/assertible-github-status-check-master)

## Status Badges

Assertible
offers [status badges](https://assertible.com/docs#test-badges) for
your web services and tests to display the current state of your
application. The badges can be retrieved from within
your [Assertible dashboard](https://assertible.com/login). Here's what
they look like:

![Assertible status](https://assertible.com/apis/19f22308-4694-40f3-973d-69a5584e9a95/status?api_token=8b55a286830323effb)

Cool! Pick yours up today and add it to your repository -- or start in
the [documentation](https://assertible.com/docs#test-badges)

## Example projects

There are some open source projects using Assertible with this
configuration; if you're a visual learner then one of these might be
helpful:

- [reichertbrothers.com](https://github.com/rbros/rbros.github.io)
  reichertbrothers.com is the website for a Haskell consulting
  company. The website is deployed to GitHub Pages from a Travis-CI
  build. Once the site has been successfuly deployed, a deployment
  event is triggered and Assertible's post-deployment tests will run.

- [CheckAFlip](http://checkaflip.com)
  CheckAFlip is a tool for quickly learning the best price at which to
  buy or sell any item. The app is deployed from Heroku and, after
  deploying, Heroku sends a deployment success event and Assertible
  test's get run.

_Have an open source project using Assertible for post deployment
testing? Open a PR to add it to the list, or open an issue!_

## License

All of the code snippets in this repository are licensed under MIT.

[View the license](https://github.com/assertible/deployments/blob/master/LICENSE)


---

> [assertible.com](http://assertible.com) &nbsp;&middot;&nbsp;
> GitHub [@assertible](https://github.com/assertible) &nbsp;&middot;&nbsp;
> Twitter [@AssertibleApp](https://twitter.com/AssertibleApp)
