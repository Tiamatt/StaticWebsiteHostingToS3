# AWS project - Module 4. Use CloudFront distribution to serve a Static Website Hosted on AWS S3 via CloudFormation

## **Overview**

In the past three modules, we successfully created a basic static web app and deployed it on an S3 bucket. We also set up a CodeBuild project to automate the website's build process on AWS S3. Additionally, we configured Route 53 to enable the use of a custom domain for our S3-hosted static website. Now, let's continue enhancing our architecture further.

In this module we are going to optimize our application’s performance and security while effectively managing cost by setting up Amazon CloudFront to work with your S3 bucket to serve and protect the content. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i6e5vmb0yujnubxh1gqg.jpg)

## **Agenda**

In this module I'll show you how to configure CloudFront distribution to serve a simple static website in ten steps via CloudFormation:
✳️ Step 1. Register a custom domain name
✳️ Step 2. Create S3 buckets for your root domain and subdomain
✳️ Step 3. Create Route 53 hosted zone and configure your domain
✳️ Step 4: Request public SSL/TLS certificate from AWS Certificate Manager (ACM) for our domain name and all its subdomains 
✳️ Step 5. Create an Origin Access Identity (OAI) as special CloudFront user
✳️ Step 6. Create CloudFront distribution for subdomain that contains static website
✳️ Step 7. Create a policy for S3 bucket for subdomain to allow OAI to access bucket content
✳️ Step 8. Create another CloudFront distribution for root domain that redirects requests to S3 bucket for subdomain
✳️ Step 9. Create a Route 53 record set to route DNS traffic for root domain and subdomain to CloudFront domain
✳️ Step 10. Test your static website

## **Architecture**

The high-level architecture for our project is illustrated in the diagram below:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q310reo394suguhddayx.png)

## **Source code**

Source code for this project is available on my GitHub profile in a public repository named **StaticWebsiteHostingToS3** (check it [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3))
:point_right: Frontend  source code either [simple project](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/cloudformation/module4/frontend) or [Angular project](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/frontend/app-for-aws)
:point_right: Module 4 CloudFormation templates [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/cloudformation/module4)

List of CloudFormation templates:
:one: Template for S3 buckets [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/1-s3-buckets-stack.yaml)
:two: Template for Route 53 hosted zone [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/2-route53-hosted-zone-stack.yaml)
:three: Template for ACM certification [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/3-acm-certification-stack.yaml)
:four: Template for CloudFront distribution and OAI [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/4-cloudfront-stack.yaml)
:five: Template for Route 53 record set [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/5-route53-record-set-stack.yaml)



## **Initial Setup**

Create required accounts:
:point_right: AWS account
:point_right: GitHub account

Install required tools locally: 
:point_right: any IDE (personally I prefer Visual Studio Code) or any text editor 
:point_right: git


## **AWS Resources**

Here is the list of AWS resources that we are going to provision with CloudFormation:
:point_right: S3 buckets
:point_right: S3 bucket policy
:point_right: ACM certificate
:point_right: CloudFront OAI
:point_right: CloudFront distributions
:point_right: Route53 hosted zone
:point_right: Route53 record set group


## **Why use CloudFront?**

Amazon CloudFront is a **CDN (Content Delivery Network)** provided by AWS team. CDNs are primarily used for caching, and many customers also use AWS CloudFront as a security layer, or use it to handle network spikes. 

With CloudFront, when a user requests a webpage or an image, the request is routed to one of Amazon’s 400+ **edge locations** (caching servers). If the edge server already has the resource cached, it’s served to the client. If the resource isn’t on the edge server, it makes a request to an origin server that hosts your content (S3 bucket in our case).

The edge server then saves a copy of the response locally so it can handle the next request without pestering the origin server. 

This reduces load on the origin server, helping you keep the instance housing your application small, and is able to reduce latency for clients by moving commonly requested resources closer to the requestor. 

"Amazon S3 and CloudFront — is a match made in the cloud!" it said. Using CloudFront for our project offers three primary benefits: enhanced security, improved performance, and cost efficiency.

#### ✳️ Enhanced security

