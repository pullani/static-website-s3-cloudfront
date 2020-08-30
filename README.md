
<br>
<h3 align="center">A Static Website Using AWS S3, CloudFront and Godaddy DNS</h3>

  

---

  

<p align="center"> Serve a static website from AWS S3 bucket using CloudFront and Godaddy DNS

<br>

</p>

  

## 📝 Table of Contents

- [Getting Started](#getting_started)

- [Setting up AWS S3 Bucket](#aws_s3)

- [AWS Certificate Manager (ACM)](#aws_acm)

- [CloudFront Integration](#cloudfront)

  
## Getting Started <a name = "getting_started"></a>

We will use a Free Tier AWS account and a domain purchased on Godaddy. Assuming www.mydomain.com as our domain name.

## Setting up AWS S3 Bucket <a name = "aws_s3"></a>

Go to [AWS S3 Management Console]([https://s3.console.aws.amazon.com/s3/home](https://s3.console.aws.amazon.com/s3/home)) and change the AWS region to the one closest to you. 

Steps involved in configuring S3:

- Create an S3 bucket
- Configure Public access settings.
- Add Bucket policy
- Configure bucket property for static web hosting.
- Get public end point to the bucket.

#### Creating S3 bucket:
Create an S3 bucket under the name of your domain. 
Eg. www.mydomain.com

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/create_bucket.jpg?raw=true" alt="content">

#### Set Bucket public access permissions:
Set the configuration as following:

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/public_access.jpg?raw=true" alt="content">

#### Add bucket policy:
Use the following policy by renaming domain name.
```

{
    "Version": "2008-10-17",
    "Id": "PolicyForPublicWebsiteContent",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::www.mydomain.com/*"
        }
    ]
}

```

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/policy.jpg?raw=true" alt="content">

#### Select bucket property to host static webpage:
In 'Properties' tab, choose 'Static Website Hosting'. 
<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/index01.jpg?raw=true" alt="content">

Give your web page's index html file name in the index field and error html page in the corresponding one if you have one for the later. 

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/index02.jpg?raw=true" alt="content">


#### Upload the static files to bucket
Ensure index html is in S3 root directory. Otherwise it is not possible to configure S3 to serve webpage without specifiying its location in the bucket tree. 

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/files.jpg?raw=true" alt="content">

Now the website should be accessible via the index html:

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/object_url.jpg?raw=true" alt="content">

#### Public endpoint URL for index html:
The S3 public endpoint to your static website is region specific.
Refer to [https://docs.aws.amazon.com/general/latest/gr/s3.html](https://docs.aws.amazon.com/general/latest/gr/s3.html) and get your region alias, obtain your bucket endpoint.
So the endpoint will look like:
```
http://www.mydomain.com.s3-website.your_region.amazonaws.com
```
```
Now, your 'region specific' bucket root endpoint will directly serve the index html without user specifying it in the url.  
i.e., 
http://www.mydomain.com.s3-website.region.amazonaws.com
will directly serve you the index html.

For the above to work ensure the index html file is in the bucket root directory.
```
<b>Now check the above endpoint is serving your website as expected on a browser.</b> (Connection will not be secured as the URL is http.)

Note: 
A region unspecific endpoint like
http://www.mydomain.com.s3.amazonaws.com/
will not serve you the index html unless you specify it in the url like
http://www.mydomain.com.s3.amazonaws.com/index.html



## AWS Certificate Manager (ACM) <a name = "aws_acm"></a>
In the next section we will configure CloudFront with our domain to let users access website directly under our domain name. To proceed, we need SSL certificates for our custom domain. 

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/acm_home.jpg?raw=true" alt="content">

[AWS certificate manager](https://console.aws.amazon.com/acm/home) is a free SSL certificate manager from AWS. Certificates are limited to use within AWS services.

```
Important:
For using custom domain names to redirect requests to CloudFront, we need an ACM certificate in the region specifically "US West (N. Virginia)" 
```
#### Obtaining certificates:
We will get a <b>wildcard</b> certificate for our domain. This will help us to use the same certificates across multiple subdomains later.

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/acm_cert.jpg?raw=true" alt="content">

Steps:
- Change the region to 'US West (N. Virginia)' from your ACM home page.
- Click on 'Request Certificate' and provide <b>*.mydomain.com</b> in the domain field:
< Image here>
- On the next page select Email verification if you have an email account under your domain name, or otherwise use DNS verification.
- DNS verification for Godaddy account:

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/acm_dns_validation.jpg?raw=true" alt="content">

Once the DNS validation method is selected console will
ask us to add the CNAME to DNS entry:
```
Name field will look like:
_c98d6d2cae8dd5688z0u890zd.www.mydomain.com.
```
Copy the <b>_c98d6d2cae8dd5688z0u890zd</b> part alone and paste it in the "Host" field of the Godaddy CNAME entry.
```
Value field will look like:
_749063zab7efcbab23.zdxcnfdgtt.acm-validations.aws.
```
Copy the value field as such and paste it in the "Points to" field of the Godaddy CNAME entry.

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/wild_card_cname.jpg?raw=true" alt="content">

The certificates should be issued immediately once the CNAME record is added.

## CloudFront Integration <a name = "cloudfront"></a>

Procedure is to create a 'CloudFront distribution' with the 'Origin domain' as our previously created S3 bucket containing our static files. We will use our region specific bucket endpoint here. We will also provide our domain name and ACM certificates in the distribution settings.



#### Subdomain to redirect traffic to CloudFront 
```
Important:
Godaddy 'A' name record of root domain '@' cannot be pointed to a URL. If this is ultimately needed for you, consider transferring your domain to a service like Amazon Route53. Route53 allows root domain to be redirected to a URL. 
```
So, for our purpose we will use a subdomain in a CNAME record pointing to our CloudFront URL.

For example we will use, cdn.mydomain.com

#### Creating a CloudFront distribution


Steps:

- Go to [CloudFront Console](https://console.aws.amazon.com/cloudfront/home)  (CloudFront does not require any region selection.)

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/cloud_front_home.jpg?raw=true" alt="content">

- Click on 'Create Distribution' and select content delivery method as web.

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/cloud_front_static.jpg?raw=true" alt="content">

- Paste your 'Region specific S3 endpoint' in the 'Origin Domain Name' field. Avoid clicking the auto suggestion to click on S3 bucket.
>Leave the 'Origin path' unfilled. Previoulsy we ensured our index.html to be in the bucket root directory inorder to serve the webpage from S3 without specifying index html file in the URL.
- Set 'Viewer Protocol Policy' to 'Redirect HTTP to HTTPS'

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/cdn_source.jpg?raw=true" alt="content">

- Under 'Distribution Settings' fill our subdomain name 'cdn.mydomian.com'
- Choose 'Custom SSL Certificate' and select the willdcard ACM certificate we previoulsy ceated from the dropdown.

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/cdn_cert.jpg?raw=true" alt="content">

- Click on Create Distribution

It will take a couple of minutes for the Distribution to deploy.
<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/cdn_success.jpg?raw=true" alt="content">
Once the distribution is deployed, we can get our CloudFront URL from the 'General' settings of distribution.
It will look like:
```
d2c8z7f9rkd1td.cloudfront.net
```
<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/cdn_url.jpg?raw=true" alt="content">

#### Final step:  Adding Subdomain CNAME on Godaddy:
Add a CNAME with 'Host' as our subdomain 'cdn' and pointing to our CloudFront URL d2c8z7f9rkd1td.cloudfront.net.

<img src="https://github.com/pullani/static-website-s3-cloudfront/blob/master/content/cdn_cname_godaddy.jpg?raw=true" alt="content">

```
Now check cdn.mydomain.com on a web browser. It should load the website on a snap from the CloudFront!
```
