# Shared github actions

This repo contains an action to setup codeartifact as read-only.

## Example

This action works in tandem with [aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials).
Make sure to configure your runner with the appropriate permissions!

```
name: Node.js CI

permissions:
    id-token: write
    contents: read

on:
    push:
        branches: ["main"]
    pull_request:
        branches: ["main"]

jobs:
    build:
        runs-on: ubuntu-latest

        strategy:
            matrix:
                node-version: [20.x]

        steps:
            - uses: actions/checkout@v4
            - name: Use Node.js ${{ matrix.node-version }}
              uses: actions/setup-node@v4
              with:
                  node-version: ${{ matrix.node-version }}
                  cache: "npm"
            - name: configure aws credentials
              uses: aws-actions/configure-aws-credentials@v4.0.2
              with:
                  role-session-name: GitHub_to_AWS_via_FederatedOIDC
                  role-to-assume: "arn:aws:iam::533267148632:role/GitHubAction-AssumeRoleWithAction"
                  aws-region: "eu-central-1"
            - name: "Setup CodeArtifact"
              uses: dittodev/github-action-setup-codeartifact@main
              with:
                  domain: "ditto"
                  repository: "repo"
                  domain-owner: "533267148632"
                  namespace: "@climate-tech"
                  role-to-assume: "arn:aws:iam::533267148632:role/GitHubAction-AssumeRoleWithAction"
                  aws-region: "eu-central-1"
            - run: npm ci
            - run: npm test
```

