# aws-serverless-image-resizer

**When you upload an image to an S3 bucket (say, source-bucket), a Lambda function automatically resizes it and saves the resized version in another bucket (say, destination-bucket)**

## Architecture

```
[Upload Image]
      ↓
[S3 Source Bucket]
      ↓ (Event Trigger)
[AWS Lambda]
      ↓ (Resized Image)
[S3 Destination Bucket]
```

### Step 1: Create S3 bucket
* **We’ll use two buckets:**

1] ```source-bucket-image-resizer``` -> Upload original images here

2] ```resized-bucket-image-resizer``` -> Lambda saves resized images here

