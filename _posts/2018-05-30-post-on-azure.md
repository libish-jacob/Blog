---
layout: post
date: '2018-05-30 12:35 +0530'
published: true
title: post on azure
categories:
  - Azure
description: post on azure tags description.
tags:
  - Powershell
---
## Introduction
This post is about implementing a web-job in azure which will restart a service on a predefined interval. When restarting a service on a predefined interval clearly says that there is some problem with the implementation, we may rely on this as an emergency measure until we tackle the problem. Here we will ignore the implication behind it but will focus on how to get this done. To get this done, we need an application in AD which has the required permission to restart the service. The components which are required in this process are as mentioned below. Even when scripts are available to create application for you, it is good to go via the portal way to make sure that we do what we understood.


AD app
PowerShell script
Web Job service

## AD app
It is with this application that we are going to restart the application. This application should have the required permission to restart the service. In order to create this app you have to go to the 
Azure AD blade -> App registration -> New application registration

Once you have created the application using the desired name and an url(dummy also works), you have to set the service principal for the application. For this you should go to the settings of the application and select keys. Now to add key, you have two different ways. One by using a public/private key and the other by using a key and a password pair. Here we will try the key password pair for our example. To do this, create a key by entering a description and expiry as desired. Now save this and it will generate a password for you. You must save it to some safe place as you will not be able to find it again. Now your application is ready. Now we must assign a role to this application.

![7.png]({{site.baseurl}}/images/7.png)


## How to assign a role to AD app
In order to assign a role to the application in the desired resource group, you have to go to the desired resource group then select Access control and click on Add(+). Now it will open a blade to where you can select a desired role. In this case we will select the owner role. Now to the “select” text box, type your application name which you have created in your AD. It will list your application. Select that and click on save. Now you have assigned a role for the application in your resource group. Now with this you are ready to restart your application. We now need a script which can execute the restart logic.

## Powershell script.
Below is the script which is the restart logic for your webjob.

	$ProgressPreference= "SilentlyContinue"
	$appId = "your application id"
	$tenantId = "the tenant id"
	$password = "password"
	$subscriptionId = "subscription id"
	
	$secpasswd = ConvertTo-SecureString $password -AsPlainText -Force
	$mycreds = New-Object System.Management.Automation.PSCredential ($appId, $secpasswd)
	Add-AzureRmAccount -ServicePrincipal -Tenant $tenantId -Credential $mycreds
	Select-AzureRmSubscription -SubscriptionId $subscriptionId
	Stop-AzureRmWebApp -Name 'name of your service' -ResourceGroupName 'Name of your resource group'
	Start-AzureRmWebApp -Name 'name of your service' -ResourceGroupName 'Name of your resource group' >
	
** Description
$appId = "This is the application id of the application which you have created in the AD"
$tenantId = " Go to Azure AD blade, select app registration, Select endpoints and find the tenant id from any of the url mentioned. From the url you can find the tenant id. It is the one which looks like a guid."

$password - This is the secret key which you got at the time of creating a key for your application. The one which you have saved
$subscriptionId = "In azure, Go to the service that you want to restart and you can find the subscription id”

Now we need a web job which can execute this job on a periodic interval without any intervention.

## WebJob
To create a webjob, go to the resource group and select your service where you need a restart logic. Select webJobs from the settings blade of your service. Click on Add(+) and create a new web-job. Give a proper name then Upload the PowerShell script which you have created earlier and Select Triggered as the type of the job. And to the cron expression provide the desired trigger pattern. In our example we will try to run this job every minute. The pattern should be 0 * * * * *. Now say ok and your job will restart the service.
