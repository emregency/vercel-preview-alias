# Vercel Preview URL Aliasing Action

![stars](https://badgen.net/github/stars/emregency/vercel-preview-alias)
![forks](https://badgen.net/github/forks/emregency/vercel-preview-alias)

[![Maintainability Rating](https://sonarcloud.io/api/project_badges/measure?project=emregency_vercel-preview-alias&metric=sqale_rating)](https://sonarcloud.io/dashboard?id=emregency_vercel-preview-alias)
[![Technical Debt](https://sonarcloud.io/api/project_badges/measure?project=emregency_vercel-preview-alias&metric=sqale_index)](https://sonarcloud.io/dashboard?id=emregency_vercel-preview-alias)
[![Bugs](https://sonarcloud.io/api/project_badges/measure?project=emregency_vercel-preview-alias&metric=bugs)](https://sonarcloud.io/dashboard?id=emregency_vercel-preview-alias)
[![Code Smells](https://sonarcloud.io/api/project_badges/measure?project=emregency_vercel-preview-alias&metric=code_smells)](https://sonarcloud.io/dashboard?id=emregency_vercel-preview-alias)
[![Reliability Rating](https://sonarcloud.io/api/project_badges/measure?project=emregency_vercel-preview-alias&metric=reliability_rating)](https://sonarcloud.io/dashboard?id=emregency_vercel-preview-alias)
[![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=emregency_vercel-preview-alias&metric=security_rating)](https://sonarcloud.io/dashboard?id=emregency_vercel-preview-alias)

This action aliases the Preview URLs of Vercel to the Target URL that you provide. It is advised to use it as part of your CI/CD efforts.

## About Vercel

[​Vercel](https://vercel.com) is a cloud platform for **static sites** and **Serverless Functions** that fits perfectly with your workflow. It enables developers to host **Jamstack** websites and web services that **deploy instantly**, **scale automatically**, and requires **no supervision**, all with **no configuration**.

## Inputs

| Name               | Required | Description                                                                                                                                            |
| ------------------ | :------: | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| vercel-token       |   [x]    | Vercel token. see https://vercel.com/account/tokens                                                                                                    |
| vercel-preview-url |   [x]    | Preview URL that you want to alias. If you use Vercel GitHub App, you can configure to a GitHub event e.g. `github.event.deployment_status.target_url` |
| vercel-target-url  |   [x]    | Target URL that you want e.g. `beta.example.com`                                                                                                       |
| vercel-scope       |   [ ]    | If you are working in a team scope, you should set this value to your `team ID`.                                                                       |

## Outputs

### `success`

Returns `true` if successful.

## Example Usage

### Integrate Vercel & GitHub

You can seamlessly integrate Vercel to Github in your [settings](https://vercel.com/account/git-integrations).

More details [here](https://vercel.com/github)

### Enable Vercel for GitHub

> The Vercel for GitHub integration automatically deploys your GitHub projects with Vercel, providing Preview Deployment URLs, and automatic Custom Domain updates.
> [link](https://vercel.com/docs/v2/git-integrations)

Set `github.enabled: true` in vercel.json.

Example of `vercel.json` :

```json
{
  "version": 2,
  "public": false,
  "scope": "emregency",
  "github": {
    "enabled": true
  },
  "builds": [{ "src": "./public/**", "use": "@now/static" }],
  "routes": [{ "src": "/(.*)", "dest": "public/$1" }]
}
```

When set to `true`, Vercel will deploy the given project to a preview URL on each `push`. You can use this feature as part of your CI/CD where each push is deployed for `serverless` testing.

Using this action, you can now alias the Preview URL to a target URL that you specify. _e.g. to run your tests with [Cypress](https://cypress.io/), or [Checkly](https://checklyhq.com/)_.

### Github Action in action

An example of workflow in your repo `.github/workflow/alias+test.yml`:

```yaml
name: Alias + Test
on: [deployment_status, push]
jobs:
  alias:
    if: github.event.deployment_status.state == 'success'
    runs-on: ubuntu-latest
    steps:
      - uses: emregency/vercel-preview-alias@v1.5
        with:
          # more about GitHub secrets https://docs.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets
          vercel-token: ${{ secrets.VERCEL_TOKEN }} # Required
          vercel-preview-url: ${{ github.event.deployment_status.target_url }} #Required
          vercel-target-url: https://beta.diari.io #Required
          vercel-scope: ${{ secrets.VERCEL_TEAM_ID }} #Optional
  test:
    needs: alias
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v1
      # https://github.com/cypress-io/github-action
      - name: Cypress run
        uses: cypress-io/github-action@v2
        with:
          record: true # great for test history in your cypress dashboard
          browser: chrome
          headless: true
          working-directory: test # useful for monorepos
        env:
          # Base URL to test against
          CYPRESS_BASE_URL: https://beta.diari.io
          # pass the Dashboard record key as an environment variable
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
          # pass GitHub token to allow accurately detecting a build vs a re-run build
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

# Thanks

This action wouldn't exist without [@amondnet](https://github.com/amondnet) and his great Vercel deployment action [Vercel Actions](https://github.com/marketplace/actions/vercel-action)
