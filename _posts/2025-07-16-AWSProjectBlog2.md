# AWS Projects: File Streaming with AWS Media Services
### Processing Videos for Web Streaming
### Timothy Coleman
### 08/09/25
Level: 100  
Cost: < $1.00 for 2 minutes of content


Hello, my name is Timothy Coleman and I'm a UCLA undergrad working towards becoming a Solutions Architect. This is first of a series of cloud projects that I will document on GitHub. In making these posts, I aim to build cloud-engineering and architectural skills, improve my understanding of cloud principles and GitHub, and demonstrate commitment, learning, and growth.


The goal of this project is to processing file-based media into videos for web streaming

The following services / resources are utilized:
- Amazon S3
- Amazon CloudFront
- AWS Elemental MediaConvert
-----
## Storage
My goal here is to create and configure a system of storage

First, 2 S3 buckets are created--one for source assets and another for outputs from MediaConvert jobs. Public read access is only allowed through Amazon CloudFront, a content delivery network (CDN). A Cross Origin Resource Sharing (CORS) policy bucket is applied to the outputs bucket.

CDN Benefits:
1. Improved performance to end users
2. Reduces load on backend origin infrastructure by caching content closer to the edge

### Steps
1. Create a Source Bucket
   - Choose a globally unique name
   - Select the region US-EAST-1
   - Keep all other configurations on their defaults and create the bucket
2. Create a Destination Bucket
   - Follow all the instructions in step 1
   - Enable bucket versioning then create the bucket
3. Add the following CORS configuration to the destination bucket:
   
         [
          {
              "AllowedHeaders": [
                  "*"
              ],
              "AllowedMethods": [
                  "GET"
              ],
              "AllowedOrigins": [
                  "*"
            ],
              "ExposeHeaders": [],
              "MaxAgeSeconds": 3000
          }
         ]
5. Create an IAM Role
   - Select the AWS Service role type
   - Select the MediaConvert use case
   - Name the role "vod-MediaConvertRole", then finish creating

### Takeaways
* CORS may enable client web apps that are loaded in one domain to interact with resources in another domain
-----
## CloudFront
My goal here is to create and configure a content delivery network

### Steps
1. Select Create Distribution
2. Choose a unique name
3. Choose the Amazon S3 origin type
4. Choose the destination bucket as the S3 origin
5. Enable private S3 bucket access to CloudFront
6. Customize cache settings
   - Select HTTPS only for Viewer Protocol Policy
   - Select Simple CORS for Response Header Policy
7. Enable Security Protections and Monitor Mode
8. Create the Distrubtion
9. Edit the price class setting to "Use only North America and Europe"
-----
## Transcoding

My goal here is to create a MediaConvert job that convert video inputs into an Apple HLS input group

Apple HLS: an adaptive bitrate stack that encodes the source into mutliple files at different bitrates and divides the content into small chunks, usually around 5 - 10 seconds

### Steps
1. Create a MediaConvert job
   - Input the following url into Input 1  
     https://static.us-east-1.prod.workshops.aws/public/09743ab7-164b-4f79-a072-45f200ee05e0/assets/VANLIFE.m2ts
      - This URL contains the video content
   - Add an Apple HLS output group
   - Create a unique custom group name
   - Add the destination bucket + the following text "/assets/VANLIFE/HLS/"
   - Scroll down and add two more outputs (not output groups)
   - Set the configurations for Output 1
      * Name: _360
      * Segment: \$dt\\$ , this appends datetime to each media segment for identification and the prevention of accidental overrides
      * Resolution (w x h): 640 and 360
      * Rate Control Mode: QVBR
      * Quality Tunning Level: Single Pass HQ
      * Max bits/s: 1000000
   - Set the configurations for Output 1
      * Name: _540
      * Segment: \$dt\$
      * Resolution (w x h): 960 and 540
      * Rate Control Mode: QVBR
      * Quality Tunning Level: Single Pass HQ
      * Max bits/s: 2000000
   - Set the configurations for Output 1
      * Name: _720
      * Segment: \$dt\$ , this appends datetime to each media segment for identification and the prevention of accidental overrides
      * Resolution (w x h): 1280 and 720
      * Rate Control Mode: QVBR
      * Quality Tunning Level: Single Pass HQ
      * Max bits/s: 3000000
   - Under the AWS Intergration tab, select the recently created MediaConvert role
   - Select a 10 second status update interval
   - Finish creating the job
  1. Create Job Template
   - Under Job Details, select Export JSON
   - Import this file into a job template
   - Create a name for the job template
   - After creating the template, export its JSON file

