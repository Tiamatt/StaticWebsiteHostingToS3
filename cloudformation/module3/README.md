# AWS project - Module 3. Use Your Custom Domain for Static Website Hosted on AWS S3 via Route 53 and CloudFormation


## **Overview**

In [Module 1](https://dev.to/tiamatt/aws-project-module-1-host-a-static-website-on-aws-s3-via-cloudformation-2pa2), we have created a simple static website and hosted it on S3 bucket. To access our website we used Amazon S3 website endpoint which looked like `http://example.s3-website-us-east-1.amazonaws.com`. However for users the link is too long and hard to remember. If you want your website appear legitimate to visitors you should use a custom domain name such as `http://example.com`.

In this module I'll show you how to configure Route 53 to use your domain to access the website on S3 bucket in six simple steps via CloudFormation:
:eight_spoked_asterisk: Step 1. Register a custom domain name
:eight_spoked_asterisk: Step 2: Create an S3 bucket for your root domain and (optionally) for subdomain
:eight_spoked_asterisk: Step 3. Build and deploy static files to S3 bucket automatically using CodeBuild
:eight_spoked_asterisk: Step 4. Create Route 53 hosted zone and configure your domain
:eight_spoked_asterisk: Step 5. Create Route 53 record set to route your traffic for your domain to S3 bucket
:eight_spoked_asterisk: Step 6. Test your static website


## **Architecture**

The high-level architecture for our project is illustrated in the diagram below:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yq9z2zaddpr3p5b2d4x4.png)

In this module we will focus on configuring S3 buckets and Route 53 records so that:
:eight_spoked_asterisk: your users will be able to use domain name (such as `example.com`) to access your static website hosted on S3 bucket for root domain

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g00hpz8icffse4px3ygq.png)

:eight_spoked_asterisk: (optionally) your users will be able to use subdomain (such as `www.example.com`) to access your static website hosted on S3 bucket for root domain by redirecting here from S3 bucket for subdomain

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0tu8s40h7hwfx5ntc82l.png)


## **Source code**

Source code for this project is available on my GitHub profile in a public repository named **StaticWebsiteHostingToS3** (check it [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3))
:point_right: Frontend  source code [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/frontend/app-for-aws)
:point_right: Module 3 CloudFormation templates [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/cloudformation/module3)

List of CloudFormation templates:
:one: Template for S3 buckets [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/1-static-website-hosting-stack-v2.yaml)
:two: Template for CodeBuild project [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/2-codebuild-stack-v2.yaml)
:three: Template for Route 53 hosted zone [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/3-route53-hosted-zone-stack-v1.yaml)
:four: Template for Route 53 record set [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/4-route53-record-set-stack-v1.yaml)


## **Initial Setup**

Create required accounts:
:point_right: AWS account
:point_right: GitHub account

Install required tools locally: 
:point_right: any IDE (personally I prefer Visual Studio Code) or any text editor 
:point_right: git
:negative_squared_cross_mark: no need to install any package locally for our Angular project


## **AWS Resources**

Here is the list of AWS resources that we are going to provision with CloudFormation:
:point_right: S3 bucket(s)
:point_right: S3 bucket policy
:point_right: CodeBuild project
:point_right: IAM role for CodeBuild project
:point_right: GitHub Source Credential for CodeBuild project
:point_right: Route53 hosted zone
:point_right: Route53 record set group(s)


## **Why use domain name?**

Why should you get a custom domain? Let's break it down!

:ballot_box_with_check: **Brand Superpowers**: A custom domain name represents your brand and helps people easily recognize and remember you. Keep it consistent and unleash your brand's power.

:ballot_box_with_check: **Memorable Magic**: The secret to success is having a domain name that sticks in people's minds like a catchy tune. The easier it is to remember, the more people will visit your site.

:ballot_box_with_check: **Credibility Boost**: With a custom domain, your website becomes the ultimate credibility booster. It shows visitors that you mean business and adds a professional touch.

:ballot_box_with_check: **Authority Quest**: As your domain ages, it gains super search engine authority! By creating quality content and building links, you'll level up your domain's power over time.

