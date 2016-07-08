# Assertible post deployment testing

With Assertible's [Github Deployments integration](github-intgration),
you can run tests against your API or web application every time you
launch a new version. Integrating into your existing CI workflow
should be straightforward; below are examples for common services.

## CI and Deployment services

- [Heroku](#heroku) ([website](heroku))
- [Travis CI](#travis-ci) ([website](travis-ci))

## <img src="https://s3-us-west-2.amazonaws.com/assertible/integrations/heroku-logo.png" width="50" alt="Heroku" /> Heroku

If you're using Heroku and have the GitHub integration enabled, then
your Assertible integration will work without any further
configuration. On the 'Deployment' page of your Heroku app, you'll
want to see that your GitHub repository is connected:

![Heroku Github integration](https://s3-us-west-2.amazonaws.com/assertible/integrations/heroku-github-connected.png)

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


[github-integration]: https://assertible.com/docs#github-deployments
[heroku]: https://heroku.com
[travis-ci]: https://travis-ci.org
