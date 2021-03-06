# An AWS Lambda Function to Copy S3 Objects

We use this AWS Lambda function to copy objects from the FTF production Active Storage S3 bucket to the FTF review app Active Storage bucket as each new object is added to the production bucket.

### IAM Policy and Role

There is an IAM policy named S3Copy in the FTF AWS console with the following settings:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketTagging",
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::ftf-active-storage-prod/"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketTagging",
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::ftf-active-storage-review-apps/"
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:*"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

There is also a IAM role named S3Copy with the above S3Copy policy assigned to it. That IAM role and policy have been assigned to the Lambda function, giving it read access to the production bucket and read/write access to the review app bucket.

### S3 Configuration

The Lambda function requires that a tag named `TargetBucket` be added to the source bucket, which is currently the production Active Storage bucket, with the value(s) of the buckets to copy each new object to.

Tag Name | Required
---|---
TargetBucket | Yes

The value of `TargetBucket` is a space-separated list of buckets to which the objects will be copied. Optionally, the bucket names can contain a @ character followed by a region to indicate that the bucket resides in a different region.

For example: `my-target-bucket1 my-target-bucket1@us-west-2 my-target-bucket3@us-east-1`

### Building the Lambda Package

1. Clone this repo

```
git clone git@github.com:FundThatFlip/S3Copy.git
cd S3Copy
```

2. Install dependencies

```
npm install async
npm install aws-sdk
```

3. Zip up the files inside of the `S3Copy` folder (not the folder itself) using your favourite zipping utility

### Creating the Lambda Function

Follow these steps if you need to replace our existing Lambda function or if you would like to create a new one.

1. Create a new Lambda function on AWS.
2. Upload the ZIP package to your Lambda function.
3. Set your Lambda function's execution role to use an existing role, then assign it the S3Copy IAM role (or the new role you created).
4. Add an event to the source S3 bucket:
 * Event: All object create events
 * Sent to: Lambda function
 * Lambda: S3Copy (or the name of your new Lambda function)