:ballot_box_with_check: **Email Address**: The bonus perk of having a custom domain — your very own personalized email address! 

:ballot_box_with_check: **Portability**: With a custom domain, you can seamlessly move to different web hosting providers or website platforms while keeping the same web address. It's the ultimate flexibility that lets you adapt and expand your online presence without losing your trusted brand identity.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w9f6ta2vrggxjwir6geh.jpg)

Few things you should consider when coming up with a domain name: 
:point_right: Keep it short and sweet
:point_right: Make it unique
:point_right: Use easy to spell and pronounce name

## **Step 1. Register a custom domain name**

So you've decided to pay for a custom domain name. Congratulations on taking a significant first step towards establishing search authority, credibility, and trust!

To use a domain name (such as `example.com`), you must find a domain name that isn't already in use and register it. When you register a domain name, you reserve it for your exclusive use everywhere on the internet, typically for one year. (from AWS Developer Guide)

Buying a new domain generally costs between $10 and $20 a year. Price differences depend on which registrar you buy your domain name from, and what kind of domain you're buying. Different registrars offer different packages, so it's worth shopping around to find your best fit. 

:pushpin: Note: you can register a new domain with AWS, Google Domains, NameCheap, GoDaddy, HostGator,  or any other registrar. [Here](https://www.forbes.com/advisor/business/software/best-domain-registrar/) is the list of **Best Domain Registrar 2023** according to Forbes. 

Once you choose a domain registrar, all you need is simply to follow 3 general steps:

:eight_spoked_asterisk: Step 1. Choose a unique domain name
:eight_spoked_asterisk: Step 2. Choose a domain name registrar
:eight_spoked_asterisk: Step 3. Purchase and register your domain name

Follow **How To Register A Domain Name (2023 Guide)**  [link](https://www.forbes.com/advisor/business/how-register-domain-name/) for general instructions. 

Here are the links to some registrars:

:point_right: AWS using Amazon Route 53 [link](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/getting-started-s3.html#getting-started-find-domain-name)
:point_right: Google Domains [link](https://domains.google/) (my personal choice, I'm paying $12 annually)
:point_right: GoDaddy [link](https://www.godaddy.com/domains)
:point_right: HostGator [link](https://www.hostgator.com/domains)
:point_right: NameCheap [link](https://www.namecheap.com/domains/)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/d3zy03wfwld7kkfs3wof.png)


## **Step 2: Create an S3 bucket for your root domain and (optionally) for subdomain**

Let's take a step back to quickly recap domain name structure. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/se5dnlvsip6r2m0q756t.png)

Domain names have a specific structure that consists of several parts. Let's break it down:

:one: **Top-Level Domain (TLD):** This is the rightmost part of a domain name and represents the highest level in the domain name system hierarchy. Common examples include _.com_, _.org_, _.net_, _.edu_, _.gov_, etc.

:two: **Second-Level Domain (SLD):** The SLD is the part that comes before the TLD and is the main identifier of your website or brand. It typically represents the name of your organization, business, or the purpose of your website.

:three: **Subdomain:** A subdomain is an optional part that appears before the SLD. It allows you to organize and categorize different sections or services of your website. For example, in `blog.example.com`, "blog" is a subdomain.

:four: **Third-Level Domain and beyond:** It is possible to have additional levels of subdomains. For instance, `support.blog.example.com` has two levels of subdomains ("support" and "blog") before the SLD ("example") and the TLD (".com").

:pushpin: Note: **A domain name** _isn’t the same_ thing as a **uniform resource locator (URL)**. A URL is the full web address of a site, and while it does contain the domain name, it contains other information, too. Each URL includes the internet protocol (most commonly HTTP or HTTPS) being used to call up the page. URLs can also help point browsers to a specific file or folder being hosted on a web server (for example `https://www.my-site.com/blog?page=1`).

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8zkgt2q4mp8rh1max7a1.jpg)

Now enough theory, let's talk about implementation! 

In [Module 1](https://dev.to/tiamatt/aws-project-module-1-host-a-static-website-on-aws-s3-via-cloudformation-2pa2) we have discussed how to provision S3 bucket to host our static website. To configure it with Route 53 we need to make some adjustments. 

:pushpin: When you configure an Amazon S3 bucket for website hosting, you must **give the bucket the same name as the record** that you want to use to route traffic to the bucket. For example, if you want to route traffic for `example.com` to an S3 bucket that is configured for website hosting, the name of the bucket must be `example.com`.

Once you registered a new domain name, we can use it to provision an S3 bucket with the same name.  

> Get the full CloudFormation template for S3 buckets from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/1-static-website-hosting-stack-v2.yaml).

:one: The following piece of code helps to create a new S3 bucket and configure it to host a static website:

```YAML
Resources:
  myS3BucketForRootDomain:
    Type: 'AWS::S3::Bucket'
    DeletionPolicy: Retain # keep S3 bucket when its stack is deleted
    Properties:
      BucketName: my-domain-name # use the name of your domain, such as example.com
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html
      VersioningConfiguration: # turn versioning on in case we need to rollback newly built files to older version
        Status: Enabled
      # AccessControl: PublicRead # throws an error: Bucket cannot have public ACLs set with BlockPublicAccess enabled
      OwnershipControls:
        Rules:
          - ObjectOwnership: ObjectWriter
      PublicAccessBlockConfiguration:
        BlockPublicAcls: false
        BlockPublicPolicy: false
        IgnorePublicAcls: false
        RestrictPublicBuckets: false
```

This S3 bucket for root domain will contain our static website files. Don't forget to replace `my-domain-name` with your domain name (such as `example.com`).

We will push static files to S3 bucket for root domain with the help of CodeBuild in Step 3. 

Once we configure Route 53 in Step 4 and Step 5, your users will be able to use `my-domain-name` (such as `example.com`) to access your static website hosted on S3 bucket for root domain.

:two: If you also want your users to be able to use subdomain (such as `www.example.com`), to access your static website, create a second S3 bucket. 

:pushpin: Our second (subdomain) bucket will not host a static website. We will keep it empty and configure it to route traffic to the first bucket. 

The following piece of code helps to create the second S3 bucket to redirect the traffic to the first bucket:

```YAML
Resources:
  myS3BucketForSubdomain:
    Type: 'AWS::S3::Bucket'
    DeletionPolicy: Retain # keep S3 bucket when its stack is deleted
    Properties:
      BucketName:  my-subdomain.my-domain-name # use the name of subdomain with domain, such as www.example.com
      WebsiteConfiguration:
        RedirectAllRequestsTo: # Configure the bucket to route traffic to the S3 bucket for root domain
          HostName: !Ref myS3BucketForRootDomain
          Protocol: http
      AccessControl: BucketOwnerFullControl
```

:pushpin: Note: `RedirectAllRequestsTo` property of S3 bucket helps to configure second (subdomain) bucket to route traffic to the first (root domain) bucket.

You can use CloudFormation `Condition` function to provision second (subdomain) bucket only if a subdomain was specified as a parameter:

```YAML
Parameters:
  paramSubdomain:
    Description: OPTIONAL. Specify a subdomain (such as 'www' or 'apex' for www.example.com or apex.example.com). You can leave it empty to skip.
    Type: String
    Default: www

Conditions:
  # HasSubdomainName is false if paramSubdomain value is empty string
  HasSubdomainName: !Not [!Equals [!Ref paramSubdomain, '']] 

Resources:
 myS3BucketForSubdomain:
    Condition: HasSubdomainName # skip this resource if paramSubdomain value is empty string
    Type: 'AWS::S3::Bucket'

``` 

:three: The next step is to create a bucket policy for the first (root domain) bucket.

We need to give a public read access to S3 bucket objects. Here is the policy:  

```YAML
Resources:
  myPolicyForS3BucketForRootDomain:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      Bucket: !Ref myS3BucketForRootDomain
      PolicyDocument:
        Version: 2012-10-17
        Statement:
          - Sid: PublicReadForGetBucketObjects
            Effect: Allow
            Principal: '*'
            Action: 's3:GetObject'
            Resource: !Sub "${myS3BucketForRootDomain.Arn}/*"
```
:four: We are done with _Resources_, now let's output Amazon S3 website endpoint for both root domain and subdomain to make it easier to navigate using the link once the stack is built.

```YAML
Outputs:
  outputS3WebsiteURLForRootDomain:
    Description: Amazon S3 website endpoint for root domain
    Value: !GetAtt myS3BucketForRootDomain.WebsiteURL
  outputS3WebsiteURLForSubomain:
    Condition: HasSubdomainName
    Description: Amazon S3 website endpoint for subdomain
    Value: !GetAtt myS3BucketForSubdomain.WebsiteURL
```

:five: We are ready to create and run CloudFormation stack based on our template. 

> If you need to understand how to create a stack in AWS Console, please read [Hands-on AWS CloudFormation - Part 1. It All Starts Here](https://dev.to/tiamatt/hands-on-aws-cloudformation-part-1-it-all-starts-here-5153).

Upload our template file for S3 buckets ([link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/1-static-website-hosting-stack-v2.yaml)) to AWS CloudFormation to create a stack. Specify your domain name and subdomain as parameters values. (I'm going to use domain name I've already registered, which is `tiamatt.com`.) Run the stack.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ozzt6lr8wcrpjpc0npll.png)

Once the stack built, you should see two newly created buckets and the policy under _Resources_ tab:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2vph8o41euvbh7o6jpf2.png)

Under _Outputs_ tab you can find two URLs to our website - for domain and subdomain:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8p6zhwola9gg5stb7n8o.png)

Currently, if you navigate to `http://my-domain-name.com.s3-website-us-east-1.amazonaws.com` using Amazon S3 website endpoint for root domain then you should get 404 error as we haven't deployed static content to S3 bucket yet.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w3jqvmb2h6i4etu5p9hx.png)

To resolve this issue we need to deploy the static content to our S3 bucket for root domain (see Step 3).

If you navigate to `http://www.my-domain-name.com.s3-website-us-east-1.amazonaws.com` using Amazon S3 website endpoint for subdomain, then your link will be changed to `http://my-domain-name.com` with _This site can’t be reached_ error. To resolve this issue we need to configure Route 53 (see Step 4 and Step 5).

## **Step 3. Build and deploy static files to S3 bucket automatically using CodeBuild**

In [Module 2](https://dev.to/tiamatt/aws-project-module-2-automate-the-build-of-a-static-website-on-aws-s3-via-codebuild-and-cloudformation-nc2) we have discussed how to automate the build of a static website on AWS S3 via CodeBuild step by step. Please follow the guide to provision CodeBuild project and run CloudFormation stack for CodeBuild.

> Get the full CloudFormation template for CodeBuild project from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/2-codebuild-stack-v2.yaml).

:pushpin: Note: you don't need to know anything about Angular or install NodeJS, NPM and Angular CLI locally. CodeBuild project will automate the build of Angular project and safe built files for a static website on S3 bucket for root domain.

:one: When you run the stack, change the value of _paramS3BucketNameForRootDomain_ input parameter to name that you used building a stack for S3 bucket for root domain (such as `example.com`). Then run the stack. And don't forget to pass your GitHub access token as value of _paramPersonalGitHubAccessToken_ input parameter.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/54bhpasbqlujad8cthtz.png)

Once the stack built, you should see a newly created CodeBuild project and its role under _Resources_ tab:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yyei3vge9cfr5zx25qzl.png)

You can find our newly create CodeBuild project under CodeBuild -> Build projects:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xga7prg9ij60eilclexj.png)

:two: Update frontend source code and watch how it will be built automatically. 

Open any IDE or notepad on your machine and make some changes in `frontend\app-for-aws\src\app\app.component.html` file (it's totally up to you). I want to change the title to

```HTML
<!-- Resources -->
<h2>Good news, everyone! Testing Module 3!</h2>
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e7ndx2e370c4l91skm5y.png)

Push the changes to remote repo. Then navigate to our CodeBuild project on AWS Console, you should see that build was triggered automatically and its status is _in progress_. Wait till the status has changed to _succeeded_:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gxh8q2gnxdcgjrwwbu97.png)

Go to the browser and use Amazon S3 website endpoint for root domain to navigate to the website:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ny7emiz1jenou5ikcxu8.png)

:pushpin: Note: Amazon S3 website endpoint for subdomain will not work as we haven't configured Route 53 yet (see Step 4 and Step 5).


## **Step 4. Create Route 53 hosted zone and configure your domain**

:pushpin: Note:  When you register your domain using AWS registrar, Route 53 automatically creates a hosted zone with the same name. Skip this step if you have already registered your domain with AWS.

Let's take a step back to quickly recap what Route 53, hosted zones and records are.

**Route 53** is _Domain Name System (DNS) web service_ provided by AWS team. It offers domain registration, DNS routing, and health checking services (from AWS FAQs):

:point_right: With Route 53, you can create and manage your public DNS records. Like a phone book, Route 53 lets you manage the IP addresses listed for your domain names in the Internet’s DNS phone book. 

:point_right: Route 53 also answers requests to translate specific domain names like into their corresponding IP addresses like _192.0.2.1_. You can use Route 53 to create DNS records for a new domain or transfer DNS records for an existing domain. The simple, standards-based REST API for Route 53 allows you to easily create, update and manage DNS records. 

:point_right: Route 53 additionally offers _health checks_ to monitor the health and performance of your application as well as your web servers and other resources. You can also register new domain names or transfer in existing domain names to be managed by Route 53.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/b53c1lvpihpvxs7qz0cg.jpg)

In our case we need to configure DNS routing to transfer internet traffic from our domain (such as `example.com`) to Amazon S3 website endpoint (such as `http://example.s3-website-us-east-1.amazonaws.com`) to access the website on S3.

In order to configure our domain name to point to Amazon S3 website endpoint, we need to create a public hosted zone and add routing records there.

**Hosted zone** is an Amazon Route 53 concept. A hosted zone is analogous to a traditional DNS zone file; it represents a _collection of records_ that can be managed together, belonging to a single parent domain name. All resource record sets within a hosted zone must have the hosted zone's domain name as a suffix. For example, the `example.com` hosted zone may contain records named `www.example.com`, and `www.blog.example.com`, but not a record named `www.amazon.com` (from AWS FAQs). 

Each **record** contains information about how you want Route 53 to route traffic for a specific domain, and its subdomains. For example, you can specify a record to route your traffic for `example.com` to S3 bucket, or API Gateway, or Load Balancer, or another AWS resources.

> Get the full CloudFormation template for Route 53 hosted zone from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/3-route53-hosted-zone-stack-v1.yaml)

:one: The following piece of code helps to create a public hosted zone:

```YAML
Resources:
  myRoute53HostedZone:
    Type: 'AWS::Route53::HostedZone'
    Properties:
      HostedZoneConfig: 
        Comment: My public hosted zone for example.com
      Name: example.com
```

:pushpin: Note: A hosted zone and the corresponding domain should have **the same name**.

:two: Let's output hosted zone ID that we will use for Route 53 record set configuration. Also we can get the list of all name servers (NS) where the traffic will be routed to:

```YAML
Outputs:
  outputRoute53HostedZoneId:
    Description: Public hosted zone ID (such as Z23ABC4XYZL05B)
    Value: !GetAtt myRoute53HostedZone.Id # the same as !Ref myRoute53HostedZone  
  outputRoute53HostedZoneNameServers:
    Description: List of name servers for newly created public hosted zone
    Value: !Join [', ', !GetAtt myRoute53HostedZone.NameServers] 
```
:three: Upload our template file for hosted zone ([link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/3-route53-hosted-zone-stack-v1.yaml)) to AWS CloudFormation to create a stack.

Once the stack built, a new public hosted zone will be provisioned.

Under _Resources_ tab you can find the link to hosted zone:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gfeiv7syu6trtsl2lcqc.png)

Under _Outputs_ tab you can find the hosted zone ID for our domain and the list of all name servers (NS):

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/htenm1wq49gfs7nz3t11.png)