If you followed [Module 3](https://dev.to/tiamatt/aws-project-module-3-use-your-custom-domain-for-static-website-on-aws-s3-via-route-53-and-cloudformation-34cn), you may have noticed that in order to access our website hosted on S3 bucket, we granted public read access to it:

```YAML
 # Create a policy for myS3BucketForRootDomain to let Route53 access website content
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

(check the whole template [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module3/1-static-website-hosting-stack-v2.yaml#L94))

However it's always **a best practice to keep your S3 buckets private** to ensure security. So, a question arises: How can you host the static content of your website from **a private** S3 bucket? Well, CloudFront comes to the rescue. Later on in this module you can witness firsthand that with a CloudFront Distribution it is possible to serve content from a private S3 bucket.

Moreover, by configuring HTTPS with CloudFront, you establish secure end-to-end connections to your origin servers. For this purpose we will request public SSL/TLS certificate from AWS Certificate Manager (ACM) for our domain name and all its subdomains.

#### ✳️ Improved performance

The broad network of edge locations and CloudFront caches copies of content close to the end users that results in lowering latency, high data transfer rates and low network traffic. All these make CloudFront fast. 

Here is how it's going to work in our case:

A user requests website URL such as `www.example.com` which we will host on S3 bucket. The request will be routed to our CloudFront Distribution by Route 53. Now, if the requested content can be served from the cache it will be delivered immediately from the edge location closest to the end users. If the content is not cached (in case of initial request or when cache is expired), CloudFront requests the content directly from S3 bucket.

#### ✳️ Cost efficiency

For small data volumes, using S3 is cost-effective due to the free monthly transfer of the first GB. However, as data usage increases, CloudFront becomes more cost-efficient. The pricing difference for data transfer costs widens significantly when transferring terabytes of data, making CloudFront a more favorable option for content delivery compared to serving content directly from S3.

> There is an excellent article "AWS CloudFront Pricing and Cost Optimization Guide" I highly recommend reading ([link](https://www.cloudforecast.io/blog/aws-cloudfront-pricing-and-cost-guide/)).

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e3ze6z8x6vijduac61c4.jpg)



## **Step 1. Register a custom domain name**

In [Module 3](https://dev.to/tiamatt/aws-project-module-3-use-your-custom-domain-for-static-website-on-aws-s3-via-route-53-and-cloudformation-34cn) we covered the topic of domain name and Route 53. If you haven't had the chance to go through Module 3 and confused why use domain name, how to register it, and why use Route 53 with Amazon S3 website endpoint, please refer to Steps 1 and 4 in the following [link](https://dev.to/tiamatt/aws-project-module-3-use-your-custom-domain-for-static-website-on-aws-s3-via-route-53-and-cloudformation-34cn).

## **Step 2. Create S3 buckets for your root domain and subdomain**

:pushpin: When you configure an Amazon S3 bucket for website hosting, you must **give the bucket the same name as root domain/subdomain name** that you want to use to route traffic to the bucket. For example, if you want to route traffic for `www.example.com` to an S3 bucket that is configured for website hosting, the name of the bucket must be `www.example.com`.

Once you registered a new domain name, we can use it to provision an S3 bucket with the same name.

> Get the full CloudFormation template for S3 buckets from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/1-s3-buckets-stack.yaml)

1️⃣ The following piece of code helps to create a new S3 bucket for subdomain (such as `www.example.com`) and configure it to host a static website:

```YAML
Resources:
  myS3BucketForSubdomain:
    Type: 'AWS::S3::Bucket'
    DeletionPolicy: Retain # keep S3 bucket when its stack is deleted
    Properties:
      BucketName: my-subdomain.my-domain-name # use the name of subdomain with domain, such as www.example.com
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html
      VersioningConfiguration: # turn versioning on in case we need to rollback newly built files to older version
        Status: Enabled
      AccessControl: BucketOwnerFullControl
```

This S3 bucket for subdomain will contain our static website files. Don't forget to replace `my-subdomain.my-domain-name` with your root domain name with subdomain (such as `www.example.com`).

2️⃣ If you also want your users to be able to use root domain (such as `example.com`), to access your static website, create a second S3 bucket.

📌 Our second (root domain) bucket will not host a static website. We will keep it empty and configure it to redirect the request to the first bucket.

The following piece of code helps to create the second S3 bucket for root domain (such as `example.com`) and set it up to redirect requests to S3 bucket for subdomain (such as from `example.com` to `www.example.com`):

```YAML
Resources:
  myS3BucketForRootDomain:
    Type: 'AWS::S3::Bucket'
    DeletionPolicy: Retain # keep S3 bucket when its stack is deleted
    Properties:
      BucketName: my-domain-name # use the name of your domain, such as example.com
      WebsiteConfiguration:
        RedirectAllRequestsTo: # Configure the bucket to route traffic to the subdomain bucket
          HostName: !Ref myS3BucketForSubdomain
          Protocol: https
      AccessControl: BucketOwnerFullControl
```
Don't forget to replace `my-domain-name` with your root domain name (such as `example.com`).

📌 Note: keep both buckets private. Lets leverage CloudFront to secure our S3 buckets and let an Origin Access Identity (OAI) - which is a special CloudFront user - to access S3 buckets safely. A new policy for S3 bucket (for subdomain only) will be created later on, in CloudFormation template for CloudFront ([link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/4-cloudfront-stack.yaml#L114)) to let CloudFront OAI access content from a private S3 bucket.

3️⃣ We are done with Resources, now let's output Amazon S3 website endpoint and regional domain names for our buckets - we are going to use them to configure CloudFront later in this module.

```YAML
Outputs:
  outputS3WebsiteURLForRootDomain:
    Description: Amazon S3 website endpoint for root domain
    Value: !GetAtt myS3BucketForRootDomain.WebsiteURL
  outputS3RegionalDomainNameForRootDomain:
    Description:  Regional domain name of S3 bucket for root domain
    Value: !GetAtt myS3BucketForRootDomain.RegionalDomainName 
  outputS3WebsiteURLForSubdomain:
    Description: Amazon S3 website endpoint for subdomain
    Value: !GetAtt myS3BucketForSubdomain.WebsiteURL
  outputS3RegionalDomainNameForSubdomain:
    Description:  Regional domain name of S3 bucket for subdomain
    Value: !GetAtt myS3BucketForSubdomain.RegionalDomainName  
```

4️⃣ We are ready to create and run CloudFormation stack based on our template.

> If you need to understand how to create a stack in AWS Console, please read [Hands-on AWS CloudFormation - Part 1. It All Starts Here](https://dev.to/tiamatt/hands-on-aws-cloudformation-part-1-it-all-starts-here-5153).

Upload our template file for S3 buckets ([link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/1-s3-buckets-stack.yaml)) to AWS CloudFormation to create a stack. Specify your domain name and subdomain as parameters values. (I'm going to use domain name I've already registered, which is `tiamatt.com`.) Run the stack.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xvdnspvyy0wwstyhbnny.png)

Once the stack built, you should see two newly created buckets and the policy under _Resources_ tab:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cbd52y2da0ttwn0ubksh.png)

Under _Outputs_ tab you can find website endpoint and regional domain names for both domain and subdomain:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/uk3h6gr0444xdkmcqh6l.png)

5️⃣ Currently, if you navigate to `http://my-subdomain.my-domain-name.com.s3-website-us-east-1.amazonaws.com` using Amazon S3 website endpoint for _subdomain_ then you should get 403 Forbidden page as both our S3 buckets are private.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jomi0vf7lzbpwemfl8ju.png)

If you navigate to `http://my-domain-name.com.s3-website-us-east-1.amazonaws.com` using Amazon S3 website endpoint for _root domain_ then it should redirect your request to Amazon S3 website endpoint for _subdomain_.

:six:  To deploy the static content to our S3 bucket for subdomain you can:
👉 either open S3 Console and manually upload both [index.html](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/frontend/index.html) and [error.html](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/frontend/error.html) files 
👉 or automate the build of a static website ([Angular project](https://github.com/Tiamatt/StaticWebsiteHostingToS3/tree/main/frontend/app-for-aws)) on AWS S3 using CodeBuild (please follow the steps outlined in [Module 2](https://dev.to/tiamatt/aws-project-module-2-automate-the-build-of-a-static-website-on-aws-s3-via-codebuild-and-cloudformation-nc2))



## **Step 3. Create Route 53 hosted zone and configure your domain**

In [Module 3](https://dev.to/tiamatt/aws-project-module-3-use-your-custom-domain-for-static-website-on-aws-s3-via-route-53-and-cloudformation-34cn) we also covered the topic of Route 53 hosted zone. Please follow Step 4 in the following [link](https://dev.to/tiamatt/aws-project-module-3-use-your-custom-domain-for-static-website-on-aws-s3-via-route-53-and-cloudformation-34cn) to configure a hosted zone for your domains.

> Get the full CloudFormation template for Route 53 hosted zone from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/2-route53-hosted-zone-stack.yaml)



## **Step 4. Request public SSL/TLS certificate from AWS Certificate Manager (ACM) for our domain name and all its subdomains**

In the previous three modules, you may have observed that both our Amazon S3 website endpoint (e.g., `http://example.s3-website-us-east-1.amazonaws.com`) and the custom domain name URL (e.g., `http://example.com`) were using an unsecured HTTP protocol. However it's essential to always protect your websites with HTTPS, even if they don't handle sensitive information. 

📌 Note: HTTPS is crucial for protecting the communication between your websites and users' browsers. It prevents intruders, including malicious attackers and intrusive companies, from tampering with the data. Intruders can exploit unprotected communications to deceive users, steal sensitive information, install malware, or inject unwanted ads. They can exploit any resource, such as images, cookies, scripts, or HTML, that travels between your websites and users. These intrusions can happen anywhere in the network, like a user's device, a Wi-Fi hotspot, or a compromised ISP.

Basically  **HTTPS** (Hypertext Transfer Protocol Secure) is a **combination of the HTTP with the SSL/TLS** (Secure Socket Layer / Transport Layer Security) protocol.

AWS provides **AWS Certificate Manager (ACM)** service that handles the complexity of creating, storing, and renewing public and private SSL/TLS certificates and keys that protect your AWS websites and applications. You can provide certificates for your integrated AWS services either by issuing them directly with ACM or by importing third-party certificates into the ACM management system.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8d1jygwkm8wp923tgmgy.png)

Source: [IT Cooking website](https://www.it-cooking.com/tag/ssl/)

Now, let's request public SSL/TLS certificate from AWS Certificate Manager (ACM) for our domain name and all its subdomains to use to secure network communications and establish the identity of websites over the Internet.

> Get the full CloudFormation template for ACM certification from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/3-acm-certification-stack.yaml)

1️⃣ The following piece of code helps to create a request:

```YAML
Resources:
  mySSLCertificate:
    Type: 'AWS::CertificateManager::Certificate'
    Properties:
      DomainName: example.com
      SubjectAlternativeNames:
        - *.example.com # request a wildcard certificate for all subdomains
      DomainValidationOptions:
        - DomainName: example.com # DNS record for the root domain
          HostedZoneId: your-HostedZoneId
      ValidationMethod: DNS
```
Before the ACM can issue a certificate for your site, it must verify that you own or control all of the domain names that you specified in your request. You can choose either **email validation** or **DNS validation** when you request a certificate. Let's go with `DNS validation`.

2️⃣ We need to specify certificate ARN as output as we are going to use it during creation of CloudFront distributions (as will be mentioned in Step 8 of this module):

```YAML
Outputs: 
  outputCertificateArn:
    Description: Issued SSL certificate Arn
    Value: !Ref mySSLCertificate
```

3️⃣ Upload our template file for ACM certification [link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/3-acm-certification-stack.yaml) to AWS CloudFormation to create a stack.

Once the stack built (you need to wait for a while to let ACM verify your domain/subdomain names), under _Resources_ tab you should see your SSL certificate:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fusl06eqdemkl9a8nh4t.png)

Under _Outputs_ tab you can find certificate ARN:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/52zfmomlwivmm4p3x6b1.png)

