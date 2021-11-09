# Terraform Cloud

Terraform Cloud is a managed service that provides you with Terraform CLI to provision infrastructure, either on demand or in response to various events.

By default, Terraform CLI performs operation on the server where it is invoked, it is perfectly fine if you have a dedicated role who can launch it, but if you have a team who works with Terraform – you need a consistent remote environment with remote workflow and shared state to run Terraform commands.

Terraform Cloud executes Terraform commands on disposable virtual machines, this remote execution is also called remote operations.

## Migrate Terraform files to Terraform Cloud

- Create a Terraform Cloud account from this link https://app.terraform.io/signup/account
- Create an Organization. Click "Start from Scratch", enter a name and click create.
- Create a workspace.
  - Create a new GitHub repo that will contain your tf configuration
  - Push the files from the previous projects to the repo
  - On Terraform Cloud, click create workspace
  - Select "version control workflow"
    as our VCS(Version Control System)

![1](https://user-images.githubusercontent.com/47898882/141002333-7b86dd2e-906a-49d4-8b73-f12aba5e4952.JPG)

- Select the Version Control Provider you've created. In this instance we'll be using github

![2](https://user-images.githubusercontent.com/47898882/141002344-44d8cc8a-8798-40b7-9e64-c82274d660d6.JPG)

- Add the migrated terraform project codebase [repository](https://github.com/brpo01/terraform-project)

![3](https://user-images.githubusercontent.com/47898882/141002348-f81a5a52-9a11-4b44-9081-c97b07a5ee07.JPG)

- Provide a description for your workspace and click "Create workspace"

![4](https://user-images.githubusercontent.com/47898882/141002350-ad137788-9f0a-4082-9ffd-3493e5290899.JPG)

- Configure variables

  - Click Variables from the top tab in your workspace, under the workspace name
  - Scroll down to Environment Variables and create variables for your access key id and secret access key you got from AWS.

  ![5](https://user-images.githubusercontent.com/47898882/141002360-0630f447-263d-4502-89dc-04ba93053b29.JPG)

- Run **plan** and **apply** from console
  - Click on the Runs tab
  - Click "Queue plan"
  - After the **plan** run is complete, click "Confirm and apply". Enter a comment and click "Confirm plan" to apply the configuration.

![6](https://user-images.githubusercontent.com/47898882/141002364-495afa1a-a159-42a1-a84a-90ec23f2685e.JPG)

- Test automated **terraform plan**
  - Edit a file in your repo, and commit. A **plan** run should be triggered automatically.

## Practice Task 1

- Configure 3 branches in your terraform repo for dev, test and prod
- Make necessary configurations to trigger runs automatically only for **dev** environment.

  - Create a new workspace, select "version control workflow"

  - Select "GitHub" as version control provider

   ![7](https://user-images.githubusercontent.com/47898882/141002371-a3293dbe-884f-41a4-9b7a-ed0cd31194bb.JPG)

  - Choose the repo that contains your .tf files
  - Enter the workspace name (e.g terraform-cloud-dev)

  - Click Advanced options, under VCS branch, enter the branch you want to configure (e.g dev)

  ![8](https://user-images.githubusercontent.com/47898882/141004382-0ef6fd0e-2e53-4222-ae09-dc6d0bbab647.JPG)

  - Select a source workspace to create a run trigger

  ![9](https://user-images.githubusercontent.com/47898882/141004390-90a8efec-5051-429f-8b45-7b2bee25fd0a.JPG)

  - Click Create Workspace
  
  - Configure your variables

Change the name of terraform.tfvars to **_terraform.auto.tfvars_**. Terraform Cloud will automatically pick up the variables from the version control

- Click the Settings drop down, beside Variables
- Click Run Triggers
- Select the workspace you created previously
- Create email and Slack notifications
  - Click on Settings -> Notifications
  - Select Email
  - Enter a name for the notification
  - Select notfication recipients
  - Under Triggers, click the "Only certain events" radio button
  - Check the boxes you want to be notfied for

![10](https://user-images.githubusercontent.com/47898882/141004392-02ae3204-a5fd-4d2a-9347-6c2ef39cd226.JPG)

![11](https://user-images.githubusercontent.com/47898882/141004394-47c30815-696f-45d0-8d20-d64870168d95.JPG)


- To configure Slack notification, choose Slack instead of Email
- See how to get your Slack webhook here: https://api.slack.com/messaging/webhooks#create_a_webhook
- Then configure everything else as with Email configuration
- Apply destroy
  - Click Settings -> "Destruction and Deletion"
  - Under Manually destroy, click "Queue destroy plan"
  - Enter the workspace name and click "Queue destroy plan"

## Practice Task 2 (https://learn.hashicorp.com/tutorials/terraform/module-private-registry-share)

- Create a simple Terraform repository that will be your module
  - Clone from this repo https://github.com/hashicorp/learn-private-module-aws-s3-webapp
  - Your repo's name should be in the form terraform-\<PROVIDER>-\<NAME>
  - Click "Tag release" from the left pane
  - Create a new release
  - Click "Create a new release", and add 1.0.0 to the tag version field, and set the Release title to anything you like.

  - Click "Publish release"

![12](https://user-images.githubusercontent.com/47898882/141006248-e67ce941-25c2-45f9-b233-064099beef21.JPG)

- Import the module into your private repository
  - On your Terraform cloud, click Registry on the top pane
  - You'll need to add a VCS provider. Select GitHub (Custom) when prompted
  - Follow the outlined steps
  - Click connect and continue
  - Go back to Registry
  - Click "Publish private module" 
  - Click the VCS you configured and find the name of your module repo
  - Select the module and click the "Publish module" button

  - Copy the configuration details, you'll need it later for when you want to use the module.

  ![13](https://user-images.githubusercontent.com/47898882/141006256-51dd8737-17a3-4133-8e5c-b24a0b01daee.JPG)

  - Create a configuration that uses the module

  - You can fork this repo https://github.com/hashicorp/learn-private-module-root/
  - Create main.tf, variables.tf and outputs.tf files
  - In your main.tf file, paste in the following block

  ```
  terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
  }

  provider "aws" {
    region = var.region
  }

  module "s3-webapp" {
    source  = "app.terraform.io/Darey-PBL/s3-webapp/aws"
    name   = var.name
    region = var.region
    prefix = var.prefix
    version = "1.0.0"
  }
  ```

  ![{3DA80531-0AB9-404F-902B-2CFB7BADB882} png](https://user-images.githubusercontent.com/76074379/132107226-0824d01d-8782-4a00-b372-c5e6a2aba978.jpg)

  replace the **module** block with the configuration details you copied earlier.

  - In your variables.tf file, add the following

  ```
  variable "region" {
    description = "This is the cloud hosting region where your webapp will be deployed."
  }

  variable "prefix" {
    description = "This is the environment your webapp will be prefixed with. dev, qa, or prod"
  }

  variable "name" {
    description = "Your name to attach to the webapp address"
  }
  ```

- Terraform Cloud uses the outputs.tf file to display your module outputs as you run them in the web UI.

```
 output "website_endpoint" {
 value = module.s3-webapp.endpoint
}
```

## Create a workspace for the configuration

In Terraform Cloud, create a new workspace and choose your GitHub connection.

Terraform Cloud will display a list of your GitHub repositories. You may need to filter by name to find and choose the your root configuration repository, called learn-private-module-root.

Leave the workspace name and "Advanced options" unchanged, and click the purple "Create workspace" button to create the workspace.

Once your configuration is uploaded successfully, choose "Configure variables."

You will need to add the three Terraform variables prefix, region, and name. These variables correspond to the variables.tf file in your root module configuration and are necessary to create a unique S3 bucket name for your webapp. Add your AWS credentials as two environment variables, AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY and mark them as sensitive.

![{92E2D916-F4D6-4222-93AA-43D58B17D539} png](https://user-images.githubusercontent.com/76074379/132107624-9a62b4d9-02a9-4f72-9db7-b13f559f4261.jpg)

## Deploy the infrastructure

Test your deployment by queuing a plan, then confriming and applying the plan in your Terraform Cloud UI.

![{9123759B-4144-461F-8F83-248FA6AB30B3} png](https://user-images.githubusercontent.com/76074379/132107732-f8ebd9be-aab8-4298-868d-a568eaef3525.jpg)

![{074CD6D3-90F4-472D-95AE-9CB8FDC67061} png](https://user-images.githubusercontent.com/76074379/132107757-70919cc7-48d5-466b-a17b-f47a37fdb265.jpg)

## Destroy the deployment

You will need queue a destroy plan in the Terraform Cloud UI for your workspace, by clicking the "Queue destroy plan" button.

![{5F6A230B-E0CD-4435-9617-7D474C28F447} png](https://user-images.githubusercontent.com/76074379/132109240-5e9f941b-9eb6-49af-a6d7-23205dc79faa.jpg)