If you navigate to Route 53 -> Hosted zones, you will see your newly created hosted zone (such as `example.com`) with two records of type NS and SOA:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4lok93tl5v9wtnygvn1b.png)

Both **a name server (NS)** record and **a start of authority (SOA)** record are created by Route 53 automatically while provisioning a new public hosted zone. You rarely need to change these records.

:eight_spoked_asterisk: **Name server (NS) record** lists the four name servers that are the authoritative name servers for your hosted zone.
 
:eight_spoked_asterisk: **Start of authority (SOA) record** identifies the base DNS information about the domain.

:pushpin: Note: When you register your domain outside of AWS Route 53, you need to connect your DNS to Route 53.

As I registered my domain on Google Domains, I'll show you how I connected my Google domain to Route 53. Similar steps can be applied for other registrar platforms. 

:one: First, go to your Route 53 hosted zone and copy all four NS records from there:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nj11jjj989j3rd48wznc.png)

:two: Now, log into your Google Domains account. Then click on **My domains** -> select **DNS** from menu -> click on **Custom name servers (Active)** tab. Click on **Manage name servers**.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w17vqgaz4dvjojwv09ub.png)

Click on **Add another name server** and paste four NS records from the Route 53 Record Sets panel one by one. Don't forget to **Save** changes:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6hfx2hc3ck82sssj7aep.png)