If you navigate to AWS Certificate Manager (ACM), then go to _List certificates_,  you will see your newly created certificate and domains it covers - `example.com` and `*.example.com`:


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bltqpwyqtrkohk1t5zfw.png)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ny5ulqwm4p6z7tbetxyc.png)

## **Step 5. Create an Origin Access Identity (OAI) as special CloudFront user**

As we mentioned before, we kept our S3 buckets private which means buckets don't have any public access. Thus in order to let CloudFront access static content from S3 bucket, we must either set an Origin Access Identity (OAI) or permit public access to S3 bucket. However, the latter choice poses a significant security risk, which is why we prefer to opt for the OAI approach.

Basically, Origin Access Identity (OAI)  as special CloudFront user that helps to prevent others from viewing your S3 content by simply using the direct URL for the content. By using OAI, we can prevent others from accessing the files using Amazon S3 URLs.

> Get the full CloudFormation template for CloudFront from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/4-cloudfront-stack.yaml)

The following piece of code helps to create an OIA:

```YAML
Resources:
  myCloudFrontOAI:
    Type: 'AWS::CloudFront::CloudFrontOriginAccessIdentity'
    Properties:
      CloudFrontOriginAccessIdentityConfig:
        Comment: 'OAI for S3 origins'  # a comment to describe the origin access identity
```

