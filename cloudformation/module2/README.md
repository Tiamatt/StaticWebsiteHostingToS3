# AWS project - Module 2. Automate the build of a Static Website Hosted on AWS S3 via CodeBuild and CloudFormation

## **Overview**
In [Module 1](https://dev.to/tiamatt/aws-project-module-1-host-a-static-website-on-aws-s3-via-cloudformation-2pa2), we have created a simple static web app and hosted it on S3 bucket. However we took baby steps to deploy our static content to S3 bucket *manually*. Ideally we want to use a tool that would rebuild the source code *every time* a code change is pushed to the repository and deploy built files to S3 bucket *automatically*.

In this module I'll show you how to automate the build and deployment via AWS CodeBuild in six simple steps:
:point_right: Step 1. Create buildspec YAML file
:point_right: Step 2. Provide CodeBuild with access to GitHub repo
:point_right: Step 3. Configure how AWS CodeBuild builds your source code 
:point_right: Step 4. Create IAM role for CodeBuild project
:point_right: Step 5. Run the CloudFormation stack
:point_right: Step 6. Update frontend source code and watch how it will be built automatically

## **Architecture**
The high-level architecture for our project is illustrated in the diagram below:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jk5l77vr3ise73jk3zcb.png)

## **Source code**

Source code for this project is available on GitHub in a public [StaticWebsiteHostingToS3](https://github.com/Tiamatt/StaticWebsiteHostingToS3) repository.
:point_right: Check for frontend  source code [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/frontend/app-for-aws)
:point_right: Check for Module 2 CloudFormation templates [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/cloudformation/module2)

## **Initial Setup**

Install required tools: 
:point_right: any IDE (personally I prefer Visual Studio Code) or text editor
:point_right: git

Note, you don't need to know anything about Angular or install NodeJS, NPM and Angular CLI locally. We will configure CodeBuild project with all necessary installation packages.  

## **AWS Resources**

Here is the list of AWS resources that we are going to create:
:point_right: CodeBuild project
:point_right: IAM role for CodeBuild project
:point_right: GitHub Source Credential for CodeBuild project

## **Step 1. Create buildspec YAML file**

_Buildspec_ is a collection of build commands and related settings, in YAML format, that CodeBuild uses to run a build. We need to create and include a buildspec YAML file as part of the source code of frontend app. In our case the file is stored in the root directory of Angular app, which is _frontend/app-for-aws_ folder.

> Shortcut: get buildspec YAML file [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/frontend/app-for-aws/buildspec.yml).

:one: _Install_ phase of buildspec file 

Use the _install_ phase only for installing packages in the build environment. In order to run the production build of our Angular web app, CodeBuild server needs Angular installation. Meanwhile Angular depends on NodeJS and NPM. Thus we need to specify NodeJS installation as a runtime and add commands to install Angular CLI and node modules based on dependencies specified in _package.json_ ([link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/frontend/app-for-aws/package.json)):

```YAML
phases:
  install:
    runtime-versions:
       nodejs: 18
    commands:
      - echo Install Angular CLI and all node_modules 
      - cd $CODEBUILD_SRC_DIR/frontend/app-for-aws
      - npm install && npm install -g @angular/cli
``` 

:two: _Build_ phase of buildspec file 

Use the _build_ phase for commands that CodeBuild runs during the build. In our case we need to navigate to our frontend folder and run the production build:

```YAML
  build:
    commands:
      - echo Build process started now
      - cd $CODEBUILD_SRC_DIR/frontend/app-for-aws
      - ng build --configuration=production
```

:three: _Post build_ phase of buildspec file

Use the _post_build_ phase for commands that CodeBuild runs after the build. Here we can list all the files of newly created _dist/app-for-aws_ folder for easy debugging:

```YAML
post_build:
    commands:
      - echo Build process finished, upload artifacts to S3 bucket
      - cd dist/app-for-aws
      - ls -la
```

:four: _Artifacts_ of buildspec file

Artifacts represent information about where CodeBuild can find the build output and how CodeBuild prepares it for uploading to the S3 output bucket. Here we need to specify the path to _dist_ folder so that CodeBuild won't deploy a whole frontend project to S3 bucket but files for prod build only: 

```YAML
 artifacts:
  base-directory: 'frontend/app-for-aws/dist*'
  discard-paths: yes
  files:
    - '**/*'  
```
`discard-paths` helps to make sure to put all the files in the root folder instead of subfolder. In our case it will take all the files (including _index.hmtl_ file) from _dist/app-for-aws_ folder and put them in the root of S3 bucket.

## **Step 2. Provide CodeBuild with access to GitHub repo**

:one: In order to access source code for frontend located on your GitHub account, CodeBuild needs some access privilege. The easiest and safest way to grand CodeBuild an access to GitHub is to create a personal access token. 

Navigate to your GitHub and click your profile photo, then click **Settings**. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/uk47xvwvsifo3m91rdtf.png)

