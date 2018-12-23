# Static Website on AWS with Serverless Framework
This repository contains infrastructure code for hosting a static website on AWS using Serverless Framework. The website is served using an S3 bucket and cached using cloudfront. This approach doesn't require any servers and can serve millions of users without a problem.

## Architecture
Each piece of infrastructure is declared in a separate file inside of the `/resources` directory. The `serverless.yml` file centralizes all resources, including:

- 2 S3 Buckets (1 for the static files, 1 for redirecting `.www`)
- 2 Cloudfront Distributions (1 for the root domain, 1 for redirecting `.www`)
- DNS A and AAAA records from the root domain and the `www` domain to cloudfront

Unfortunately there is no way to provision an ssl certificate and autoverify it using cloudformation so a few things need to be configured manually as explained in the setup section.

## Requirements
- Have nodejs v8.10+ and npm installed
- Have serverless framework installed (`npm install -g serverless`)
- Have your aws credentials [setup](https://serverless.com/framework/docs/providers/aws/guide/credentials/)


## Set up
1. Clone this repository
2. Install the dependencies (`npm install`)
3. Register a domain name on AWS using Route53.
4. Issue a certificate for `example.com` and `www.example.com` using ACM.
5. Have the certificate verified by creating DNS records on Route53 using ACM.
6. Go into `serverless.yml` and change `nm` and `dn` to match your domain in the `custom` variables section.
7. Change `certArn` to your certificate's ARN which can be found in the ACM console.
8. Change `dhzId` to the Hosted Zone ID which Route53 created for you when you registered the domain.
9. Deploy the infrastructure using `serverless deploy --verbose` and wait until it's finished (which should take aprox. 20 min because of cloudfront).

### Using a different Registrar
In case you prefer to use a registrar that isn't amazon, steps 4 to 9 still apply but you need to create a hosted zone for your domain manually. You can do that by adding the following to the resources section in `DNS-config.yml`:

```
Resources:
...
  WsHostedZone:
    Type: AWS::Route53::HostedZone
    Properties:
      Name: ${self:custom.dn}
```
Now, instead of using the `dhzId` variable in step 8, refer to the hosted zone you created manually:

```
Resources:     
   DNSRecords:
    Type: AWS::Route53::RecordSetGroup
    Properties:
      HostedZoneId:
        Ref: WsHostedZone
      RecordSets:
...
```

Declaring the hosted zone will also create `NS` and `SOA` records in Route53. You must use the nameserver addresses in the `NS` record as the nameservers for your domain. This can be done through your registrar's website but the exact procedure varies from registrar to registrar.

## Usage
Once all of the infrastructure has been provisioned, any static files you upload to the main website bucket will be served on your domain using https. As a convenience, I've added a `ReactUploadCommand` and a `VanillaUploadCommand` to the stack output, which should be printed in your terminal when you deploy the stack using the `--verbose` flag. These commands will upload the files inside of the `/website` directory to S3. Additionally, Cloudfront's cache will be invalidated so that your users are served the latest version of the site.

The commands should look as follows:

```
VanillaUploadCommand: aws s3 sync website/ s3://example-bucket-website --delete && aws cloudfront create-invalidation --distribution-id ZZZZZZZZZZZZ --paths '/*' && aws cloudfront create-invalidation --distribution-id XXXXXXXXXXXX --paths '/*'
```

```
ReactUploadCommand: (cd ./website && npm run build) && aws s3 sync website/build/ s3://example-bucket-website --delete && aws cloudfront create-invalidation --distribution-id ZZZZZZZZZZZZ --paths '/*' && aws cloudfront create-invalidation --distribution-id XXXXXXXXXXXX --paths '/*'
```

`ZZZZZZZZZZZZ` and `XXXXXXXXXXXX` are the corresponding cloudfront distribution ids.


## Clean up
First go to S3 empty both the redirect and website buckets. Second, delete the dns record created by Route53 to verify the domain ownership. Then you can use `serverless remove` to clean up all resources provisioned by this project as stated in the serverless framework [documentation](https://serverless.com/framework/docs/providers/aws/guide/quick-start/#cleanup). Alternatively you can delete the stack on cloudformation and then delete the deployment bucket for the project. Finally, don't forget to delete the domain itself on Route53.
