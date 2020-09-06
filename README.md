# Power Apps Checker Action

This Action runs Power Apps Checker against the specified solution and creates a new issue with the results provided. Before you can run the checker, you would need to create an Application in Azure AD on your tenant, with permission to PowerApps-Advisor App. You can use the [PowerShell script](https://docs.microsoft.com/en-us/powershell/powerapps/get-started-powerapps-checker?view=pa-ps-latest#sample-script-to-create-an-aad-application) to create the Azure AD application with the correct permission. Once this is done, you would also need to create a secret on this Application in Azure AD, store this secret as a GitHub secret on your repo and also use it in this Action.

## Inputs

| Input | Required | Default | Description |
| ------| -------- | ------- | ----------- |
| token | ✅ | - | [GitHub Personal Access Token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token). This is required for downloading the solution files from Releases. You can also use ``${{ secrets.GITHUB_TOKEN }}`` |
| solutionName | ✅ | - | The Unique Name of the solution. This is used in Solution Upgrade step, if you are deploying a Managed Solution. |
| sourceEnvironmentUrl | ✅ | - | Environment URL from where the Solution will be downloaded from. |
| applicationId | ✅ | - | Application Id that will be used to connect to the Power Apps environment and also to the Power Apps checker. Refer [Scott Durow's video](https://www.youtube.com/watch?v=Td7Bk3IXJ9s) if you do not know how to set Application Registration and scopes in Azure to facilitate this. |
| applicationSecret | ✅ | - | Secret that will be used in the Connection String for authenticating to Power Apps. Use GitHub Secrets on your repo to store this value. Refer to [Using Encrypted Secrets](https://docs.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#using-encrypted-secrets-in-a-workflow) on how to use this in your workflow. |
| tenantId | ✅ | - | Azure AD Tenant Id |  
| ruleSet |  | Solution Checker | Solution Checker or AppSource Certification |
| geography | ✅ | - | PreviewUnitedStates, UnitedStates, Europe, Asia, Australia, Japan, India, Canada, SouthAmerica, UnitedKingdom, France, UnitedArabEmirates, USGovernment, USGovernmentL4, USGovernmentL5DoD, China | 
| locale |  | en | bg, ca, cs, da, de, el, en, es, et, eu, fi, fr, gl, hi, hr, hu, id, it, ja, kk, ko, lt, lv, ms, nb, nl, pl, pt-BR, pt-pt, ro, ru, sk, sl, sr-Cyrl-RS, sr-Latn-RS, sv, th, tr, uk, vi, zh-HANS, zh-HANT |

## Outputs

| Output | Description |
| ------| ----------- |
| issueNumber | Issue Number with the Power Apps Solution Checker recommendations. |

## Example

```yaml
name: powerapps-solution-checker-sample
on:
  workflow_dispatch:
    inputs:
      solutionName:
        description: "Solution Name"
        required: true
        default: 'Solution Name'
      sourceEnvironmentUrl:
        description: "Source Environment URL"
        required: true
        default: 'https://xxxx.crm.dynamics.com'       
      ruleSet:
        description: "Rule Set"
        required: true
        default: 'Solution Checker' 
      geography:
        description: "Geography"
        required: true
        default: 'Australia'                           
      saveResults:
        description: "Save results of Power Apps Solution Checker"
        required: false
        default: "true"                              

jobs:
  build:
    runs-on: windows-latest

    steps:
      - uses: actions/checkout@v2
                                
      - name: Solution Checker
        id: powerapps-solution-checker
        uses: rajyraman/powerapps-solution-checker@v1
        with: 
          token: ${{ secrets.GITHUB_TOKEN }}
          solutionName: ${{ github.event.inputs.solutionName }}
          sourceEnvironmentUrl: ${{ github.event.inputs.sourceEnvironmentUrl }}
          applicationId: ${{ secrets.APPLICATION_ID }}
          applicationSecret: ${{ secrets.APPLICATION_SECRET }}
          tenantId: ${{ secrets.TENANT_ID }}
          geography: ${{ github.event.inputs.geography }}
          saveResults: ${{ github.event.inputs.saveResults }}
          
      # Ideally, you would have a deploy step here, which should be blocked until issues identified by the Power Apps Solution checker have been resolved         
      - run: echo "Solution Checker identified issues. DON'T deploy."
        if: steps.powerapps-checker.outputs.issueNumber != ''
        
      - run: echo "AOK. Deploy."
        if: steps.powerapps-checker.outputs.issueNumber == ''        

```


## Credits

1. Wael Hamze for [Xrm CI Framework](https://github.com/WaelHamze/xrm-ci-framework)
2. Microsoft for [PowerShellForGitHub](https://github.com/microsoft/PowerShellForGitHub)