Easy peasy!



## **Step 6. Create CloudFront distribution for subdomain that contains static website**

Now buckle up for creating CloudFront distribution!

> Get the full CloudFormation template for CloudFront from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/4-cloudfront-stack.yaml)

The following piece of code helps to create a CloudFront distribution for subdomain:

```YAML
Resources:
  myCloudFrontDistributionForSubdomain:
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Comment: CloudFront distribution points to S3 bucket for subdomain
        Origins: # info about origins for this distribution
          - DomainName: my-subdomain.my-domain-name.s3.${AWS::Region}.amazonaws.com' # Regional domain name of S3 bucket for subdomain (outputS3RegionalDomainNameForSubomain)
            Id: 'S3Origin-my-subdomain-my-domain-name' # unique identifier of an origin access control for this origin
            S3OriginConfig:
              OriginAccessIdentity: !Sub 'origin-access-identity/cloudfront/${myCloudFrontOAI}'
        Aliases: # info about CNAMEs (alternate domain names), if any, for this distribution
          - my-subdomain.my-domain-name
        # let CloudFront replace HTTP status codes in the 4xx and 5xx range with custom error messages before returning the response to the viewer
        CustomErrorResponses:
          - ErrorCode: 403 # 403 from S3 indicates that the file does not exists
            ResponseCode: 404 # HTTP status code that you want CloudFront to return to the viewer along with the custom error pag
            ResponsePagePath: '/error.html' # path to the custom error page that you want CloudFront to return to a viewer when your origin returns the HTTP status code specified by ErrorCode, for example, /4xx-errors/403-forbidden.html
            ErrorCachingMinTTL: 60 # minimum amount of time, in seconds, that you want CloudFront to cache the HTTP status code specified in ErrorCode
        DefaultCacheBehavior:
          AllowedMethods:
            - GET
            - HEAD
            - OPTIONS
          CachedMethods:
            - GET
            - HEAD
            - OPTIONS
          Compress: true
          DefaultTTL: 3600 # in seconds, 1 hour
          ForwardedValues:
            QueryString: false
            Cookies:
              Forward: none
          MaxTTL: 86400 # in seconds, 24 hours
          MinTTL: 60 # in seconds, 1 min
          TargetOriginId: 'S3Origin-my-subdomain-my-domain-name'
          ViewerProtocolPolicy: 'redirect-to-https' # 'allow-all'
        DefaultRootObject: 'index.html' 
        Enabled: true # enable distribution
        HttpVersion: http2 # the maximum HTTP version(s) that you want viewers to use to communicate with CloudFront
        PriceClass: PriceClass_All # allowed values: PriceClass_100 | PriceClass_200 | PriceClass_All
        ViewerCertificate:
          AcmCertificateArn: my-ACM-certificate-Arn
          SslSupportMethod: sni-only
```

