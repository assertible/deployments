# <img src="https://assertible.com/images/logo/logo-512x512.png" width="50" alt="Assertible logo" style="margin-bottom:-10px" /> Assertible post deployment testing

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
your CI and deployments

**Resources**
- [Setting up Assertible and GitHub Deployments](https://assertible.com/docs#github-deployments)
- [GitHub Deployments API](https://developer.github.com/v3/repos/deployments/)


## Configurations

- [Heroku](#heroku) ([website](https://heroku.com))
- [Travis CI](#travis-ci) ([website](https://travis-ci.org))

## <img src="https://s3-us-west-2.amazonaws.com/assertible/integrations/heroku-logo.png" width="50" alt="Heroku" style="margin-bottom:-10px" /> Heroku

If you're using Heroku and have the GitHub integration enabled, then
your Assertible integration will work without any further
configuration. On the 'Deployment' page of your Heroku app, you'll
want to see that your GitHub repository is connected:

<img alt="Heroku Github integration" src="https://s3-us-west-2.amazonaws.com/assertible/integrations/heroku-github-connected.png" style="display:block;margin:auto" />


You can read how to enable this for your Heroku account here:

- https://devcenter.heroku.com/articles/github-integration

## <img src="https://s3-us-west-2.amazonaws.com/assertible/integrations/TravisCI-Mascot.png" width="50" /> Travis CI

> Note that the examples below assume that you have a $GH_TOKEN
> environment variable defined in your Travis environment. See the [API
> token section](#creating-an-api-token).

If you deploy a website or API from Travis-CI (especially if you're
using the `deploy` or `after_success` steps), then running your
Assertible tests after a successful deployment should be
straight-forward.

**Sections**

- [Using the `after_deploy` step](#deploy)
- [Using the `after_script` step](#after-success)
- [Creating an API token](#creating-an-api-token)

### `deploy`

If you use the `deploy` step in your Travis configuration then you can
create a GitHub deployment event in the `after_deploy` step, like this:

```yaml
after_deploy:
  - |
    DEPLOY_ID=$(curl -XPOST --verbose "https://$GH_TOKEN@api.github.com/repos/$TRAVIS_REPO_SLUG/deployments" -H "Content-Type:application/json" --data '{"ref":"master", "auto_merge":false, "required_contexts": []}' | python -c "import json,sys;obj=json.load(sys.stdin);print obj['id'];")
    curl -XPOST "https://$GH_TOKEN@api.github.com/repos/$TRAVIS_REPO_SLUG/deployments/$DEPLOY_ID/statuses" --data '{"state":"success"}'
```

_Note: If the `deploy` command does not exit successfully then
`after_deploy` won't run_

Depending on which 'provider' you use for your deploy, you may not
need the `after_deploy` step at all. For example, if you use the
Heroku provider and your Heroku account is linked with your GitHub
repository, then you will already receive deployment events and your
Assertible integration will work.

### `after_success`

If your `.travis.yml` runs any deployments during the `after_success` step,
then you have two options. Using the same code snippet as above, you can:

- Add the lines at the end of your existing `after_success` scrips, or
- Run the lines aboves during the `after_script` step in your
  `.travis.yml`.

### Creating an API Token

The code snippet's above assume that you have a `GH_TOKEN` environment
variable in your Travis configuration. If you don't already have that,
the easiest way to set it up is described below:

- [Create a Peronal Access Token](https://github.com/settings/tokens)
  in your GitHub settings. Make sure to give it 'repo' access.

- Add an environment variable in your
  [Travis-CI repository settings](https://docs.travis-ci.com/user/environment-variables/#Defining-Variables-in-Repository-Settings)
  named `GH_TOKEN`.
