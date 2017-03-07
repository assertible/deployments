<div align="center">
  <a href="https://assertible.com">
    <img src="https://assertible.com/images/logo/logo-512x512.png" width="100" alt="Assertible logo" />
  </a>
</div>

<div align="center">
  <h1>Continuously test your web services</h1>
  <h3>Post-deployment testing with Assertible</h3>
</div>

<div align="center">
  <!-- NOTE: This badge is for the 'assertible/deployments' service and it should always be passing -->
  <a href="https://assertible.com/docs/guide/web-services#web-service-badges">
    <img
      src="https://assertible.com/apis/4b4e1f08-63db-4e48-a738-750731c2321a/status?api_token=8b55a286830323effb"
      alt="Assertible status badge"
    />
  </a>
</div>
<br/>

Assertible **extends your CI pipeline** to provide **automated
post-deployment API testing**. In this repo, you'll learn how to start
continuously testing your API with Assertible and your existing CI
provider. If you don't have an Assertible account yet, you
can [get started for free](https://assertible.com/signup).

<br/>
<div align="center">
  <a href="https://assertible.com">
    <img
      src="https://s3-us-west-2.amazonaws.com/assertible/blog/assertible-3-steps.png"
      alt="Assertible GitHub status checks"
    />
  </a>
</div>
<br/>

## How it works

Setting up post-deployment testing only takes two steps:

1. [Connect Assertible to GitHub](#connect-assertible-to-github)
2. [Send deployment events to your GitHub repo](#send-deployment-events-to-your-github-repo)

After connecting one of your web services to a GitHub repo, Assertible
will _watch_ that repo for any `deployment` events. When a successful
deployment event is received, Assertible will automatically run your
tests.

_Psst - Don't host your code on GitHub? No problem! You can use
a [Trigger URL](https://assertible.com/docs/guide/tests#trigger-urls)
to initiate your tests from any script._

### Connect Assertible to GitHub

You can connect Assertible to GitHub from
the
[GitHub integrations directory](https://github.com/integrations/assertible) or
your [Assertible dashboard](https://assertible.com/dashboard). Learn
more about those options
[here](https://assertible.com/docs/guide/tests#connect-assertible-to-github).
Once you're connected, set up a
[GitHub deployment integration](https://assertible.com/docs/guide/automation#github-deployments) and
select the repo to watch for deployment events.

### Send deployment events to your GitHub repo

When a successful `deployment` event occurs on your repo, your API's
tests will be run - you can even deploy to staging or review
environments. Some CI/CD providers like [Heroku](#-heroku) already
send deployment events. In these cases no additional configuration is
required.

We've put together some scripts you can use to manually send
deployment events to your repo using the GitHub API:

- [Simple two line script](#simple-two-line-script)
- [`github_deploy` script](#github_deploy-script)

## Configurations

Find your continuous integration or deployments provider below to see
the recommended steps for setting up your Assertible integration:

- [Heroku](#-heroku) ([website](https://heroku.com))
- [Travis CI](#-travis-ci) ([website](https://travis-ci.org))
- [Circle CI](#-circle-ci) ([website](https://circleci.com))
- [Wercker](#-wercker) ([website](http://www.wercker.com/))
- [Integration scripts](#integration-scripts)
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

You can read how to enable this for your Heroku
account
[here](https://devcenter.heroku.com/articles/github-integration)

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

Read more about `deployment`
step [here](http://old-devcenter.wercker.com/articles/deployment/).

## Integration scripts

The examples in this repo are using these two simple scripts to send deployment events to GitHub. These are both open sourced and you are free to copy and use them in your own code.

#### Simple two line script

The simplest way to send deployment events to your GitHub repository is using this two-line:

```
DEPLOY_ID=$(curl -XPOST --verbose "https://$GH_TOKEN@api.github.com/repos/$TRAVIS_REPO_SLUG/deployments" -H "Content-Type:application/json" --data '{"ref":"master", "auto_merge":false, "required_contexts": []}' | python -c "import json,sys;obj=json.load(sys.stdin);print obj['id'];")
curl -XPOST "https://$GH_TOKEN@api.github.com/repos/$REPO/deployments/$DEPLOY_ID/statuses" --data '{"state":"success"}'
```

The first line creates a _new_ deployment on your repo and sends it to
GitHub. The second line then _updates_ that deployment with a
_success_ status.

The second line is important -- Assertible will only run your tests
after a _successful_ deployment event.

#### `github_deploy` script

For a more comprehensive set of features, you may wish to use the
`github_deploy` script also provided in this repo. It has several
improvements over the short two-line script:

- Support for Pending deployments
- Support for `environment_url`
- Support for `auto_inactive`

Assertible uses the `environment_url` from the deployment events to
determine what environment to test -- for example, staging or
production and will then run your tests against the environment being
deployed.

```
$ export GH_TOKEN=lk34j234
$ DEPLOY_ID=$(./github_deploy.sh $CIRCLE_SHA1 org/repo https://staging.url.com "pending")
.. run your deploy steps ...
$ ./github_deploy.sh $CIRCLE_SHA1 org/repo https://staging.url.com "success" $DEPLOY_ID
```

_Note: If your code isn't hosted on GitHub that's OK! Assertible
offers a Trigger URL you can use to initiate your API tests_.


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

- Add that token as an environment variable in
  your
  [Wercker project settings](http://devcenter.wercker.com/docs/environment-variables/creating-env-vars) named
  `GH_TOKEN`.


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

All of the code snippets in this repository are licensed under
MIT. [View the license](https://github.com/assertible/deployments/blob/master/LICENSE)


---

> [assertible.com](http://assertible.com) &nbsp;&middot;&nbsp;
> GitHub [@assertible](https://github.com/assertible) &nbsp;&middot;&nbsp;
> Twitter [@AssertibleApp](https://twitter.com/AssertibleApp)