-----
## Playback

I ran into a problem accessing the video through the browser; the page read Access Denied. I jumped back to the MediaConvert configuration and realized that I forgot to add the "/assets/VANLIFE/HLS/" to the back of the destination when creating the job. After correcting my error, I edited the instructions to reflect the proper setup. Still, after correcting this mistake, I was denied access to the content through the browser. So now I'm thinking "Okay, Access Denied? That sounds like an S3 or CloudFront issue". When I'm checking to verfiy the CORS policy has been enabled, I notice that the destination bucket blocks all public access. Once I allow unconditional public access, I'm able to view the video in the browser. Unconditional public access isn't ideal so, I check for ways to mimimize access.

I removed all public access once again, but I'm still able to access the page now. I thought that this could've been a cached video and not coming directly from the bucket, but when I disable caching in CloudFront I'm still able to access the video. The most likely cause of my problem was a typo in pasting the URL. One extra period or dash at the end produces the same error message I mentioned before.

### Takeaways
* Make sure that URLS are exact to the character
* Browser accessibility issues may stem from S3 configuration
-----
## Automation

### Steps
1. Create an IAM Role
  - Create a role for Lambda
  - Choose the role "AWSLambdaBasicExecutionRole"
  - Enter the name "VODLambdaRole"
  - Finish creating the role then add the following inline policy:
    
          {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Action": [
                      "logs:CreateLogGroup",
                      "logs:CreateLogStream",
                      "logs:PutLogEvents"
                  ],
                  "Resource": "*",
                  "Effect": "Allow",
                  "Sid": "Logging"
              },
              {
                  "Action": [
                      "iam:PassRole"
                  ],
                  "Resource": [
                      "<ARN for VODMediaConvertRole>"
                  ],
                  "Effect": "Allow",
                  "Sid": "PassRole"
              },
              {
                  "Action": [
                      "mediaconvert:*"
                  ],
                  "Resource": [
                      "*"
                  ],
                  "Effect": "Allow",
                  "Sid": "MediaConvertService"
              }
          ]
         }
   - Under resource, add the ARN for the VODMediaConvertRole created earlier
   - Enter VODLambdaPolicy as the policy name
   - Create policy
