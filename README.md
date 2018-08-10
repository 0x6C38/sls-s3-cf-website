# Static Website on AWS with Serverless Framework
This repository contains infrastructure code for hosting a static website on AWS using Serverless Framework. The website is served using an S3 bucket and cached using cloudfront and it can serve millions of users, without a problem.

## Architecture
Each piece of infrastructure is declared in it's own file in the `/resources` directory. The `serverless.yml` file centralizes all resources, including:

- 2 S3 Buckets (1 for the static files, 1 for redirecting `.www`)
- 2 Cloudfront Distributions (1 for the root, 1 for redirecting `.www`)
- DNS A and AAAA records from the base domain and the `www` domain to cloudfront

## Requirements
- Have nodejs v8.10+ and npm installed
- Have serverless framework installed (`npm install -g serverless`)
- Have your aws credentials [setup](https://serverless.com/framework/docs/providers/aws/guide/credentials/)


## Set up
1. Clone this repository
2. Install the dependencies (`npm install`)
3. Register a domain name on AWS using Route53.
4. Issue a certificate for `example.com` and `www.example.com` using ACM,
5. Have the certificate verified by creating DNS records on Route53 using ACM.
6. Go into `serverless.yml` and change `nm` and `dn` to match your domain the `custom` section.
7. Change `certArn` to your certificate's ARN which can be found on the ACM console.
8. Change `dhzId` to the Hosted Zone ID which Route53 created for you when you registered the domain.
9. Deploy the infrastructure using `serverless deploy --verbose` and wait until it's finished which should be aprox. 20 min because of cloudfront.

### Using a different Registrar
In case you prefer to use a registrar that is not amazon for registering your domain name, steps 4 to 9 still apply but you need to create the hosted zone for your domain manually, which you can do by adding this to the resources on DNS-config.yml:

```
```

You also need to...

## Usage
Once all of the infrastructure has been provisioned, any static files you upload to the main website bucket will be served on your domain using https.


## Clean up
First go to S3 empty both the redirect and website buckets. Then you can use `serverless remove` to clean up all resources provisioned by this project as stated in the serverless framework [documentation](https://serverless.com/framework/docs/providers/aws/guide/quick-start/#cleanup). Alternatively you can delete the stack on cloudformation. Finally, don't forget to delete the domain on Route53.