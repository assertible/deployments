<div align="center">
  <a href="https://assertible.com">
    <img src="https://assertible.com/images/logo/logo-512x512.png" width="100" alt="Assertible logo" />
  </a>
</div>

# Post deployment testing with Assertible

[![Assertible status](https://assertible.com/tests/c5116251-e4de-4a69-a82b-aa7045b6de60/status?api_token=138d9afe7ffa6f508e)](https://assertible.com/docs#test-badges)

> Post deployment testing is the process of running automated tests
> against your production or staging environment after deploying a new
> application version. [Assertible](https://assertible.com) integrates
> into your GitHub deployments pipeline to run your tests and
> assertions after each successful deploy.

With Assertible's
[Github Deployments integration](https://assertible.com/docs#github-deployments),
you can **run tests against your API or web application every time you
launch a new version**. Integrating into your existing CI workflow
should be straightforward. This document, and the configuration files
in this repo, will help you set up your Assertible tests to run after
new deployments.

If you don't have an Assertible account yet, you can
[**sign up for free**](https://assertible.com/signup).

## Configurations

Find your continuous integration or deployments provider below to see
the recommended steps for setting up your Assertible integration:

- [Heroku](#-heroku) ([website](https://heroku.com))

- [Travis CI](#-travis-ci) ([website](https://travis-ci.org))

- [Additional resources](#additional-resources)

- [Example Projects](#example-projects)

- [Badges!](#test-badges)

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

You can see a runnable `travis.yml` in the repo here:

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

### Creating an API Token

The configurations above assume that you have a `GH_TOKEN` environment
variable in your Travis configuration. If you don't already have that,
the easiest way to set it up is described below:

- [Create a Peronal Access Token](https://github.com/settings/tokens)
  in your GitHub settings. Make sure to give it 'repo' access.

- Add than token as an environment variable in your
  [Travis-CI repository settings](https://docs.travis-ci.com/user/environment-variables/#Defining-Variables-in-Repository-Settings)
  named `GH_TOKEN`.

## Additional resources

These links provide more information on the underlying technology and
services that make this work:

- [Assertible - Getting Started](https://assertible.com/docs)
- [Setting up Assertible and GitHub Deployments](https://assertible.com/docs#github-deployments)
- [GitHub Deployments API](https://developer.github.com/v3/repos/deployments/)

## Example projects

There are some open source projects using Assertible with this
configuration; if you're a visual learner then one of these might be
helpful:

- [reichertbrothers.com](https://github.com/rbros/rbros.github.io)
  reichertbrothers.com is the website for a Haskell consulting
  company. The website is deployed to GitHub Pages from a Travis-CI
  build. Once the site has been successfuly deployed, a deployment
  event is triggered and Assertible's post-production tests will run.

- [CheckAFlip](http://checkaflip.com)
  CheckAFlip is a tool for quickly learning the best price at which to
  buy or sell any item. The app is deployed from Heroku and, after
  deploying, Heroku sends a deployment success event and Assertible
  test's get run.

_Have an open source project using Assertble for post deployment
testing? Drop us a note and we'll add it to the list!_

## Test Badges

Assertible offers
[test badges](https://assertible.com/docs#test-badges) so you can
display the current status of your API or website's assertions. The
badges can be retrieved from within your
[Assertible dashboard](https://assertible.com/login). Here's what they
look like:

![Assertible status](https://assertible.com/tests/c5116251-e4de-4a69-a82b-aa7045b6de60/status?api_token=138d9afe7ffa6f508e)

Cool! Pick yours up today and add it to your repository -- or start in
the [documentation](https://assertible.com/docs#test-badges)

## License

All of the code snippets in this repository are licensed under MIT.

[View the license](https://github.com/assertible/deployments/blob/master/LICENSE)