Let me clarify a few things.

We pointed **Origins** to S3 bucket for subdomain (that contains static website, such as `www.example.com`) from which CloudFront should get the files to distribute. Here we also specified newly created CloudFront OAI (see Step 5) under **S3OriginConfig**.

In **CustomErrorResponses** section we set up CloudFront to replace 403 HTTP status code (that indicates the file does not exists) with 404 code along with the custom error message specified in `/error.html` page before returning the response to the viewer. 

In **DefaultCacheBehavior** section we set up some cache parameters such as **AllowedMethods** that controls which HTTP methods CloudFront processes and forwards to our S3 bucket; **CachedMethods** that caches the responses to GET, HEAD, and OPTIONS requests; **ViewerProtocolPolicy** makes sure a viewer submits an HTTP request, CloudFront redirects the viewer to the new URL with HTTPS protocol.

Under **ViewerCertificate** property please use your ACM certificate ARN (as mentioned in Step 4).

## **Step 7. Create a policy for S3 bucket for subdomain to allow OAI to access bucket content**

While provisioning CloudFront distribution we associated our CloudFront OAI (as mentioned in Step 5) with the origin so that viewers can only access objects in our S3 bucket for subdomain (that contains static website, such as `www.example.com`) through CloudFront. However it alone is insufficient. To ensure proper access, we must also provide permission for CloudFront OAI to read the files stored in our S3 bucket for subdomain. This can be achieved by creating a bucket policy.

Additionally, for the sake of security, let's restrict any access to S3 bucket for non-SSL requests. This is an additional precautionary step as we have already configured the CloudFront distribution to automatically redirect non-SSL requests to HTTPS.

> Get the full CloudFormation template for CloudFront from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/4-cloudfront-stack.yaml)

The following piece of code helps to create a policy for S3 bucket for subdomain:

