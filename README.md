# Deploying a Node.js Web App using GitHub actions

In this lab, you will learn to deploy a Node.js app to Azure App Service and set up a CI/CD workflow using GitHub Actions

## Overview

**GitHub Actions** gives you the flexibility to build an automated software development lifecycle workflow. You can write individual tasks ("Actions") and combine them to create a custom workflow. Workflows are configurable automated processes that you can set up in your repository to build, test, package, release, or deploy any project on GitHub.

With **GitHub Actions** you can build end-to-end continuous integration (CI) and continuous deployment (CD) capabilities directly in your repository. 

### What’s covered in this lab
In this lab, you will:

1. How to set up a workflow with GitHub Actions 

1. Create a workflow with GitHub Actions to add CI/CD to your Node.Js web app


### Prerequisites

1. You will need a **GitHub** account. If you do not have one, you can sign up for free [here](https://github.com/join)

1. **Microsoft Azure Account**: You will need a valid and active Azure account for this lab. If you do not have one, you can sign up for a [free trial](https://azure.microsoft.com/en-us/free/).

### Setting up the GitHub repository

**Node-Express** is an example Node.js website site. Let us fork the Parts Unlimited repository to get started with this lab.

## Create an Azure App Service

Instead of running this locally, let's create this as a web app hosted in Azure. 

1. Stop the app running locally. Click on the Azure icon in the sidebar. 

1. Click on the `+` icon to create a new app service under the **VSCode GitHub Universe HOL** subscription.

   ![](assets/images/create-app-service.png)


1. Give your webapp a unique name (we recommend calling it **theCatSaidNo-{your name}**)

1. Select **Linux** as your OS and **Python 3.7** as your runtime. 

1. Browse to your new site! 

## Set up CI/CD with GitHub Actions 

We'll use GitHub actions to automate our deployment workflow for this web app. 

1. Inside the App Service extension, right click on the name of your app service and choose "Open in Portal".

1. From the Overview page, click on "Get publish profile". A publish profile is a kind of deployment credential, useful when you don't own the Azure subscription. Open the downloaded settings file in VS Code and copy the contents of the file.

   ![](assets/images/get-publish-profile.png)


1. We will now add the publish profile as a secret associated with this repo. On the GitHub repository, click on the "Settings" tab.

   ![](assets/images/github-settings.png)


1. Go to "Secrets". Create a new secret called "WebApp_PublishProfile" and paste the contents from the settings file.

   ![](assets/images/create-secret.png)


1. Now click on "Actions" in the top bar and create a new workflow. 

   ![](assets/images/new-action.png)


1. Find the **Python application** template (not the Python Package one!) and select "Set up this workflow".

   ![](assets/images/python-action.png)


1. Let's get into the details of what this workflow is doing.

   - **Workflow Triggers**: Your workflow is set up to run on push events to the branch
     
     ```yaml
        on: [push]
     ```

     For more information, see [Events that trigger workflows](https://help.github.com/articles/events-that-trigger-workflows).
   
   - **Running your jobs on hosted runners:** GitHub Actions provides hosted runners for Linux, Windows, and macOS. We specified hosted runner in our workflow as below.

       ```yaml
       jobs:
       build:
       runs-on: ubuntu-latest

      ```
   - **Using an action**: Actions are reusable units of code that can be built and distributed by anyone on GitHub. To use an action, you must specify the repository that contains the action.
      
      ```yaml
      - uses: actions/checkout@v1
       - name: Set up Python 3.7
         uses: actions/setup-python@v1
         with:
            python-version: 3.7
      ```

   - **Running a command**: You can run commands on the job's virtual machine. We are running below python commands commands to install dependencies in our requirements.txt, lint, and test our application.

      ```yaml
        - name: Install dependencies
        run: 
            python -m pip install --upgrade pip
            pip install -r requirements.txt
        - name: Lint with flake8
         run: 
            pip install flake8
            # stop the build if there are Python syntax errors or undefined names
            flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
            # exit-zero treats all errors as warnings. The GitHub editor is 127 chars wide
            flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
        - name: Test with pytest
         run: 
            pip install pytest
            pytest
     ```

    >For workflow syntax for GitHub Actions see [here](https://help.github.com/en/github/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)

1. Now, paste these lines of code to the end of the `pythonapp.yml` file in GitHub. Change the `app-name` to the name of your web app. We are using [GitHub Azure Actions](https://github.com/Azure/actions/blob/master/README.md)to login to Azure with the publish profile stored in GitHub secrets which you created previously.

    ```yml
        - uses: azure/webapps-deploy@v1
        
        with:
            app-name:  # Replace with your app name
            publish-profile: ${{ secrets.WebApp_PublishProfile }}
    ```

   ![](assets/images/add-yaml.png)

1. Once you're done, click on "Start commit". Committing the file will trigger the workflow.

1. You can go back to the Actions tab, click on your workflow, and see that the workflow is queued or being deployed. Wait for the job to complete successfully.

   ![](assets/images/workflow-complete.png)

## Test out your app!

1. Back in VS Code, go to the App Service extension, and right click on your app service and click on "Browse Website". 

1. Let's test our GitHub Actions workflow we just made. Add the following lines of code to `templates/home.html` in the body class, after we load in the catpaw image:

    ```html
    <div>
        <h1 style="text-align:center;"> Press the button!<h1>
    </div>
    ```

    ![](assets/images/add-html-code.png)


1. In the terminal, run the following commands:

    ```cmd
    git add .
    git commit -m "test ci/cd"
    git push
    ```

1. Browse back to your website!
