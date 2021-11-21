# Mkdocs AWS Static S3 Website
This CloudFormation script makes it easy to deploy [Mkdocs](https://www.mkdocs.org/) to a Static Website on AWS S3.

The S3 bucket is not public since it uses CloudFront and Origin Access Identities to front
traffic that is connecting to the website.

### Dependencies
- Mkdocs Installed
- AWS CLI Configured
- ACM Certificate
- DNS Name

## Input Parameters

| Name | Description  | Required  | Default Value  |
| :---:   | :-: | :-: | :-: |
| BucketName | The name of the S3 bucket to house your static content. | True | N/A |
| CertificateARN | The ACM certificate ARN for your website. | True | N/A |
| ErrorPage | The Error page for your website. | True | error.html |
| IndexPage | The index page of your website. | True | index.html |
| FunctionName | The name of the lambda edge function | True | N/A |

## Components

### S3 Bucket

The S3 bucket is where the static content is placed. For Mkdocs this would be the location your `site`
directory is added after running `mkdocs build`.

### CloudFront Distribution
The CloudFront distribution is what is publicly exposed for users to connect to. When creating the
CloudFront distribution an Origin Access Identity is created to allow communication from
CloudFront to the private S3 bucket.

### Lambda Edge Function
I used [this document](https://aws.amazon.com/blogs/compute/implementing-default-directory-indexes-in-amazon-s3-backed-amazon-cloudfront-origins-using-lambdaedge/) as a reference document when creating the Lambda function in CloudFormation.

The Lambda Edge Function reads requests to the static website that are requesting `/` for sub-directories.
Once identified it rewrites the request to `/index.html` for the appropriate sub-directory.

This is needed when using Mkdocs on static websites since sub URIs use index.html and by default in an S3 static website
if you access a subdirectory other than the root one `/`, your `index.html` will not resolve.

## Working Example

### Deploying The CloudFormation

Navigate to the AWS Console -> CloudFormation -> Create Stack with New Resources.

Select "Upload template file" and choose the file `mkdocs-static-website.yaml`.

Next enter a stack name and fill in the required parameters, then create the stack.

Once completed you may upload your static Mkdocs content or follow-along to create a sample.



### Mkdocs

You can use [Mkdocs](https://www.mkdocs.org/) to quickly write documentation and serve it on static websites. For details on how to use Mkdocs check out their website.

```bash
$ mkdocs new mywebsite
INFO     -  Creating project directory: mywebsite
INFO     -  Writing config file: mywebsite/mkdocs.yml
INFO     -  Writing initial docs: mywebsite/docs/index.md
```

Once created modify the templates until your desired content exists and then build.
```bash
$ cd mywebsite/
$ ls
docs  mkdocs.yml
$ mkdocs build
INFO     -  Cleaning site directory
INFO     -  Building documentation to directory: /mywebsite/site
INFO     -  Documentation built in 0.06 seconds
```

Upload the content to the S3 bucket.
```bash
$ aws s3 cp site/ s3://spencer-static-website-test --recursive
```

You can now access the website directly from the CloudFront Distribution URL.

### Mapping DNS with Route 53

If you are not using a subdomain and would like to map the static s3 website to the base URL
of your website address you need to create an A record in Route53.

Navigate to your hosted zone and create a new record with a record type "A" and route traffic to "Alias to CloudFront distribution". From there select your CloudFront Distribution and click save. 