2. Create a Lambda function
   - Start a new function from scratch and name it VodLambdaConvert
   - Run it on the lastest version of python
   - Choose VODLambdaRole as the execution role and finish creating
   - Configure a new trigger
     * Trigger type: S3
     * Bucket: source bucket
     * Event: All object create events
     * Acknoledge the Recursive Invocation then finish creating the trigger
   - Configure environment variables as key-value pairs
     * DestinationBucket: name of media bucket
     * MediaConvertRole: name of media convert role
   - Under general configuration, set the timeout to 2 minutes
   - Copy the following code and paste it into the lambda_function.py file:

         import logging
         import subprocess
         import xml.etree.ElementTree as ET
         import boto3
         import os 
         import json
         import uuid
         
         S3 = boto3.client("s3")
         LOGGER = logging.getLogger('boto3')
         LOGGER.setLevel(logging.INFO)
         REGION = os.environ['AWS_DEFAULT_REGION']
         
         # Get the account-specific mediaconvert endpoint for this region and chache them
         if 'MEDIACONVERT_ENDPOINT' in os.environ:
             ENDPOINTS = os.environ["MEDIACONVERT_ENDPOINT"]
         else:
             try:
                 MC = boto3.client('mediaconvert', region_name=REGION)
                 response = MC.describe_endpoints()
             except Exception as e:
                 print("Exception:\n", e)
                 raise 
             else:
                 ENDPOINTS = response["Endpoints"][0]["Url"]
                 # Cache the mediaconvert endpoint in order to avoid getting throttled on
                 # the DescribeEndpoints API.
                 os.environ["MEDIACONVERT_ENDPOINT"] = ENDPOINTS
         
         # Add the account-specific endpoint to the client session 
         MEDIACONVERT = boto3.client('mediaconvert', region_name=REGION, endpoint_url=ENDPOINTS)
         
         destinationS3 = 's3://' + os.environ['DestinationBucket']
         mediaConvertRole = os.environ['MediaConvertRole']
         
         def lambda_handler(event, context):
             statusCode = 200
             body = {}
             
             try:
                 #Grab Event Info + Enviornment Variables
                 assetID = str(uuid.uuid4())
                 jobMetadata = {'assetID': assetID}
                 sourceS3Bucket = event['Records'][0]['s3']['bucket']['name']
                 sourceS3Key = event['Records'][0]['s3']['object']['key']
                 sourceS3 = 's3://'+ sourceS3Bucket + '/' + sourceS3Key
                 sourceS3Basename = os.path.splitext(os.path.basename(sourceS3))[0]
                 
                 # Loop through records provided by S3 Event trigger
                 for s3_record in event['Records']:
                     LOGGER.info("Working on new s3_record...")
                     
                     # Extract the Key and Bucket names for the asset uploaded to S3
                     key = s3_record['s3']['object']['key']
                     bucket = s3_record['s3']['bucket']['name']
                     LOGGER.info("Bucket: {} \t Key: {}".format(bucket, key))
                     
                     # Load Job Settings Template
                     LOGGER.info("Loading Job Settings...")
                     with open('job_template.json') as json_data:
                         jobTemplate = json.load(json_data)
                         LOGGER.info("Input JobSettings:")
                         jobSettings=jobTemplate['Settings']
                         jobSettings['Inputs'][0]['FileInput'] = sourceS3
                        
                     basekey = 'assets/' 
                     S3KeyHLS = basekey + assetID +'/HLS/'+ sourceS3Basename
             
                     LOGGER.info("Creating MediaConvert Job...") 
                     
                     #update job.json
                     update_job_settings(jobSettings,assetID,destinationS3,S3KeyHLS)
                     MEDIACONVERT.create_job(Role=mediaConvertRole, UserMetadata=jobMetadata, Settings=jobSettings, StatusUpdateInterval="SECONDS_10")
         
             except Exception as e:
                 print ('Exception: %s' % e)
                 statusCode = 500
                 raise
         
             finally:
                 return {
                     'statusCode': statusCode,
                     'body': json.dumps(body),
                     'headers': {'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*'}
                 
             }    
         def update_job_settings(jobsettings,assetID,destinationBucket,S3HLS):
             """
             Update MediaConvert Job Settings
             
             :param jobsettings:         Loaded JobSettings JSON template
             :param assetID:             Tagged Metadata ID
             :param destinationBucket:   Bucket for Transcoded Media to reside
             :param S3HLS                S3 path for HLS transcode files
             :return:
             """
             LOGGER.info("Updating Job Settings...")
             jobsettings['OutputGroups'][0]['OutputGroupSettings']['HlsGroupSettings']['Destination'] \
             = destinationBucket + '/' + S3HLS
             
             LOGGER.info("Updated Job Settings...")

## Takeaways
* If a role is not create for a particular service, then the service will not trust it, and not allow the role to be used
