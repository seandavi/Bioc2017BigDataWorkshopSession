# Working with AWS Resources from R
Sean Davis  
7/26/2017  

# Introduction

To follow along with this little tutorial, open a *FREE* account with AWS:

https://aws.amazon.com/free/

If you have an Amazon account, you can use that username and password to login. If not, simply create a new account. After providing some some basic information and completing the account process, look for a search bar like this one.

![service-search](images/service-search.png)

Type “IAM” into the search bar and click on the first result, Amazon’s [Identity and Access Management](https://aws.amazon.com/iam/) system. On the left side of the IAM dashboard you should see a menu item called “Users”.

![iam-dashboard](images/iam-dashboard.png)

Click on “Users” and then you should see yet another dashboard which will list the users you create. If you’re new to AWS then you shouldn’t see any user names listed! Click on the blue “Add user” button to create a new user.

![add-user](images/add-user.png)

In the user name field just enter a name that you can easily remember, and make sure to check the check box to give this user programmatic access.

![add-jeff](images/add-jeff.png)

After you’ve checked the box, you can click Next: Permissions, then you should see the screen below.

![permissions](images/permissions.png)

On this page you first need to select how you’re going to assign AWS permissions to this user. Click *Attach existing policies directly* so that you can specify which services you’re going to use. In this case, we are going to be using AWS s3 and EC2. Starting with the s3 component, type s3 into the search box. 

![s3-access](images/s3-access.png)

Amazon S3 is just a big hard drive that you can access over the internet (like a programmable Dropbox). Check the box next to AmazonS3FullAccess so that you can create and destroy files in Amazon S3. Go through a similar process for the EC2 service. If you wanted to use a different web service, you could search for the name of that service here to provide even more access to this user. You can always return to the permissions page to add more services later if needed.

When you’re finished, click Next: Review. On the following screen click Create user in order to get your credentials. You should now see this screen.

![secret](images/secret.png)

In order to use AWS in R you need two strings. 

- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY

Click *show* under the secret access key heading in order to see the secret access key. Copy and paste each key into a file for safe keeping or download the `csv` file version. **Never share these keys since anyone with them can use your AWS account**. 

> **After you leave this page you will never be able to retrieve the secret access key, so make sure you’ve recorded it.** 

Don’t worry if you lose your secret access key though, you can just create a new user to get a new key. After you have both keys you can click the *Close* button. You’re now ready to start using AWS in R!

Note: **Never share your AWS credentials, and never include them in ANY code. Instead, set environment variables or use an AWS credential file.**

# Amazon s3

Now let’s test out our big cloud hard drive with the `aws.s3` package which you can install with `install.packages("aws.s3")`. To use the package you’ll need to set up three environmental variables like I have below by specifying the `access key ID`, and the `secret access key` which we just set up, plus the *AWS region* which specifies the Amazon server farm geographically closest to you (usually). You can get a list of all regions for all services [here](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region), and you click here to see which S3 regions are available.



```r
library(aws.s3)
```


```r
# Set up your keys and your region here.
# Note that you should not do this, but instead
# included this code in a .Rprofile file, a .Renviron 
# file, or an AWS config file (best option).
Sys.setenv("AWS_ACCESS_KEY_ID" = "EA6TDV7ASDE9TL2WI6RJ",
           "AWS_SECRET_ACCESS_KEY" = "OSnwITbMzcAwvHfYDEmk10khb3g82j04Wj8Va4AA",
           "AWS_DEFAULT_REGION" = "us-east-2")
```

Now, let's kick the tires a bit by creating a CSV file that we can store to the cloud.


```r
# mtcars is a built-in data set. Let's create a new CSV file that we can upload
# to AWS S3.
write.csv(mtcars, "mtcars.csv", row.names = FALSE)
```

Now let’s upload this file to S3. Amazon organizes S3 into *buckets*, which are like named cloud hard drives. You *must* use a name that nobody else has used before, so we’re going to introduce some randomness to come up with a unique bucket name. The put_bucket() function creates the bucket.



```r
# Coming up with unique bucket names is tricky, so we'll add a random string
# to the name of this bucket.
bucket_name <- paste(c("bioc2017-test-", sample(c(0:9, letters), 
                                            size = 10, replace = TRUE)), collapse = "")

# Now we can create the bucket.
put_bucket(bucket_name,region='us-east-2')
```

```
## [1] TRUE
```


```r
bucket_exists(bucket_name)
```

```
## [1] TRUE
## attr(,"x-amz-id-2")
## [1] "IHJIn6MExzQWZf/Fr9nywusNXvlFPoo8uNYisB9Wf0Ha2lU5ONldvoW6xOMEiYghmUEYmXy6aZQ="
## attr(,"x-amz-request-id")
## [1] "049D683B4FB76C87"
## attr(,"date")
## [1] "Wed, 26 Jul 2017 14:18:28 GMT"
## attr(,"x-amz-bucket-region")
## [1] "us-east-2"
## attr(,"content-type")
## [1] "application/xml"
## attr(,"transfer-encoding")
## [1] "chunked"
## attr(,"server")
## [1] "AmazonS3"
```

```r
bucket_list_df()
```

```
##                             Bucket             CreationDate
## 1                ExtremeComputeBio 2015-09-29T10:10:14.000Z
## 2  aws-logs-377200973048-us-east-1 2017-01-02T22:59:36.000Z
## 3  aws-logs-377200973048-us-west-2 2017-03-03T01:19:59.000Z
## 4         bioc2017-test-amh0jcnhvd 2017-07-26T12:21:07.000Z
## 5         bioc2017-test-fkn9b8mqxh 2017-07-26T14:18:25.000Z
## 6         bioc2017-test-gbtd3xb9qu 2017-07-26T12:31:31.000Z
## 7         bioc2017-test-h59vvn6h52 2017-07-26T14:08:47.000Z
## 8         bioc2017-test-hz27fet2b6 2017-07-26T12:26:01.000Z
## 9         bioc2017-test-r368y5kbph 2017-07-26T14:07:56.000Z
## 10        bioc2017-test-rv8woh29qx 2017-07-26T14:11:39.000Z
## 11        bioc2017-test-w6bc9xqvck 2017-07-26T12:22:39.000Z
## 12        bioc2017-test-wezlxjojt9 2017-07-26T14:18:07.000Z
## 13           giab.s3.amazonaws.com 2017-03-03T23:39:28.000Z
## 14       gov.cancer.ccr.publicdata 2017-06-23T12:55:57.000Z
## 15                            osf1 2017-05-25T22:40:18.000Z
## 16              sdavis.nci.nih.gov 2015-06-12T19:22:35.000Z
## 17             target-osteosarcoma 2017-05-31T18:50:46.000Z
## 18             teamcgc.nci.nih.gov 2016-02-02T16:53:35.000Z
```

Now we can put files in our bucket using put_object() by specifying which bucket we want to use:


```r
# Let's put our CSV file in the bucket.
put_object("mtcars.csv", bucket = bucket_name)
```

```
## [1] TRUE
```

We can then pull files out of the bucket given the name of the file with the save_object() function. Let’s make sure the file we pulled out of the bucket is the same as the file we put in!


```r
# We've put data in The Cloud! Now let's get it back on our computer:
save_object("mtcars.csv", bucket = bucket_name, file = "mtcars_s3.csv")
```

```
## [1] "mtcars_s3.csv"
```


```r
# Are the files the same?
mtcars_s3 <- read.csv("mtcars_s3.csv")
all.equal(mtcars, mtcars_s3)
```

```
## [1] "Attributes: < Component \"row.names\": Modes: character, numeric >"              
## [2] "Attributes: < Component \"row.names\": target is character, current is numeric >"
```

```r
# and delete the files
unlink('mtcars_s3.csv')
unlink('mtcars.csv')
```

Alternatively, we can read the object directly. The `get_object` function creates a `rawConnection` that we can read using many R input functions, such as `read.csv`.


```r
# Get the raw connection
rc = get_object('mtcars.csv',bucket_name)
# and convert to text.
m = read.csv(text=rawToChar(rc))
head(m)
```

```
##    mpg cyl disp  hp drat    wt  qsec vs am gear carb
## 1 21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
## 2 21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
## 3 22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
## 4 21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
## 5 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
## 6 18.1   6  225 105 2.76 3.460 20.22  1  0    3    1
```

We can do a one-step approach if we like using the `s3read_using` function:


```r
m1 = s3read_using(read.csv,object='mtcars.csv',bucket=bucket_name)
```

Looks like it works! Now let’s delete the hard drive we created since we won’t need to use it anymore.


```r
# We're finished with this bucket, so let's delete it.
delete_bucket(bucket_name)
```

```
## Client error: (409) Conflict
```

```
## [1] FALSE
## attr(,"x-amz-request-id")
## [1] "DBD941477CF81364"
## attr(,"x-amz-id-2")
## [1] "udSwRkjrGTt33kBnjl5q0q7RGiuOWa1A46TfgFdkMaUTdW19IEPgzhRhswBOrfb5QSeagNg+3yE="
## attr(,"content-type")
## [1] "application/xml"
## attr(,"transfer-encoding")
## [1] "chunked"
## attr(,"date")
## [1] "Wed, 26 Jul 2017 14:18:28 GMT"
## attr(,"server")
## [1] "AmazonS3"
```

*Always clean up after yourself during these experiments and you should not incur any charges.*

# EC2: Elastic Compute Cloud

Amazon Elastic Compute Cloud (Amazon EC2) is a service, accessible via a web-based API, that provides secure, resizable compute capacity in the cloud. It is designed to make web-scale cloud computing easier for developers and researchers. 

- *Scale*: Services like EC2 enable you to increase or decrease capacity at will, driven by computational demand. You can commission one, hundreds, or even thousands of server instances simultaneously. You can also use "Auto Scaling" to maintain availability of your EC2 fleet and automatically scale your applications up and down depending on needs and to maximize performance and minimize cost. 
- *Cost*: There are a [number of pricing options available](https://aws.amazon.com/ec2/pricing/spot-instances/), but for many bioinformatics workloads, using spot pricing may be acceptible and offers substantial discounts over "on demand" pricing. 
- *Security*: Cloud systems can be incredibly secure, but the security model used by all cloud providers is a "shared" security model. Like all of the big cloud providers, AWS supplies data center and network architecture designed to meet the requirements of the most security-sensitive organizations. However, the security of the systems that we set up, as users of the EC2 service, is up to us to maintain. 

AWS has a [Getting started with AWS ](https://aws.amazon.com/ec2/getting-started/) that you can use to create an instance using the web console, AWS SDK, or AWS command line tools.

## Creating and monitoring an AWS instance

This section details creating an AWS instance from within R. It assumes that you have your AWS key and secret available. We will be using the [preloaded AMI](http://bioconductor.org/help/bioconductor-cloud-ami/#preloaded_ami) from Bioconductor.

The package that will help us with the details is the `aws.ec2` package.


```r
install.packages('aws.ec2')
```


```r
library(aws.ec2)
```

We will need a "keypair" to allow SSH access later.


```r
keypair_name = 'bioc2017'
create_keypair('bioc2017',path = 'bioc2017.pem')
```

```
## keyName:         bioc2017 
## keyFingerprint:  e1:02:a5:2b:2b:2b:59:5d:90:39:b5:60:91:42:78:e0:ee:0e:2f:fe
```

Next, we need to create a network configuration that will allow us to interact with the rstudio server that will be running on the AMI. This is messy, but follow along.


```r
s <- describe_subnets()[[1]]
g <- create_sgroup("biocAMI_group", "new security group", vpc = s)
authorize_ingress(g, cidr = '0.0.0.0/0', port = 80, protocol = "tcp")
```

```
## [1] TRUE
```

```r
authorize_ingress(g, cidr = '0.0.0.0/0', port = 22, protocol = "tcp")
```

```
## [1] TRUE
```


```r
imageid = 'ami-a4697eb2'
# Launch the instance using appropriate settings
i <- run_instances(image = imageid, 
                   type = "t2.micro", # <- you might want to change this
                   subnet = s, 
                   sgroup = g)
```

Our AWS EC2 image is turning on. We can look into lots of details of the image, including getting the DNS name for connecting to the instance via RStudio server.


```r
# RStudio Server will be available at the "publicIp" address returned in `i`
str(describe_instances(i))
dnsName = describe_instances(i)[[1]]$instancesSet[[1]]$dnsName
```

After a few minutes, we should be able to browse to the url with username `ubuntu` and password `bioc`.


```r
browseURL(paste0('http://',dnsName))
```
And cleanup.





```r
# Stop and terminate the instances
stop_instances(i[[1]])
# terminating an instance will result in losing
# any and all data!
terminate_instances(i[[1]])
# and cleanup our security group.
Sys.sleep(120)
delete_sgroup(g)
delete_keypair(keypair_name)
```


# sessionInfo


```r
sessionInfo()
```

```
## R Under development (unstable) (2016-10-26 r71594)
## Platform: x86_64-apple-darwin13.4.0 (64-bit)
## Running under: macOS Sierra 10.12.4
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] aws.ec2_0.1.10 aws.s3_0.3.7  
## 
## loaded via a namespace (and not attached):
##  [1] Rcpp_0.12.11        digest_0.6.12       rprojroot_1.2      
##  [4] mime_0.5            aws.signature_0.3.5 R6_2.2.0           
##  [7] backports_1.0.5     magrittr_1.5        evaluate_0.10      
## [10] httr_1.2.1          stringi_1.1.5       curl_2.5           
## [13] xml2_1.1.1          rmarkdown_1.4       tools_3.4.0        
## [16] stringr_1.2.0       yaml_2.1.14         base64enc_0.1-3    
## [19] htmltools_0.3.5     knitr_1.16
```
