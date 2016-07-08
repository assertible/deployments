# Assertible post deployment testing

With Assertible's
[Github Deployments integration](https://assertible.com/docs#github-deployments),
you can run tests against your API or web application every time you
launch a new version. Integrating into your existing CI workflow
should be straightforward; below are examples for common services.

### CI and Deployment services

- [Heroku](#heroku) (https://heroku.com)
- [Travis CI](#travis-ci) (https://travis-ci.org)

### Heroku

If you have Heroku connected to your GitHub repository already, then
it is already set up to send deployment events and will work out of
the box with Assertible. In most cases, no further configuration is
required.

### Travis CI

If you're deploying from [Travis-CI](https://travis-ci.org) during the
`deploy` or `after_success` steps and not already receiving deployment
events to your repository, then you can use one of these
configurations:

`deploy` -> `after_deploy`

If you use the `deploy` step in the `.travis.yml`, then the best
option is the use the `after_deploy` step:

```yaml
    after_deploy:
      - |
        DEPLOY_ID=$(curl -XPOST --verbose "https://$GH_TOKEN@api.github.com/repos/$TRAVIS_REPO_SLUG/deployments" -H "Content-Type:application/json" --data '{"ref":"master", "auto_merge":false, "required_contexts": []}' | python -c "import json,sys;obj=json.load(sys.stdin);print obj['id'];")
        curl -XPOST "https://$GH_TOKEN@api.github.com/repos/$TRAVIS_REPO_SLUG/deployments/$DEPLOY_ID/statuses" --data '{"state":"success"}'
```