```YAML
Resources:
  myPolicyForS3BucketForSubdomain:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      Bucket: my-subdomain.my-domain-name
      PolicyDocument:
        Statement:
        - Action: 's3:GetObject'
          Effect: Allow
          Resource: 'arn:aws:s3:::my-subdomain.my-domain-name/*'
          Principal:
            CanonicalUser: !GetAtt myCloudFrontOAI.S3CanonicalUserId
        # deny access for non SSL access to S3 bucket
        - Sid: AllowSSLRequestsOnly 
          Effect: Deny
          Principal: '*'
          Action: 's3:*'
          Resource:
          - !Sub 'arn:aws:s3:::my-subdomain.my-domain-name'
          - !Sub 'arn:aws:s3:::my-subdomain.my-domain-name/*'
          Condition:
            Bool:
              'aws:SecureTransport': false
```

📌 Note, we don't need to create a policy for S3 bucket for root domain as we keep it empty and has already configured it to redirect the request to S3 bucket for subdomain.

## **Step 8. Create another CloudFront distribution for root domain that redirects requests to S3 bucket for subdomain**

This step is similar to Step 6 for creating a CloudFront distribution for a subdomain, but there is one important difference. It is important to note that there is a known issue with CloudFront to S3 redirects that can result in an "Access denied" error.

📌 Important! If you have configured S3 bucket to redirect the request to another bucket, **avoid using the regional domain name of the bucket** as the value of **DomainName** under **Origins** when configuring your CloudFront distribution. Instead, **use S3 website endpoint**, as it will save you a significant amount of time and effort.

```YAML
Resources:
  myCloudFrontDistributionForRootDomain:
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Origins:
          # - DomainName: my-domain-name.s3.${AWS::Region}.amazonaws.com # regional domain name of S3 bucket for root domain (outputS3RegionalDomainNameForRootDomain)
          - DomainName: my-domain-name.s3-website-${AWS::Region}.amazonaws.com' # use Amazon S3 website endpoint for root domain (outputS3WebsiteURLForRootDomain)
```

📌 Important! If you are using S3 website endpoint as the value of **DomainName** under **Origins** then make sure to set up  **CustomOriginConfig** property instead of **S3OriginConfig**. It resolves the following issue: _"Resource handler returned message: Invalid request provided: The parameter Origin DomainName does not refer to a valid S3 bucket."_

```YAML
Resources:
  myCloudFrontDistributionForRootDomain:
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Origins:
         - DomainName: ....
           CustomOriginConfig:
              HTTPPort: 80 # required
              HTTPSPort: 443 # required
              OriginProtocolPolicy: 'http-only'
```

> Get the full CloudFormation template for CloudFront from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/4-cloudfront-stack.yaml)

The following piece of code helps to create create a CloudFront distribution for root domain:

```YAML
Resources:
  myCloudFrontDistributionForRootDomain:
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Comment: CloudFront distribution points to S3 bucket for root domain
        Origins: # info about origins for this distribution
          # Important! Known issue with Cloudfront to s3 redirect giving Access denied error
          # see https://stackoverflow.com/questions/22740084/amazon-s3-redirect-and-cloudfront 
          # - DomainName: my-domain-name.s3.${AWS::Region}.amazonaws.com # Regional domain name of S3 bucket for root domain (outputS3RegionalDomainNameForRootDomain)
          - DomainName: my-domain-name.s3-website-${AWS::Region}.amazonaws.com # use Amazon S3 website endpoint for root domain (outputS3WebsiteURLForRootDomain)
            Id: 'RedirectS3Origin-my-domain-name' # unique identifier of an origin access control for this origin
            # Important, for Amazon S3 website endpoint as DomainName need to be configured as CustomOriginConfig and NOT as S3OriginConfig
            # It resolves the following issue: "Resource handler returned message: "Invalid request provided: The parameter Origin DomainName does not refer to a valid S3 bucket.""
            # see https://stackoverflow.com/questions/40095803/how-do-you-create-an-aws-cloudfront-distribution-that-points-to-an-s3-static-ho 
            # see https://github.com/hashicorp/terraform-provider-aws/issues/7847 
            CustomOriginConfig:
              HTTPPort: 80 # required
              HTTPSPort: 443 # required
              OriginProtocolPolicy: 'http-only' 
        Aliases: # info about CNAMEs (alternate domain names), if any, for this distribution
          - 'my-domain-name'
        # let CloudFront replace HTTP status codes in the 4xx and 5xx range with custom error messages before returning the response to the viewer
        CustomErrorResponses:
          - ErrorCode: 403 # 403 from S3 indicates that the file does not exists
            ResponseCode: 404 # HTTP status code that you want CloudFront to return to the viewer along with the custom error pag
            ResponsePagePath: '/error.html' # path to the custom error page that you want CloudFront to return to a viewer when your origin returns the HTTP status code specified by ErrorCode, for example, /4xx-errors/403-forbidden.html
            ErrorCachingMinTTL: 60 # minimum amount of time, in seconds, that you want CloudFront to cache the HTTP status code specified in ErrorCode
        DefaultCacheBehavior:
          AllowedMethods:
            - GET
            - HEAD
            - OPTIONS
          CachedMethods:
            - GET
            - HEAD
            - OPTIONS
          Compress: true
          DefaultTTL: 3600 # in seconds, 1 hour
          ForwardedValues:
            QueryString: false
            Cookies:
              Forward: none
          MaxTTL: 86400 # in seconds, 24 hours
          MinTTL: 60 # in seconds, 1 min
          TargetOriginId: 'RedirectS3Origin-my-domain-name'
          ViewerProtocolPolicy: 'redirect-to-https' # 'allow-all'
        DefaultRootObject: 'index.html' 
        Enabled: true # enable distribution
        HttpVersion: http2 # the maximum HTTP version(s) that you want viewers to use to communicate with CloudFront
        PriceClass: PriceClass_All # allowed values: PriceClass_100 | PriceClass_200 | PriceClass_All
        ViewerCertificate:
          AcmCertificateArn: !Ref paramACMCertificateArn
          SslSupportMethod: sni-only
```

