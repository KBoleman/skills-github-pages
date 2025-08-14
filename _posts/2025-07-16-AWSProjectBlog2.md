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
- Amazon MediaConvert
-----
## Storage

First, 2 S3 buckets are created--one for source assets and another for outputs from MediaConvert jobs. Public read access is only allowed through Amazon CloudFront, a content delivery network (CDN). A Cross Origin Resource Sharing (CORS) policy bucket is applied to the outputs bucket.

CDN Benefits:
1. Improved performance to end users
2. Reduces load on backend origin infrastructure by caching content closer to the edge

### Steps
1. Create a Storage Bucket
   - 



