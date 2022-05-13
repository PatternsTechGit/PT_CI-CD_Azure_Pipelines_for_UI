# Azure CI/CD Pipelines for UI 

## What is CI CD process in DevOps?

*CI* stands for continuous integration, a fundamental DevOps best practice where developers frequently merge code changes into a central repository where automated builds and tests run. But *CD* can either mean continuous delivery or continuous deployment

## About this exercise

In this lab we will be working on two code Bases, **Backend Code base** and **Frontend Code Base**. 

Previously we developed a base structure of an api solution in Asp.net core that have just two Api functions GetLast12MonthBalances & GetLast12MonthBalances/{userId} which returns data of the last 12 months total balances.
Later on we created a proper CI/CD Pipeline for our Api. Now our API is running at this URL for test environment https://bbbankapitest.azurewebsites.net/api/Transaction/GetLast12MonthBalances/

and at this URL for production environment 
https://bbbankapiprod.azurewebsites.net/api/Transaction/GetLast12MonthBalances/

![](/images/5.png)

Learn more about [API CI/CD Pipelines](https://github.com/PatternsTechGit/PT_CI-CD_Azure_Pipelines_for_API) 


For more details about this base project See: https://github.com/PatternsTechGit/PT_ServiceOrientedArchitecture

------------------
## Frontend Code base:

Previously we scaffolded a new Angular application in which we have integrated

- FontAwesome
- Bootstrap toolbar
- Returned data in html Table from a HTTP call we generated. 

Our recent output result at localhost looks like this
![](/images/6.png)

---------------------

## In this exercise
- We will create Azure Static App
- We will create Continuos Integrated CI Build Pipeline
- We will set a stage ( necessary permissions) before release pipeline.
- We will create Continuous Deployment CD Release Pipeline
- We will make changes in API and UI codes
---------------

## Step 1: Creating Static Website in Azure

There are several ways to create a static website in Azure. In this lab I will explain a way to create Free static website in a storage account.

- Open Azure Portal, search *Storage Account* and hit *Create* button. 
![](/images/7.png)

- We will select our resource group where all our resources should be
- Give a meaningful name to your storage account 
- Keep remaining items as default and hit *Review and Create* button.
![](/images/8.png)

- Create another storage account for Production environment in the same way.
- Now we have 2 storage accounts in which we will create 2 different static website for 2 different scenarios.
![](/images/9.png)

- Now first we will work on test environment. 
- Select test storage account we created
- Scroll down and select *Static Website*
- Enable the static website
- Give index document name. In our case it will be *index.html*
- Save this configuration
![](/images/10.png)

- It will give you a default URL at which your website is hosted
![](/images/13.png)

- Now if you scroll up and select **Containers** you will see $web file in it.
- Now if you run `npm build` command in your code editor terminal it will create a folder named *dist*
- If we manually upload all the contents of this folder into *$web* folder you will see your website running at provided URL. 
![](/images/12.png)

- But in this lab we want to automate and this task will be done by our CD pipeline.
- At the moment our $web folder is empty
- Follow the same procedures for **Production Storage Account** to create another website for production environment. 

------------------

## Step 2: Creating Continuous Integrated CI Pipeline

- Now we will start our procedure to create a BUILD pipeline. 
- Open Azure Devops portal, organization and the project you are working on.
- Select *pipelines* from the left menu
![](/images/14.png)

- Click *New Pipeline* button. 
- We will use classic editor for this lab
![](/images/15.png)

- Select github as a source and connect your github to your Azure pipeline via service connections 
![](/images/16.png)

- Use OAuth or personal access token from github to connect you account. [Follow this link](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to get personal access token from github. 
![](/images/17.png)

- Select your github repository and branch you want to authorize it to. 
![](/images/19.png)
- Click create tab

- We will create an empty job at the moment and will add tasks to it later on
![](/images/20.png)

- Give a meaningful name to your pipeline 
- Select your source by which this job will be triggered 
- Clean all the residuals before running any task
![](/images/21.png)

- Now we will add out first task.
- Our first task will be to install node package module
- To fulfill this task we will click '+' tab from the agents job
- Then search npm and add it to tasks
![](/images/22.png)

- We will be installing the node module so we will use install command
- Then give the path where your package.json file is located.
![](/images/23.png)

- Now we will use the build step
- Follow the same procedure and add another npm task
- This time name it accordingly. We will name it npm build
- We will use custom command in it
- Give path of you package.json folder
- Write the npm custom command. In out case it will *run build* and it will run `npm run build` 
- Save your task
![](/images/24.png)

- Now we will publish this build artifact 
- For this add another task.
- Search publish artifact and add *publish build artifact*
![](/images/25.png)

- Give meaningful name to this task
- Give the location where the folder will be dropped
- Give name to the folder. In our case name will be *dist*
![](/images/26.png)

- In the end make sure your automattic trigger in enabled 
- Enabling it will automatically start this pipeline whenever there is a change in github repository.
- Save and run your pipeline
![](/images/27.png)

- After running the pipeline it will follow all the steps and give this output if successful
- Here you can find the location of artifact
- We will publish this artifact in static website via CD pipeline
![](/images/31.png)


-----------------
## Step 3: Giving permission to organization to make changes in our Static Website

- Before moving on to CD pipeline we will first make changes in CI pipeline
- Open Azure Portal and go to the static website we created
- In left menu go to *Access Control (IAM)* and add role assignment
![](/images/28.png)

- First we will give Owner role to our organization
- Select Owner tab, click 'select member', search your organization name and save it
![](/images/29.png)

- Now we will assign *Storage blob data owner* role in the same way
![](/images/30.png)

- Follow the same procedures for Production storage account

-------------------------

## Step 4: Creating CD pipeline

- Now we will take our artifact from drop location of CI pipeline and publish it to Static webapp $web location
- To do this go to release pipeline from the left menu and create new pipeline

 ![](/images/32.png)

 - Select Empty job so we can add tasks to it later
 - Give meaningful name to this stage and save it
 ![](/images/33.png)

 - Now we add 2 tasks for this stage
 - First we will delete all the files from $web location so there will be no residual in it
 - Then we will publish our new artifact in $web location
 - To delete existing files from location, search Azure CLI and add it
 - We will run this powershell script to delete this file
 ```powershell
 az storage blob delete-batch --source '$web' --account-name bbbankuitest
 ```
- In Azure Cli task select your subscription
- Our script will inline Powershell script
- We will run the above given script with our storage account name in the end
![](/images/34.png)

- This will delete all the files from given location ($web) in static website

- Now we will add another task to publish our artifact in static website
- For this you need to add another task named *Azure file copy*
![](/images/35.png)

- Give name to this task
- Give the exact location of artifact which we dropped in CI pipeline
- Select the subscription in which our storage account is
- Write exact name of your storage account and container 
![](/images/36.png)

- In last process enable continuous trigger from this tab
- Enabling it will automatically run this pipeline whenever there is new file in drop location
![](/images/37.png)

- It will successfully create you CD pipeline. You can run this manually to check its functionality 
![](/images/38.png)

---------------------

## Step 5: Final changes in UI and API code

Now we are not hosting out API and website locally we have to make some changes in the code.
### 1) Changes in UI
- First in our UI code we have *environments.ts* file which has the URL of our API which we had previously hosted in local memory 

```C#
export const environment = {
  production: false,
  apiUrlBase: 'http://localhost:5070/api/'
};
```

- Now we will replace this Api Url Base with the URL on which our Api is hosted
- In pervious lab we showed the whole process by which we got this URL
https://bbbankapitest.azurewebsites.net
- Now we will copy this code and paste it in *environment.prod.ts* file. Now our UI will call below given URL of our API

```C#
export const environment = {
  production: true,
  apiUrlBase: 'https://bbbankapitest.azurewebsites.net/api/'
};
```

###  2) Changes in API

- Now to allow our Cross Origin Requests we have to to provide URLs of our UI's test and production environment to our API
- In API's *program.cs* file, in our previous labs, we allowed our UI's local host to communicate with our API by providing its URL.
- Now we will copy our URL from Static Website we created and paste it inside builder of *program.cs* folder

```C#
builder.Services.AddCors(options =>
{
    options.AddPolicy(name: MyAllowSpecificOrigins,
builder =>
 {
        builder.WithOrigins("http://localhost:4200" ,
                                  "https://bbbankuitest.z13.web.core.windows.net/",
                          "https://bbbankuiproduction.z13.web.core.windows.net/")
.AllowAnyHeader()
.AllowAnyMethod();
                      });
});
```

-Here the Urls of our API's are:

https://bbbankuiproduction.z13.web.core.windows.net/

and

https://bbbankuitest.z13.web.core.windows.net/

----------------

## Final Output in Azure Website 

Now if you make any changes in and commit all the changes. It will automatically trigger our CI pipeline.

Then our CI pipeline will drop artifact in provided drop location which will automatically trigger CD pipeline

It will complete its tasks and will publish the artifact in given location

Now you can run your UI's Url and it will give this output
![](/images/39.png)

---------------
## Production Environment

- We have already created static website for production environment
- Already allowed Cross Origin Request for production by pasting its URL in API
- We will keep our BUILD CI Pipelines as it is
- Now we will make some changes in Release CD Pipeline
- From release pipeline we created before, we will clone our Test environment 
- We will make this stage trigger manually. We want it to work only when we have finalized our test cases. 
![](/images/40.png)

 - We will make 2 changes in production stage
 - In first task where we deleted residual files from the folder we have to provide new location in which our production storage account is located
 ![](/images/41.png)

 - Now we will change the location of files to be copied in the second task.
 ![](/images/42.png)

 - Remaining items will remain the same.
 - Now if you run this run this stage manually you will be able to see your UI from the URL of production static website.
 
 --------------------









