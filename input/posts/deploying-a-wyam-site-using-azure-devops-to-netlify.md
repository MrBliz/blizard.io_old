Title: Deploying a Wyam site to Netlify with Azure Devops.
Published: 2018-12-29
Tags: 
    - Netlify
    - Wyam
    - Azure DevOps
    - Azure 
---

Netlify is a great tool for hosting static sites. For free, you can host a site that has a custom domain and HTTPS, and you don't have to worry about scaling it if you hit the front page of Hacker News. For a host of static site generators, Netlify can even generate your site for you. Simply push your code to a source code repository that Netlify supports, and Netlify will pick up the changes, generate, and deploy your site.

Unfortunately at the moment, Netlify does not have dotnet available, and hence cannot generate a Wyam site, so until Netlify does support .NET Core, you will need to have a separate CI step to generate the site and then push a zip file to netlify.

The steps that we'll follow for this process are.

* Create Wyam Site
* Publish to Github
* Create Netlify Api Key and Site ID
* Set Up Build Pipeline to create a zip file of generated site
* Push site to netlify.
* Repeat for staging

Assuming you have already got dotnet core installed, and installed the wyam global tool using `dotnet tool install -g Wyam.Tool`, do the following.

`mkdir wyamblog`

`cd wyamblog`

`wyam new --recipe Blog`

`wyam -p -w`

You should now see the front page of the blog when navigating to the default address http://localhost:5080

Now create a new git repo and remote, and push it the source code repository of your choice (i'll be using Github)

Register for an account at Netflify if you haven't already got one, and then navigate to `OauthApplications` and create a new personal access token, and remember to save it somewhere for future reference.

If you haven't an Azure Devops account, go and create one (It's free for open Source use) and then create a project.

Navigate to Pipelines in the side menu, and create one. 

For the purposes of this post, we'll be using the new Azure Devops YAML designer 
We'll now need to specify where our code is (Incidentally the reason i'm using Github and not Azure repos, is that if Netlify in future supports building wyam sites, it won't be able to access the code if it's in Azure repos). You'll need to give Azure devops permission to access your repository.

I'm going to install the Azure Pipelines app to github, and give it access to the wyamblog project in Github.

Choose the starter pipeline template and replace the code in the template with the below snippet.

 ```
trigger:
- master

pool:
  vmImage: 'Ubuntu-16.04'

variables:
  configuration: debug
  platform: x64

steps:
- task: DotNetCoreInstaller@0
  displayName: Install .NET Core SDK
  name: install_dotnetcore_sdk
  enabled: true
  inputs:
    packageType: 'sdk'
    version: '2.2.101'

- script: dotnet tool install -g Wyam.Tool
  displayName: Install Wyam

- script: wyam
  displayName: Build Site 

- task: ArchiveFiles@2
  displayName: Zip Site
  inputs:
    rootFolderOrFile: '$(Agent.BuildDirectory)/s/output' 
    includeRootFolder: true
    archiveType: 'zip'
    archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip' 
    replaceExistingArchive: true

- script:  >-
      curl
      -H 'Authorization: Bearer $(netlifyAccessToken)' 
      -H 'Content-Type: application/zip'
      --data-binary '@$(Build.BuildId).zip'
      https://api.netlify.com/api/v1/sites/$(netlifySiteId)/deploys
  workingDirectory: '$(Build.ArtifactStagingDirectory)'
  displayName: 'Upload to Netlify'
```


This will do the following

* When a commit is made to the specified branch, trigger a build using an ubuntu VM.
* Install the relevant SDK
* Install the Wyam global tool.
* Execute the Wyam Tool and build the site
* Zip the output directory
* push the output directory to Netlify using cURL

If we save that script and run it. It will fail, as there are two variables in the script that Azure Devops cannot find

`$(netlifyAccessToken)`

`$(netlifySiteId)`

We have already created the access token, but we need a site id. Navigate to `https://app.netlify.com/account/sites` and that should show you a list of sites in your Netlify account.

Netlify allows you to create sites from a GIT repo, but it also allows you to create a site by dragging and dropping a bunch of files too. If you drag the output folder onto the drop zone it will instantly create a site for us. You can verify that the site works, but what we are most interested in is the site ID. Navigate to the site Settings tab, and copy the API ID and save it somewhere.

Next in Azure devops, on the YAML edit screen, click on 'Edit in the Visual Designer', and then on the next screen go to the variables tab.

We'll need to create two new variables. First one called `netlifyAccessToken` copy the value for the Access Token you created in Netlify earlier, and set it to secret by clicking the lock. This will ensure that the token is encrypted and not printed out in any logs. Secondly create a variable called `netlifySiteId` and copy the value from the API ID from Netlify. Now save and Queue the build. Once it's finished, go back to Netlify, and you should see on the deploys tab, a new record for your build. 

## Staging

That's our master branch sorted, but what if you want to preview some changes before they go live? Perhaps see what a new theme looks like? We can do this by creating a new pipeline for a staging branch.

Firstly make sure you have the `azure-pipelines.yml` file in your local repository by pulling from github. This the file that Azure Devops has added to your repo that contains the build pipeline steps.

Next checkout a new branch, i'll call it 'staging', edit the yml file so that the trigger for new build is the name of the branch that you have just created. Make a visual change to your site to differentiate it from what is on master, then commit and push those changes.

Create a new site in Netlify by dragging and dropping again, and copy down the new API ID from the settings tab. Go back to Azure Devops and click on 'Builds' in the sidebar. 

From the menu on the right, select 'clone' and give the cloned pipeline a new name. I added the name of the branch on the end of mine.

If you click on the first step of the pipeline. If you haven't renamed it, it will probably be called 'Get Sources'. We need to change the branch that the build will get its changes from to staging.

Next click on the variables tab, and change the `netlifySiteId` variable to the new API ID for your staging site. 

Save and Queue a build, and you should see your new changes propagated. 

Now let's test out that the trigger works by making some more changes on the staging branch. After pushing your changes, and the build you should see them live!


## Summary
 In this post we created a static site using Wyam, and created a build pipeline in Azure Devops, to get around the fact that Netlify doesn't yet support dotnet. Hopefully within a few months, they will add that support, and this post will become redundant, but in the meantime, I hope you found it useful.

 This was my first use of the YAML creation experience in Azure Devops, so i certainly learnt a lot on the way.













