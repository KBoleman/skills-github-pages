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
4. Create an IAM Role
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
      * Segment: $dt$ , this appends datetime to each media segment for identification and the prevention of accidental overrides
      * Resolution (w x h): 640 and 360
      * Rate Control Mode: QVBR
      * Quality Tunning Level: Single Pass HQ
      * Max bits/s: 1000000
   - Set the configurations for Output 1
      * Name: _540
      * Segment: $dt$
      * Resolution (w x h): 960 and 540
      * Rate Control Mode: QVBR
      * Quality Tunning Level: Single Pass HQ
      * Max bits/s: 2000000
   - Set the configurations for Output 1
      * Name: _720
      * Segment: $dt$ , this appends datetime to each media segment for identification and the prevention of accidental overrides
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
