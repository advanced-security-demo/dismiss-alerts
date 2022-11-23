# Dismiss Alerts 

The dismiss alerts solution closes code scanning alerts that are commented with a  `codeql` comment in the codebase. This action should only run on the default branch as the dismissal status of an alert is a global property. If this action is run on a push to a feature branch or pull request then the suppressed alerts will also be dismissed on the default branch. 

For each language being analysed, the `AlertSuppression.ql` queries should be run in addition to any queries already run as a part of the analysis. The Alert Suppression(add link once open sourced) solution provides CodeQL packs for this and can be utilised during analysis. You can see an example here (add link).

There are two required input fields for this action: 
- `sarif-upload-id` - this is the sarif identifier
- `sarif-file` - this is the location of the sarif file

The `suppressions[]` object in the sarif is used to create a list of suppressed alerts. The API's are used to retrieve a list of already dismissed alerts. These two lists are mapped using the alert identifier (rule and location).  A comparison is done between these lists and any alert that has not already been dismissed is updated with a PATCH request using the `github/alertUrl` property. The alert `state` is updated to `dismissed` with the `dismissed reason` being `won't fix`. A duplicate alert will be marked as fixed. This is because code scanning uses the hash of the alert's line contents as the unique identifier. The suppression comment appears on the same line and therefore a new hash is created, so uniqueness no longer holds and a duplicate is created. Since the original hash no longer exists it is marked autmatically as `fixed`

## Example 

```yaml
name: "Example Workflow"
on:
  workflow_dispatch:

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false
      matrix:
        language: ["java"]

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: ${{ matrix.language }}
          packs: advanced-security/${{ matrix.language }}-alert-suppression

      - name: Autobuild
        uses: github/codeql-action/autobuild@v2

      - name: Perform CodeQL Analysis
        id: analyze
        uses: github/codeql-action/analyze@v2
        with:
          output: ${{ matrix.language }}.sarif-results

      - name: dismiss-alerts
        uses: ./
        with:
          sarif-id: ${{ steps.analyze.outputs.sarif-id }}
          sarif-file: sarif-results/${{ matrix.language }}.sarif
        env:
          GITHUB_TOKEN: ${{ github.token }}
          
```


## High Level Architecture 


<img width="609" alt="Screen Shot 2022-11-23 at 11 02 26 pm" src="https://user-images.githubusercontent.com/68650974/203543284-4926768d-e1d1-49f1-baba-b04a978b0e2d.png">