In the left sidebar, scroll down and click **Developer settings**.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/piriqhwxsxn9jh4do2ty.png)

In the left sidebar, under **Personal access tokens**, click **Tokens (classic)** (since _Fine-grained tokens_ is in beta and might not be compatible with AWS CodeBuild yet), then click **Generate new token**.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pp54wa2yqtkkxc38nt4z.png)

Select the scopes you'd like to grant this token. Note, as I've already generated a token, my screenshots shows _Edit_. Please ignore.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t16xidbrz31cmkkfahdi.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ofpurojlw44b301ph427.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u1oii2p8oz0d43jb1jmt.png)

Click **Generate token**. Copy the new token - we will need it while running a CloudFormation stack.

:two: In CloudFormation template we are going to create a new _AWS::CodeBuild::SourceCredential_ resource. Here we provide information about the credentials for a GitHub. 

It is recommend to use AWS Secrets Manager to store our credentials. But for the sake of simplicity (let's take baby steps) we will pass a newly generated GitHub access token as a CloudFormation parameter for now. 

```YAML
Parameters:
  paramPersonalGitHubAccessToken:
    Type: String
    MinLength: 10
    ConstraintDescription: Personal GitHub access token is missing
    Description: Provide your personal GitHub access token for 
CodeBuild to access your GitHub repo

Resources:
  myCodeBuildSourceCredential:
    Type: AWS::CodeBuild::SourceCredential
    Properties:
      AuthType: PERSONAL_ACCESS_TOKEN
      ServerType: GITHUB
      Token: !Ref paramPersonalGitHubAccessToken
```
## **Step 3. Configure how AWS CodeBuild builds your source code**

_AWS::CodeBuild::Project_ resource configures how AWS CodeBuild builds your source code, such as where to get the source code and which build environment to use.

First, let's give a friendly name and add description for our CodeBuild project:

```YAML
Resources:
  myCodeBuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      Name: westworld-codebuild-for-website-hosting
      Description: CodeBuild project for automatically build of static website hosted on s3
```

In **Source** code settings for the CodeBuild project we need to specify the _GITHUB_ as source code's repository type, then add repo location, BuildSpec location and authorization settings for AWS CodeBuild to access the source code to be built. Remember, in Step 2 of this module we have created `myCodeBuildSourceCredential` Source Credential resource - we need to use it under Auth Resource so that CodeBuild can access the specified GitHub repo.

```YAML
Resources:
  myCodeBuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      ...
      Source:
        Type: GITHUB
        Location: https://github.com/your-account/StaticWebsiteHostingToS3.git
        GitCloneDepth: 1
        BuildSpec: frontend/app-for-aws/buildspec.yml
        Auth:
          Resource: !Ref myCodeBuildSourceCredential
          Type: OAUTH
```

In **Triggers** code settings for the CodeBuild project we want to enable AWS CodeBuild to begin automatically rebuilding the source code every time a code change is pushed to the repository. Basically we configured CodeBuild to listen to any git pushes to _main_ branch to trigger the build.

```YAML
Resources:
  myCodeBuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      ...
      Triggers:
        Webhook: true
        FilterGroups:
          - - Type: EVENT
              Pattern: PUSH
            - Type: HEAD_REF
              Pattern: ^refs/heads/main # for feature branches use: ^refs/heads/feature/.* 
```

In **Environment** code settings for the CodeBuild project we basically configured VM where CodeBuild is going to build the source code. 

```YAML
Resources:
  myCodeBuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      ...
      Environment: # use Ubuntu standard v7
        Type: LINUX_CONTAINER
        ComputeType: BUILD_GENERAL1_SMALL
        Image: aws/codebuild/standard:7.0
```
But how do you know which container, computer and image you need? Well, based on the code source programming language and framework you can check for Docker images [here](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-available.html) and Available runtimes [here](https://docs.aws.amazon.com/codebuild/latest/userguide/available-runtimes.html).

As we have specified NodeJS v18 in our buildspec file, we can use either `Amazon Linux 2 AArch64 standard:3.0` or `Ubuntu standard:7.0` image. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rrd5y2mkbu9sxb52gt5e.png)

Based on selected image I can get _Image identifier_  from [this](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-available.html) list.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nzuhqqt75vmcghvfdwyi.png)