The rest configurations are similar to configurations we applied for CloudFront distribution for subdomain (as mentioned in Step 6).

So far we have added the following resources to our CloudFront template ([link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/4-cloudfront-stack.yaml)):
👉 a CloudFront Origin Access Identity (OAI)
👉 a CloudFront distribution for subdomain
👉 a policy for S3 bucket for subdomain
👉 a CloudFront distribution for root domain

We are done with _Resources_, now let's output CloudFront distribution domain name for both root domain and subdomain that we are going to use as Route 53 targets (as will be mentioned in Step 9 of this module):

```YAML
Outputs:  
  outputCloudFrontDistributionForSubdomainId:
    Description: CloudFront distribution ID for subdomain
    Value: !Ref myCloudFrontDistributionForSubdomain
  outputCloudFrontDistributionDomainNameForSubdomain:
    Description: CloudFront distribution domain name for subdomain
    Value: !GetAtt myCloudFrontDistributionForSubdomain.DomainName
  outputCloudFrontDistributionForRootDomainId:
    Description: CloudFront distribution ID for root domain
    Value: !Ref myCloudFrontDistributionForRootDomain
  outputCloudFrontDistributionDomainNameForRootDomain:
    Description: CloudFront distribution domain name for root domain
    Value: !GetAtt myCloudFrontDistributionForRootDomain.DomainName  
```

We are ready to create and run CloudFormation stack based on our template.

> If you need to understand how to create a stack in AWS Console, please read [Hands-on AWS CloudFormation - Part 1. It All Starts Here](https://dev.to/tiamatt/hands-on-aws-cloudformation-part-1-it-all-starts-here-5153).

Upload our template file for CloudFront ([link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/4-cloudfront-stack.yaml)) to AWS CloudFormation to create a stack. Specify your ACM certificate ARN as a parameter value. Also specify your domain name and subdomain as parameters values. (I'm going to use domain name I've already registered, which is `tiamatt.com`.) Run the stack.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/elad2139vxz7285r1aey.png)

Once the stack built, you should see two newly created CloudFront OAI and distributions, and the policy for S3 bucket for subdomain under _Resources_ tab:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dwqmbw48oyu9b7tao83k.png)

Under _Outputs_ tab you can find two CloudFront distributions - for domain and subdomain:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g62yo3k6mewa2dvm6k8g.png)

We are done with configuring CloudFront resources, now let's move on Route 53 resources.



## **Step 9. Create a Route 53 record set to route DNS traffic for root domain and subdomain to CloudFront domain**

After you create a hosted zone for your domain, such as `example.com` (as mentioned in Step 3), you need to create records to tell the Domain Name System (DNS) how you want traffic to be routed for that domain.

> Get the full CloudFormation template for Route 53 record set from [here](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/5-route53-record-set-stack.yaml)

1️⃣ The following piece of code helps to create a Route 53 record set group to route DNS traffic to CloudFront domain for both - domain and subdomain:

