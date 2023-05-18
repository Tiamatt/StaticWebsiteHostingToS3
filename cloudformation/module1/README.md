# AWS project - Module 1. Host a Static Website on AWS S3 via CloudFormation

## **Overview** 

In this module, we are going to deploy a simple static website to S3 bucket via AWS CloudFormation in three simple steps:
:point_right: Step 1. Create a simple static web app via Angular
:point_right: Step 2. Create S3 bucket and its policy, and configure it to host a static website
:point_right: Step 3. Build and deploy static website manually to newly created S3 bucket (optional, you can skip it and go to [Module 2](https://dev.to/tiamatt/aws-project-module-2-automate-the-build-of-a-static-website-on-aws-s3-via-codebuild-and-cloudformation-nc2) for build automation)

> In [next](https://dev.to/tiamatt/aws-project-module-2-automate-the-build-of-a-static-website-on-aws-s3-via-codebuild-and-cloudformation-nc2) module we will automate the deployment via CodeBuild.

As I’m strongly against managing environments manually and take Infrastructure as Code for granted (if we are not on the same page, I’d suggest you to read [this](https://www.simplethread.com/why-infrastructure-as-code/) article first), AWS CloudFormation can be a great choice to provision and manage the complete infrastructure and AWS resources in a text file. 

## **Architecture**

The high-level architecture for our project is illustrated in the diagram below:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kncof7jf0fdf4t15wji4.png)

## **Source code**

Source code for this project is available on GitHub in a public [StaticWebsiteHostingToS3](https://github.com/Tiamatt/StaticWebsiteHostingToS3) repository.
:point_right: Check for frontend  source code [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/frontend/app-for-aws)
:point_right: Check for CloudFormation template [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/cloudformation/module1)

## **Initial Setup**

Install required tools: 
:point_right: any IDE (personally I prefer Visual Studio Code)
:point_right: git

Optional:
I’m going to create a simple static website using Angular. You can use any other framework/library if you want.
:point_right: Angular, Node.js and NPM - v16 or greater
:point_right: Angular CLI 
```YAML
    # run the following command to install Angular CLI globally
    npm install -g @angular/cli
```

## **AWS Resources**

Here is the list of AWS resources that we are going to create:
:point_right: S3 bucket
:point_right: S3 bucket policy

## **Step 1. Create a simple static web app**

> Shortcut: get the source code for frontend [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/frontend/app-for-aws) (but you still need to install Angular to run it locally unless you want to skip manual build and go to [Module 2](https://dev.to/tiamatt/aws-project-module-2-automate-the-build-of-a-static-website-on-aws-s3-via-codebuild-and-cloudformation-nc2) for build automation).

Let’s create a static Angular app from scratch.

:one: Firstly, create a new folder and name it _StaticWebsiteHostingToS3_ which will store two folders:
- _cloudformation_ folder (stores CloudFormation templates)
- _frontend_ folder (stores static website source code)

```YAML
# create a parent folder 
mkdir StaticWebsiteHostingToS3
cd StaticWebsiteHostingToS3

# initialize a repository
git init

# create a folder that stores CloudFormation templates
mkdir cloudformation 

# create a folder that stores static website source code
mkdir frontend 
```

:two: Secondly, create an initial Angular project by simply running the following commands in terminal:

```YAML
# navigate to folder that should store frontend 
cd frontend

# create a new Angular project
ng new app-for-aws
# check ‘y’ and ‘SCSS’ to prompt questions such as:
# - Would you like to add Angular routing? (y/N) y
# - Which stylesheet format would you like to use? SCSS

# run the app locally
cd app-for-aws
ng serve
```

:three: Navigate to http://localhost:4200/ 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bdsc08nce8f0fixybguz.png)

You can customize the website as you wish. Now, let's provision some resources in AWS to host our website. 


## **Step 2. Create S3 bucket and its policy, and configure it to host a static website**

Amazon S3 is a perfect AWS service to host a static website.

In order to host our website on S3 bucket, let’s create and configure all required resources via CloudFormation.

> Shortcut: get the full CloudFormation template [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module1/static-website-hosting-stack-v1.yaml).

:one: The following piece of code helps to create a new S3 bucket and configure it to host a static website:

```YAML
Resources:
  myStaticWebsiteHostingBucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: your-bucket-name
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html
```

:two: Next step is to set S3 bucket permissions for website access.

When you configure a bucket as a static website, if you want your website to be public, you must disable block public access settings for the bucket and write a bucket policy that grants public read access.

That is why we are going to disable ACLs for our bucket under *PublicAccessBlockConfiguration* property and set the ownership of S3 bucket content to *Object writer* (i.e. AWS account that uploads an object owns the object and has full control over it).

```YAML
      OwnershipControls:
        Rules:
          - ObjectOwnership: ObjectWriter
      PublicAccessBlockConfiguration:
        BlockPublicAcls: false
        BlockPublicPolicy: false
        IgnorePublicAcls: false
        RestrictPublicBuckets: false
```

Basically we are disabling S3 Block Public Access settings to let users get a public access to the website.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3fdjb8r1ny01992ff9c7.png)

One issue I faced with while configuring access settings and building the stack was the following:

> Bucket cannot have public ACLs set with BlockPublicAccess enabled (Service: Amazon S3; Status Code: 400; Error Code: InvalidBucketAclWithBlockPublicAccessError)

The root of the issue was in using *AccessControl* property  with *PublicAccessBlockConfiguration* property in the template. It should work with existing buckets and fail with all new buckets.

FYI, AWS has changed the default settings of ACLs of S3 bucket:

> Starting in April 2023, Amazon S3 will introduce two new default bucket security settings by automatically enabling S3 Block Public Access and disabling S3 access control lists (ACLs) for all new S3 buckets. Once complete, these defaults will apply to all new buckets regardless of how they are created, including AWS CLI, APIs, SDKs, and AWS CloudFormation. These defaults have been in place for buckets created in the S3 management console since the two features became available in 2018 and 2021, respectively, and are recommended security best practices. There is no change for existing buckets. ([source](https://aws.amazon.com/about-aws/whats-new/2022/12/amazon-s3-automatically-enable-block-public-access-disable-access-control-lists-buckets-april-2023/))

To resolve the issue, I've commented out *AccessControl* property:

```YAML
      # AccessControl: PublicRead # throws an error: Bucket cannot
# have public ACLs set with BlockPublicAccess enabled
```
:three: Next step is to create an S3 bucket policy 

We need to give a public read access to S3 bucket objects. Here is the policy:  

```YAML
   myStaticWebsiteHostingBucketPolicy:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      Bucket: !Ref myStaticWebsiteHostingBucket
      PolicyDocument:
        Version: 2012-10-17
        Statement:
          - Sid: PublicReadForGetBucketObjects
            Effect: Allow
            Principal: '*'
            Action: 's3:GetObject'
            Resource: !Join 
              - ''
              - - 'arn:aws:s3:::'
                - !Ref myStaticWebsiteHostingBucket
                - /*
```

We can turn versioning on for our S3 bucket so that in case of bugs in new version we can easily roll back the deployment to previous version. Simply add _VersioningConfiguration_ under bucket's property:
```YAML
Resources:  
  myStaticWebsiteHostingBucket:
    Type: 'AWS::S3::Bucket'
    .... 
    Properties:
      VersioningConfiguration:
        Status: Enabled
```

:four: We are done with _Resources_, now let's output the website URL to make it easier to navigate using the link once the stack is built.

```YAML
Outputs:
  outputWebsiteURL:
    Value: !GetAtt 
      - myStaticWebsiteHostingBucket
      - WebsiteURL
    Description: Static website URL
```

:five: We are ready to create and run CloudFormation stack based on our template. 

If you need to understand how to create a stack in AWS Console, please read [Hands-on AWS CloudFormation - Part 1. It All Starts Here](https://dev.to/tiamatt/hands-on-aws-cloudformation-part-1-it-all-starts-here-5153).

I prefer to run the stack from AWS Console as it provides an end-to-end overview of a process flow. But you can always run a stack using the AWS CLI ([here is how](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-cli.html)).

Upload our template file to create a stack. Change the value of  _paramStaticWebsiteHostingBucketName_ input parameter to something unique. Then run the stack.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p8ed4fevt3c8zf6whgtb.png)

Once the stack built, you should see a newly created bucket and it's policy under _Resources_ tab:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3u5o6lvm93aasb9l05l6.png)

Under _Outputs_ tab you can find the link to our website. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2fyfq4up19xdxwaxmsvi.png)

Currently you should get 404 error as we haven't deployed static content to S3 bucket yet. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2c3bv5ss94znjpigex8l.png)



## **Step 3. Build and deploy static website manually to newly created S3 bucket**

> This step is optional - you can skip it and go to [Module 2](https://dev.to/tiamatt/aws-project-module-2-automate-the-build-of-a-static-website-on-aws-s3-via-codebuild-and-cloudformation-nc2) for build automation.

:one: First, we need to build the production version of our Angular app locally.

Open terminal for our StaticWebsiteHostingToS3 app. By default, _ng build_ command uses the production build configuration:

```YAML
cd frontend/app-for-aws
ng build
```

A new _dist_ folder should be autogenerated with static content.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/39ev7ch410x8v6w3g0dw.png)

:two: In AWS Console navigate to our S3 bucket and upload all the files from _dist/app-for-aws_ folder:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sx8f4otda63i0vvgskor.png)

Don't forget to click on "Upload" button:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0v6r40u61zqf5449x6ws.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5s3da03r7a8e17e0cmm9.png)

:three: Now refresh the website link - our Angular app should be up and running:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7u3hc8d451ub80quf7iw.png)

## **Cleanup**

You might be charged for running resources. That is why it is important to clean all provisioned resources once you are done with the stack. By deleting a stack, all its resources will be deleted as well.

Note: CloudFormation won’t delete an S3 bucket that contains objects. First make sure to empty the bucket before deleting the stack. Of course, you can automate the process by creating a Lambda function which deletes all object versions and the objects themselves. I'll try to cover that part in further article. 

## **Summary**
In this module, we have created a simple static web app and hosted it on S3 bucket. However we took baby steps to deploy our static content to S3 bucket _manually_.  Ideally we want to use a tool that would rebuild the source code *every time* a code change is pushed to the repository and deploy built files to S3 bucket *automatically*. In [Module 2](https://dev.to/tiamatt/aws-project-module-2-automate-the-build-of-a-static-website-on-aws-s3-via-codebuild-and-cloudformation-nc2) I'll show you step by step how to automate the build and deployment via AWS CodeBuild. 