Ta-da! Now our domain has been connected to Route 53.

## **Step 5. Create Route 53 record set to route your traffic for your domain to S3 bucket**
 
After you create a hosted zone for your domain, such as `example.com`, you need to create records to tell the Domain Name System (DNS) how you want traffic to be routed for that domain.

> Get the full CloudFormation template for Route 53 record set from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/4-route53-record-set-stack-v1.yaml)

:one: In our case we need to create a record to route internet traffic from root domain (such as `example.com`) to S3 bucket that hosts static website. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g00hpz8icffse4px3ygq.png)

The following piece of code helps to create a new record for S3 bucket for root domain:

```YAML
Resources:
  myRoute53RecordSetGroupForRootDomain:
    Type: 'AWS::Route53::RecordSetGroup'
    Properties:
      HostedZoneId: !Ref paramHostedZoneId
      RecordSets:
        - Name: example.com # point to an S3 bucket with root domain in the same account (such as example.com)
          Type: A # 'A' routes traffic to an IPv4 address and some AWS resources
          AliasTarget:
              DNSName: !Sub s3-website-${AWS::Region}.amazonaws.com
              HostedZoneId: !FindInMap # note, that it is different from paramHostedZoneId - this hosted zone is for region that you created the bucket in!
                - RegionMap
                - !Ref 'AWS::Region'
                - S3HostedZoneId
```

