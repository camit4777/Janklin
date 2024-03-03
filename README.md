# Janklin
Jacklin project to DeVos with the help of  code.
# azure-pipelines.yml

trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

jobs:
- job: Build
  displayName: 'Build job'
  steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '14.x'
    displayName: 'Install Node.js'

  - script: |
      npm install
      npm run build
    displayName: 'Install dependencies and build'

  - task: ArchiveFiles@2
    inputs:
      rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
      includeRootFolder: false
      archiveType: 'zip'
      archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'
      replaceExistingArchive: true

  - publish: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
    artifact: drop
    displayName: 'Publish artifact'

- job: Deploy
  displayName: 'Deploy job'
  dependsOn: Build
  condition: succeeded()
  steps:
  - task: ExtractFiles@1
    inputs:
      archiveFilePatterns: '$(System.DefaultWorkingDirectory)/drop/$(Build.BuildId).zip'
      destinationFolder: '$(System.DefaultWorkingDirectory)/drop/'

  - task: AzureWebApp@1
    inputs:
      azureSubscription: '<YourAzureSubscriptionServiceConnection>'
      appName: '<YourAzureWebAppName>'
      package: '$(System.DefaultWorkingDirectory)/drop/'


