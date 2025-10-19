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

3] Region: ```ap-south-1``` (or your nearest region)

4] Block all **public** access 


### Step 2: Create the Lambda Function

1. Go to AWS Lambda → Create Function
2. Choose:
   * Name: ```imageResizerFunction```
   * Runtime: Node.js 18.x
   * Permissions → Create a new role with basic Lambda permissions
   * Click Create Function

### Step 3: Add Required Permissions

**1. Your Lambda needs to:**

* Read from ```source-bucket```
* Write to ```resized-bucket```
  
**2. Add Permissions:**

* Go to IAM → Roles → your Lambda role → Add Permissions → Attach Policies

**3. Attach:**

* AmazonS3FullAccess
* CloudWatchLogsFullAccess

### Step 4: Add S3 Event Trigger

1. In Lambda → “Configuration” tab → Triggers → Add Trigger
2. Select:
      * Source: S3
      * Bucket: **source-bucket-image-resizer**
      * Event type: PUT (when a new object is created)
  
3. Click Add

### Step 5: Add Lambda Code

Replace the default code with this:

* **index.mjs**

```import AWS from 'aws-sdk';
import sharp from 'sharp';

const s3 = new AWS.S3();

export const handler = async (event) => {
  try {
    // Get S3 bucket and file details
    const bucket = event.Records[0].s3.bucket.name;
    const key = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, ' '));

    console.log(`Processing image: ${bucket}/${key}`);

    // Get the image from S3
    const originalImage = await s3.getObject({ Bucket: bucket, Key: key }).promise();

    // Resize the image using Sharp
    const resizedImage = await sharp(originalImage.Body)
      .resize(300, 300) // width, height
      .toBuffer();

    // Upload to destination bucket
    const destinationBucket = 'resized-bucket-image-resizer';
    const destinationKey = `resized-${key}`;

    await s3.putObject({
      Bucket: destinationBucket,
      Key: destinationKey,
      Body: resizedImage,
      ContentType: 'image/jpeg',
    }).promise();

    console.log(`✅ Resized image uploaded: ${destinationBucket}/${destinationKey}`);

    return {
      statusCode: 200,
      body: 'Image resized successfully!',
    };
  } catch (err) {
    console.error(err);
    throw new Error('Image resizing failed.');
  }
};
```

### Step 6: Package Dependencies (Sharp)
**Because Lambda runs in a Linux environment, we need to package sharp correctly**

### On your local terminal :
```mkdir aws-serverless-image-resizer
cd aws-serverless-image-resizer
npm init -y
npm install sharp aws-sdk
```

### Then ZIP all the contents

```zip -r function.zip .```


* Now upload ```function.zip``` in Lambda Console:

1. Go to Lambda → Code → Upload from → .zip file

### Step 7: Test the Project

* Go to S3 → Upload an image in ```source-bucket-image-resizer```
* Wait a few seconds
* Check **resized-bucket-image-resizer** → you’ll see a new file **resized-<filename>.jpg**

**If successful, CloudWatch Logs will show Image resized successfully!**