```YAML
Resources:
  # create a Route 53 record set group to route DNS traffic to CloudFront domain for both - domain and subdomain
  myRoute53RecordSetGroup:
    Type: 'AWS::Route53::RecordSetGroup'
    Properties:
      Comment: Route53 record for CloudFront distributions for root domain and subdomain
      HostedZoneId: my-HostedZoneId
      RecordSets:
        # for CloudFront distributions subdomain (such as www.example.com)
        - Name: 'my-subdomain.my-domain-name.'  # keep the dot
          Type: A # 'A' routes traffic to an IPv4 address and some AWS resources. E.g. if the name of the hosted zone is 'example.com' and you want to use www.example.com to route traffic to your distribution, enter 'www.'
          AliasTarget:
              DNSName: !Ref paramCloudFrontDistributionDomainNameForSubdomain
              HostedZoneId: Z2FDTNDATAQYW2 # DONT change! It is a magical alphanumeric ID provided by AWS team for CloudFront distribution 
        - Name: 'my-domain-name.'  # keep the dot
          Type: A # 'A' routes traffic to an IPv4 address and some AWS resources. E.g. if the name of the hosted zone is 'example.com' and you want to use www.example.com to route traffic to your distribution, enter 'www.'
          AliasTarget:
              DNSName: !Ref paramCloudFrontDistributionDomainNameForRootDomain
              HostedZoneId: Z2FDTNDATAQYW2 # DONT change! It is a magical alphanumeric ID provided by AWS team for CloudFront distribution 
```

📌 Important! HostedZoneId specified under AliasTarget is different from hosted zone id that we have created in Route 53 for our domain (as mentioned in Step 3). Hosted zone id for CloudFront is a magical alphanumeric ID provided by AWS team. 

See the whole list of hosted zone ids for different AWS services [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-route53-aliastarget.html#cfn-route53-aliastarget-hostedzoneid).

2️⃣  Upload our template file for Route 53 record sets ([link](https://github.com/Tiamatt/StaticWebsiteHostingToS3/blob/main/cloudformation/module4/5-route53-record-set-stack.yaml)) to AWS CloudFormation to create a stack.

Once the stack built, you should see a newly created Route 53 record set group under _Resources_ tab:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/590vd3yozrvogew38jdg.png)

Under _Outputs_ tab you can find two URLs to our website - with domain name and subdomain:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iq96oka6305496yeyfyh.png)

Now we are done with provisioning of AWS resources. Our infrastructure is ready. Let's test our website.

## **Step 10. Test your static website**

To verify that the website is working correctly, open a web browser and browse to `http://my-domain-name` (such as `example.com`). I'm using my custom domain name I've already registered, which is `tiamatt.com`:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g1utpe9wyc9293566snm.png)

Please take note of the following:
👉 Your request was redirected from root domain (`example.com`) to subdomain (`www.example.com`)
👉 the website protocol is changed from HTTP to HTTPS
👉 `index.html` was added at the end (if you remove it, it still going to show index page by default)

Also if you try to use either Amazon S3 website endpoint for _root domain_ (such as `http://example.com.s3-website-us-east-1.amazonaws.com`)  or CloudFront distribution domain name for _root domain_ (such as `abcde12345.cloudfront.net`) then again you will be redirected to `https://www.example.com/`.

If you try to use Amazon S3 website endpoint for _subdomain_ (such as `http://www.example.com.s3-website-us-east-1.amazonaws.com`), you should be redirected to **403 Forbidden** page. 

For CloudFront distribution domain name for _subdomain_ (such as `abcde67890.cloudfront.net`) you should see the index page with CloudFront url:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gb09a7ic6oqed5xaiesl.png)

## **Cleanup**

You might be charged for running resources. That is why it is important to clean all provisioned resources once you are done with the stack. By deleting a stack, all its resources will be deleted as well.

:pushpin: Note: CloudFormation won’t delete an S3 bucket that contains objects. First make sure to empty the bucket before deleting the stack. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p6plfunlsi3rm1s7at90.png)

## **Summary**

Congratulations on reaching this milestone!

In this module, we optimized our application's performance and security while managing costs. Using an Infrastructure as Code approach through CloudFormation, we provisioned all the necessary AWS resources. This streamlined and automated the deployment process, ensuring consistency and ease of management. We set up Amazon CloudFront to work with our S3 bucket for content delivery and protection. The steps involved registering a custom domain, creating S3 buckets, configuring Route 53, obtaining SSL/TLS certificate, creating an Origin Access Identity (OAI), setting up CloudFront distributions, and routing DNS traffic using Route 53. These actions helped us achieve a more efficient and secure application setup.