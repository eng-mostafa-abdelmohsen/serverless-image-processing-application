# serverless-image-processing-application
Create a serverless image processing application where users upload images to an S3 bucket, triggering an AWS Lambda function that processes and resizes the images before storing them in another S3 bucket.
# Serverless Image Processing on AWS

## 📌 Overview

This project demonstrates a **serverless image processing pipeline**
using AWS services.\
Users upload an image to an **Amazon S3 bucket**, which triggers a
**Lambda function** that resizes the image and stores the processed
result in another **S3 bucket**.

## 🏗️ Architecture

![Architecture Diagram](manara-serveless-project.JPG)

### Components

1.  **Amazon S3 (upload-image-20)**
    -   Acts as the input bucket.\
    -   Users upload raw/original images here.
2.  **AWS Lambda (image-processing-lambda)**
    -   Triggered automatically when a new object is uploaded to the
        input bucket.\
    -   Processes (e.g., resizes) the image using the AWS SDK and an
        image processing library like
        [Sharp](https://github.com/lovell/sharp) or Pillow.\
    -   Writes the processed image to the output bucket.
3.  **Amazon S3 (processed-image-20)**
    -   Acts as the output bucket.\
    -   Stores resized/processed images.

## 🔄 Flow

1.  User uploads an image → **upload-image-20** (S3 bucket).\
2.  S3 event notification triggers → **image-processing-lambda**.\
3.  Lambda resizes the image.\
4.  Resized image is saved in → **processed-image-20** (S3 bucket).

## ⚙️ Setup Instructions

### 1. Create S3 Buckets

-   Create two S3 buckets:
    -   `upload-image-20`
    -   `processed-image-20`

### 2. Create IAM Role for Lambda

-   Allow permissions for:
    -   `s3:GetObject` from `upload-image-20`
    -   `s3:PutObject` into `processed-image-20`
    -   CloudWatch Logs (for monitoring)

### 3. Create Lambda Function

-   Name: `image-processing-lambda`
-   Runtime: Node.js (with **Sharp**) or Python (with **Pillow**)
-   Attach IAM role created above.
-   Configure S3 trigger:
    -   Event type: `PUT`
    -   Bucket: `upload-image-20`

### 4. Upload Lambda Code

Example (Node.js with Sharp):

``` javascript
const AWS = require("aws-sdk");
const S3 = new AWS.S3();
const Sharp = require("sharp");

exports.handler = async (event) => {
  const bucket = event.Records[0].s3.bucket.name;
  const key = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, " "));

  try {
    const image = await S3.getObject({ Bucket: bucket, Key: key }).promise();

    const resizedImage = await Sharp(image.Body).resize(300, 300).toBuffer();

    await S3.putObject({
      Bucket: "processed-image-20",
      Key: key,
      Body: resizedImage,
      ContentType: "image/jpeg"
    }).promise();

    return { status: "Image resized successfully" };
  } catch (err) {
    console.error(err);
    throw new Error("Image processing failed");
  }
};
```

### 5. Test the Workflow

-   Upload an image (`.jpg`, `.png`, etc.) to `upload-image-20`.
-   Lambda is triggered automatically.
-   Check the `processed-image-20` bucket for the resized version.

## 📊 Monitoring

-   Use **CloudWatch Logs** to debug Lambda executions.
-   Configure **CloudWatch Metrics/Alarms** for error monitoring.

## 🚀 Future Enhancements

-   Add API Gateway for HTTP image uploads.
-   Store metadata in DynamoDB (filename, size, timestamp).
-   Support multiple image transformations (watermark, grayscale, etc.).
-   Use AWS Step Functions for complex workflows.
