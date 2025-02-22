# Azure Devops Extension- PublishHTMLReports
This extension can be used in Azure devops to publish Jmeter HTML reports as a seperate tab(Parallel to Summary tab) in Build Pipelines(Please note Release Pipelines is not supported). Right now full support of Jmeter report and any generic html report has been developed, however this extension can also be extended to publish other complex HTML reports as well.

This extension has been tested with the following Jmeter versions:

* 4.0
* 5.0
* 5.3

The file structure of Jmeter reports in version 4 and 5 hasn't changed, so it is expected that this extension will work for all 4.* and 5.* versions of jmeter.

## Usage:

### 1. Install the below extension in your azure devops org:
https://marketplace.visualstudio.com/items?itemName=LakshayKaushik.PublishHTMLReports&targetId=c2bac9a7-71cb-49a9-84a5-acfb8db48105&utm_source=vstsproduct&utm_medium=ExtHubManageList



### 2. Run command line Jmeter with -o arg in your azure pipeline.
For e.g. jmeter -n -t TestAPI.jmx -l LoadReports/results.jtl -e -o LoadReports
This produces result files and folders having detailed jmeter report of the run.


### 3. Now, use the extension in your azure devops pipeline to publish this report on Azdo.

```
- task: publishhtmlreport@1
  inputs:
    htmlType: 'Jmeter'
    JmeterReportsPath: '$(Build.SourcesDirectory)/LoadReports'
```

This will make the Jmeter report compatible to be viewed and analysed within azure devops.

If you want to publish a simple HTML document to AZDO in a seperate tab then do the following:

```
- task: publishhtmlreport@1
  inputs:
    htmlType: 'genericHTML'
    htmlPath: '$(Build.SourcesDirectory)/Sample.html'
```

## This is how Jmeter reports look within azdo after using this extension:

![Alt text](images/Sample7.png?raw=true "Dashboard")

## Customizing the extension

As the extension currently supports Jmeter report and generic HTML reports (single page), it makes sense to customize the extension to support multiple other type of HTML reports like Locust, Robot etc. This type of customization can be done by making a contribution to the extension. 

### Anatomy of the extension

This extension contains 2 major components: 

#### 1. Azure devops task(publishhtmlreport) 
This is used to publish files to be consumed by the extension. This task does a console.log('##vso[task.addattachment type=type;name=name;]') of the files to be sent to the extension. More about how this works is here: https://docs.microsoft.com/en-us/azure/devops/pipelines/scripts/logging-commands?view=azure-devops&tabs=bash.

#### 2.Frontend code for rendering report content
When js files are sent to the extension, it reads the content of the js files and does an inline append of the js files in the index.html page(main page of the extension). After all js scripts are appended in the html page, index.html containing the jmeter load test report is rendered to the user when they click on the 'Published HTML' tab parallel to the Summary tab. This code is generic enough to accept any type of content(js, html, css etc) and append that inline in index.html. 

#### There are 3 important files which will need customization if you want to include any other report type:

 ##### 1. publishhtmlreport/index.ts
 This is a typescript while which contains the logic of rendering js files and content based on 'htmlType' arguement provided to publishhtmlreport@1 task. While customizing the extension, we can add one more accepted value of htmlType, for e.g. locust and render the required files using consol.log as done for jmeter reports in the present code base.

##### 2. tab.ts
This is a typescript file which contains the code for fetching the published files from the azure devops task and inserting the fetched content into index.html (which is the root html which gets loaded when the tab for the extension is clicked). 

##### 3. index.html
This is the root html of the extension, when a user clicks on 'Published HTML' within azure devops, this html will get loaded. So anything which needs to be rendered to the user should be part of this html. There are several ways to achieve this, replacing div ids, appending to div ids, class ids etc. I have used appendChild() in most situations, but during customization anything which works and appends the into the index.html during run time can be used. 

And when the above 3 files are changed, custom implementation type can be mentioned in the 'htmlType' arguement and the extension will render whatever was appended in the root html page. For e.g. :

```
- task: publishhtmlreport@1
  inputs:
    htmlType: 'OtherComplexReport'
    JmeterReportsPath: '$(Build.SourcesDirectory)/Folder_Containing_Relevant_Files'
```
Relevant files can be html, css or js files which can be appended in the html page using appendChild() or similar implementation. 

## Build Process of the extension.

Before building the extension, you need to have docker engine installed on your local machine or if you are doing a build in a CI pipeline, docker engine needs to be installed on the pipeline agent too. 

Steps to build:

### 1. Take a git clone of this repo:
```
git clone https://github.com/lakshaykaushik/PublishHTMLReport.git

```

### 2. Specify version of Jmeter in bootstrap.sh, run.sh and Dockerfile. Then run bootstrap.sh 

Running this script will bootstrap your local repo and include essential files needed to build the extension.

### 3. Now increment the build version in task.json and vss-extension.json. 

```
"manifestVersion": 1,
    "id": "PublishHTMLReports",
    "version": "1.1.33",
    "name": "PublishHTMLReports",
    "description": "An extension which lets you publish and visualize HTML reports in Azure Devops as a seperate tab",
    "publisher": "LakshayKaushik",
```
```
 "name": "publishhtmlreport",
    "friendlyName": "publishhtmlreport",
    "description": "This task can be used to publish html reports to azdo. Currently Jmeter HTML reports are being transformed to be consumed into Azdo directly",
    "helpMarkDown": "",
    "category": "Utility",
    "author": "LakshayKaushik",
    "version": {
        "Major": 1,
        "Minor": 1,
        "Patch": 25
```
        

### 4. Generate vsix file.

Run ```publishhtmlreport/tsc``` and ```npm run build```, this will generate vsix file which can be uploaded to the marketplace.

## Contributing to the extension

This project welcomes contributions and suggestions. 

## Communication

[![Gitter](https://badges.gitter.im/PublishHTMLReports/community.svg)](https://gitter.im/PublishHTMLReports/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

I am active on gitter for this extension, message me here, if you need some kind of support for this extension: https://gitter.im/PublishHTMLReports/community?utm_source=share-link&utm_medium=link&utm_campaign=share-link