:pushpin: Note: **HostedZoneId** specified under **AliasTarget** is different from hosted zone id that we have created in Route 53 for our domain in Step 4. Hosted zone id for S3 is a magical alphanumeric ID provided by AWS team. It varies base on the region that you created the bucket in. That's why we used mapping to map S3 hosted zone id to region:

```YAML
Mappings:
  RegionMap: # based on https://docs.aws.amazon.com/general/latest/gr/s3.html#s3_website_region_endpoints
    us-east-1:
      S3HostedZoneId: Z3AQBSTGFYJSTF
    us-west-1:
      S3HostedZoneId: Z2F56UZL2M1ACD
    us-west-2:
      S3HostedZoneId: Z3BJ6K6RIION7M
    eu-central-1:
      S3HostedZoneId: Z21DNDUVLTQW6Q
    eu-west-1:
      S3HostedZoneId: Z1BKCTXD74EZPE
    ap-southeast-1:
      S3HostedZoneId: Z3O0J2DXBE1FTB
    ap-southeast-2:
      S3HostedZoneId: Z1WCIGYICN2BYD
    ap-northeast-1:
      S3HostedZoneId: Z2M4EHUR26P7ZW
    sa-east-1:
      S3HostedZoneId: Z31GFT0UA1I2HV
```
See the whole list of hosted zone ids for S3 [here](https://docs.aws.amazon.com/general/latest/gr/s3.html#s3_website_region_endpoints).

:pushpin: Also note for future reference that hosted zone id depends on your alias target. For example:
- for Global Accelerator accelerator specify Z2BJ6XQ5FK7U4H
- for  CloudFront distribution specify  Z2FDTNDATAQYW2
- etc

See the whole list of hosted zone ids for different AWS services [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-route53-aliastarget.html#cfn-route53-aliastarget-hostedzoneid).

:two: Optionally, we can create a new record for S3 subdomain  bucket which will redirect traffic to S3 bucket for root domain.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0tu8s40h7hwfx5ntc82l.png)

The following piece of code helps to create a new record for S3 bucket for subdomain:

```YAML
Resources:
  myRoute53RecordSetGroupForSubdomain:
    Condition: HasSubdomainName # skip this resource if paramSubdomain value is empty string
    Type: 'AWS::Route53::RecordSetGroup'
    Properties:
      HostedZoneId: !Ref paramHostedZoneId
      RecordSets:
        - Name: www.example.com # point to an S3 bucket with subdomain in the same account (such as www.example.com)
          Type: A # 'A' routes traffic to an IPv4 address and some AWS resources
          AliasTarget:
              DNSName: !Sub s3-website-${AWS::Region}.amazonaws.com 
              HostedZoneId: !FindInMap # note, that it is different from paramHostedZoneId - this hosted zone is for region that you created the bucket in!
                - RegionMap
                - !Ref 'AWS::Region'
                - S3HostedZoneId  
```
:three: Upload our template file for Route 53 record sets ([link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/4-route53-record-set-stack-v1.yaml)) to AWS CloudFormation to create a stack.

Once the stack built, you should see two newly created Route 53 record set groups under _Resources_ tab:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fv0g7lrepoko6v6vbqk0.png)

