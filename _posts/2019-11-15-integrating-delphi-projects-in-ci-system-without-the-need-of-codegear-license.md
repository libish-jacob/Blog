---
layout: post
modified: '2019-11-15 13:54 +0530'
published: false
comments: true
title: Integrating Delphi projects in CI system without the need of CodeGear license
---
## How to compile Delphi without license in CI build server

## Introduction:

	In this post I will walk you through the activities required to build Delphi on build servers without the need for a costly code-gear license on all build servers. One of a typical reason why Delphi is still used is because of the cost involved in converting those code base to a newer language. This is not the topic of discussion here, but we will look at how this can be maintained and can be integrated into the Continuous Integration (CI) system that we have.
	As you know, CI system today runs on cluster and when it comes to build and, we must keep the cost under control by opting for the opensource version of dependent tools or by using methods where we need only limited license or no license at all. Here we will see how to build Delphi in the build server with no license requirement for Code Gear. The environment that I am using is Bamboo and powershell.
    
### How to build without license

	Delphi application can be compiled using MsBuild which is now available as opensource and free of cost. But building Delphi in server using MsBuild does not come easily. If CodeGear is installed on a machine, then it is easy to build Delphi using MSBuild. He we will see how to pack the required components as a NuGet package and use that package in build agent.
    
### Which component should I pack as a nugget package?

	Install CodeGear Delphi on your machine and setup the environment such that it can compile Delphi project. After this, copy the BDS folder from the Delphi installation path. You can find this from CodeGear by going to Tools->options and as shown in the image below
    
![Image1.JPG]({{site.baseurl}}/images/Image1.JPG)


	Now we need one more folder. When you setup CodeGear for delphi, then CodeGear also saves a project in your AppConfig which saves the path to dependency. This is the key. Without this, we wont be able to compile Delphi in a machine where it is not configured. So lets copy this folder as well. For this, you must go to %AppData%\CodeGear\BDS and copy the 6.0 folder. Please note that I am using Delphi 2009 the folder name may change but please adjust according to the version that you use.
    
	Now we need to update the link which connects BDS to the Configuration folder(BDS in AppData). To achieve this, we have to modify the file, 6.0\bin\CodeGear.Common.Targets which is available in the main BDS folder which we have copied initially. Now, update the settings such that it reflects the new path as shown below. Please note that what we are trying to achieve is a folder structure which looks like as shown in the image below. And the path updated below is reflecting the folder structure as shown below. Remember after doing this we are going to pack this folder as a nuget package. So whoever is referring to this will get this folder and can build their Delphi project using this.
    
![Image22.JPG]({{site.baseurl}}/images/Image22.JPG)


Modify file CodeGear.Common.Targets  as shown below.

Change:  
{% highlight csharp %}
<Import Project="$(APPDATA)\CodeGear\$(BDSAppDataBaseDir)\6.0\EnvOptions.proj" Condition="Exists('$(APPDATA)\CodeGear\$(BDSAppDataBaseDir)\6.0\EnvOptions.proj') and '$(ProjectVersion)'!=''"/> 
{% endhighlight %}

To: 
{% highlight csharp %}
<Import Project="..\..\BDS\6.0\EnvOptions.proj" Condition="Exists('..\..\BDS\6.0\EnvOptions.proj') and '$(ProjectVersion)'!=''"/>
{% endhighlight %}

Now we have prepared the folder which can be packed as a nugget package which can be packed and hosted in NuGet server. The below PowerShell script explains how you can download the package from NuGet repository and use that along with msbuild to compile Delphi project. The script is self-explanatory.

{% highlight csharp %}
$location = (Resolve-Path .\).Path
if([string]::IsNullOrEmpty($env:bamboo_build_working_directory))
{
//Lets ensure that if the script is run from the root of the soulution folder then it //should run. This is required if a user wants to build it locally from his solution //folder. If not applicable, then please ignore.
    $env:bamboo_build_working_directory = $location
}

$PackageSource = "http://tswbin/artifactory/api/nuget/LacTools" 
Register-PackageSource -Name LacTools -Location $PackageSource -ProviderName NuGet -Force -Trusted

// Delphi package is packed using the name "Delphi2009"
$DelphiPackage = "Delphi2009"

Write-Output "Downloading Delphi package"
$Error.Clear()
Install-Package $DelphiPackage -source LacTools -destination ..\packages -ExcludeVersion
if($Error.Count -ne 0)
{
    Write-Output "Downloading package failed."      
    Write-Output $Error[0]    
    exit 1
} 
 
$Error.Clear()
Write-Output "Extracting Delphi package"
Get-ChildItem ..\packages\$DelphiPackage\Delphi\ -Filter *.zip | Expand-Archive -DestinationPath .\Delphi -Force
if($Error.Count -ne 0)
{
   Write-Output "Extracting Delphi package failed."
   Write-Output $Error[0]    
   exit 1
}

Write-Output "Extracting Delphi package done!"
$Error.Clear()
Write-Output "Building solution"

// Set the environment variable.
$env:BDS="$location\Delphi\6.0"
$msbuild = "C:\Program Files (x86)\MSBuild\14.0\Bin\msbuild.exe"
if (-not (Test-Path $msbuild))
{
    Write-Output "Building using compiler from visual studio."
    $msbuild = "C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\MSBuild\15.0\Bin\MSBuild.exe"
}

& $msbuild /p:Config=RELEASE /t:Clean,Build "$env:bamboo_build_working_directory\PrepProjectGroup.groupproj"
if (! $?) 
{ 
    Write-Output "msbuild failed" 
    exit 1;
}
else
{
    Write-Output "msbuild was successful" 
}

{% endhighlight %}