In **Artifacts** code settings for the CodeBuild project we want to specify what to do with build files. In our case we want to put them to our S3 bucket that hosts the static website. Note, it's very important to disable encryption for our website files. 

```YAML
Resources:
  myCodeBuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      ...
      Artifacts: # drop the build artifacts of S3 bucket that hosts static website
        Type: S3
        Name: '/' # store the artifact in the root of the output bucket
        Location: !Sub arn:aws:s3:::westworld-codebuild-for-website-hosting
        EncryptionDisabled: True #disable the encryption of artifacts in a build to see html pages
```

In **LogsConfig** code settings for the CodeBuild project we want to configure CloudWatch logs.

```YAML
Resources:
  myCodeBuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      ...
      LogsConfig:
        CloudWatchLogs:
          Status: ENABLED
          GroupName: westworld-codebuild-for-website-hosting-CloudWatchLogs
```

In **ServiceRole** code settings for the CodeBuild project we need to create a new role with proper access - see Step 4.

```YAML
Resources:
  myCodeBuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      ...
      ServiceRole: !Ref myCodeBuildProjectRole
```

## **Step 4. Create IAM role for CodeBuild project**

Final step for our CloudFormation template is to create an appropriate IAM role for CodeBuild project to get an access to S3 bucket where static website is hosted and to CloudWatch logs to stream the logs while building the project. 

```YAML
Resources:
  myCodeBuildProjectRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: role-for-westworld-codebuild-for-website-hosting
      Path: /
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Action:
              - 'sts:AssumeRole'
            Effect: Allow
            Principal:
              Service:
                - codebuild.amazonaws.com
      Policies:
        - PolicyName: policy-for-westworld-codebuild-for-website-hosting
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              # statement to create/stream CloudWatch
              - Action:
                  - logs:CreateLogGroup
                  - logs:CreateLogStream
                  - logs:PutLogEvents
                Effect: Allow
                Resource:
                  - !Sub arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:westworld-codebuild-for-website-hosting-CloudWatchLogs
                  - !Sub arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:westworld-codebuild-for-website-hosting-CloudWatchLogs:*
              # statement to access S3 bucket that hosts static website (CodeBuild will save Artifacts there)
              - Action:
                  - s3:PutObject
                  - s3:GetObject
                  - s3:GetObjectVersion
                  - s3:GetBucketAcl
                  - s3:GetBucketLocation
                Effect: Allow
                Resource:
                  - arn:aws:s3:::your-bucket-name
                  - arn:aws:s3:::your-bucket-name/*
```

## **Step 5. Run the CloudFormation stack**

Now we are ready to create and run CloudFormation stack based on our template for CodeBuild (see the whole template [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module2/2-codebuild-stack-v1.yaml)). 