Under _Outputs_ tab you can find two URLs to our website - with domain name and subdomain:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3fcthp82aqknxut5lt5r.png)

Now we are done with provisioning of AWS resources. Our infrastructure is ready. Let's test our website.

## **Step 6. Test your static website**

To verify that the website is working correctly, open a web browser and browse to `http://my-domain-name` (such as `example.com`). I'm using my custom domain name I've already registered, which is `tiamatt.com`:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/389jxqix7t9bcubbhaf8.png)

Now, let's check our subdomain. Browse to `http://your-subdomain.my-domain-name` (such as `www.example.com`). You should see that it will redirect your request to your `http://my-domain-name` domain (such as `example.com`).


## **Cleanup**

You might be charged for running resources. That is why it is important to clean all provisioned resources once you are done with the stack. By deleting a stack, all its resources will be deleted as well.

:pushpin: Note: CloudFormation won’t delete an S3 bucket that contains objects. First make sure to empty the bucket before deleting the stack. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jchikoy7jp3z500x0t6e.jpg)

## **Summary**

Well done on reaching this point!

In this module we explained how to configure Route 53 to use a custom domain for your S3-hosted static website. Say goodbye to long, hard-to-remember links! In six simple steps, we registered the domain, set up S3 buckets, deployed files with CodeBuild, configured Route 53, and tested website. 

By using a custom domain, we can boost professionalism and make our site more accessible.


## **Useful links**

Articles:
- AWS Official - Use your domain for a static website in an Amazon S3 bucket - https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/getting-started-s3.html

- AWS Official -  What is Amazon Route 53? https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html 

- AWS Official - Route 53 template snippets https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-route53.html 

- AWS Official - Route 53 - Creating a public hosted zone https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/CreatingHostedZone.html 

- AWS Official - AWS::Route53::HostedZone https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-route53-hostedzone.html 

- Finding your registrar and other information about your domain - https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/find-your-registrar.html 

- How To Connect Google Domain to Route 53 https://www.entechlog.com/blog/aws/connect-google-domain-to-aws-route-53/ 

- Read this great article - AWS CloudFormation Doesn’t Do That Yet?! No Problem https://shouldroforion.medium.com/aws-cloudformation-doesnt-do-that-yet-no-problem-fdd1ff01839 