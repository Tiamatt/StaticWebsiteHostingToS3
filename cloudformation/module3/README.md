Links:

Articles:
- AWS Official - Use your domain for a static website in an Amazon S3 bucket - https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/getting-started-s3.html

- AWS Official -  What is Amazon Route 53? https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html 

- AWS Official - Route 53 template snippets https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-route53.html 

- AWS Official - Route 53 - Creating a public hosted zone https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/CreatingHostedZone.html 

- AWS Official - AWS::Route53::HostedZone https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-route53-hostedzone.html 

- Finding your registrar and other information about your domain - https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/find-your-registrar.html 

- How To Connect Google Domain to Route 53 https://www.entechlog.com/blog/aws/connect-google-domain-to-aws-route-53/ 




Read this great article - AWS CloudFormation Doesn’t Do That Yet?! No Problem https://shouldroforion.medium.com/aws-cloudformation-doesnt-do-that-yet-no-problem-fdd1ff01839 



## Route 53
Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. You can use Route 53 to perform three main functions in any combination: domain registration, DNS routing, and health checking.

1. Register domain names
Your website needs a name, such as example.com. Route 53 lets you register a name for your website or web application, known as a domain name.
2. Route internet traffic to the resources for your domain
When a user opens a web browser and enters your domain name (example.com) or subdomain name (acme.example.com) in the address bar, Route 53 helps connect the browser with your website or web application.
3. Check the health of your resources
Route 53 sends automated requests over the internet to a resource, such as a web server, to verify that it's reachable, available, and functional. You also can choose to receive notifications when a resource becomes unavailable and choose to route internet traffic away from unhealthy resources.

## Hosted zone
A hosted zone is an Amazon Route 53 concept. A hosted zone is analogous to a traditional DNS zone file; it represents a collection of records that can be managed together, belonging to a single parent domain name. All resource record sets within a hosted zone must have the hosted zone's domain name as a suffix.