Note, we have already created and run CloudFormation stack for static website and S3 - check [this](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module2/1-static-website-hosting-stack-v1.yaml) template for provisioning S3 bucket and step by step guide in [Module 1](https://dev.to/tiamatt/aws-project-module-1-host-a-static-website-on-aws-s3-via-cloudformation-2pa2).

I prefer to run the stack from AWS Console as it provides an end-to-end overview of a process flow. But you can always run a stack using the AWS CLI ([here is how](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-cli.html)).

Upload our template file to create a stack.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8mk2i52d1ud8nzp1i72c.png)

Change the value of  _paramStaticWebsiteHostingBucketName_ input parameter to name that you used building a stack for S3 bucket. Then run the stack. And dont forget to pass your GitHub access token as value of _paramPersonalGitHubAccessToken_ input parameter.

Once the stack built, you should see a newly created CodeBuild project and its role under _Resources_ tab:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2ha7wfkiabikiydpe5jb.png)

You can find our newly create CodeBuild project under *CodeBuild* -> *Build projects*:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p2k2aaukxdz8t2qmu0k8.png)

Let's navigate to our website. Remember, we have created our stack for S3 bucket - under *Outputs* tab you can find the link to our website (note, your link might be different from mine):

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y1n5vhh8iir73mb7tarm.png)

Voila! Our static website is up and running! It was built and deployed to S3 with the initial provisioning of our CodeBuild project.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lg6rjnqiyj7hag3lhekt.png)

## **Step 6. Update frontend source code and watch how it will be built automatically**

Now it's time to check out build automation. 

:one: Let's make a small change (it's totally up to you) in our source code for static website.  

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x9tsvbeqzrfet9f3orl4.png)

You need to clone frontend project (unless you want to use your own project built in React, Vue or vanilla JavaScript) - check for frontend  source code [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/frontend/app-for-aws).

Let's make some changes in `frontend\app-for-aws\src\app\app.component.html` file. I want to change the title from 
```HTML
<!-- Resources -->
<h2>Good news, everyone!</h2>
``` 
to 
```HTML
<!-- Resources -->
<h2>Good news, everyone! CodeBuild project is up and running!</h2>
```

Open html file in any IDE or text editor. Make and save the changes.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9f7y1rbayye7m34p9pwz.png)

Now let's push the changes to remote repo. Open your terminal and run the following commands:

```YAML
# navigate to the project
cd StaticWebsiteHostingToS3
# stage the changes
git add .
# commit the changes
git commit -m"Changed the title of static website"
# push the changes to remote repo
git push
```

:two: Once you pushed your changes to GitHub and navigate to our CodeBuild project on AWS Console, you should see that build was triggered automatically and its status is _in progress_:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mq1d2oy0kr5fpq9mcsgw.png) 

Click on Build run and you will see the logs:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lpceu7y3xofbauq3t5n8.png)

You can navigate to CloudWatch logs by clicking on _View entire log_ under _Build log_ tab:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/62ke8xnd39o80rcut90r.png)

Note, that all our logs for build were stored under newly created _westworld-codebuild-for-website-hosting-CloudWatchLogs_ log group.

Also you can check our S3 bucket to see if the files were updated (look at _Last modified_ date and time).

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/aa55pfvnw0pq4tmfwd67.png)

:three: Once build status has changed to _Succeeded_, go ahead and refresh your website link. You should the code changes.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gcpfocggm4vholuir86q.png)

Note, if you don't see the changes, probably the website was cached. You can either clean the cache or open the website in incognito mode. 

## **Cleanup**

You might be charged for running resources. That is why it is important to clean all provisioned resources once you are done with the stack. By deleting a stack, all its resources will be deleted as well.

Note: CloudFormation won’t delete an S3 bucket that contains objects. First make sure to empty the bucket before deleting the stack. Of course, you can automate the process by creating a Lambda function which deletes all object versions and the objects themselves. I'll try to cover that part in further article. 

## **Summary**

Congratulations on getting this far! 

In this module, we have provisioned CodeBuild project that automate the build of a static website on AWS S3. We took Infrastructure as Code approach to provisioning and managing our AWS resources by writing a template file and running stacks using AWS CloudFormation service.

Don't forget to do your happy dance!

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5lcqhw408jfcx408yzm5.gif)