+++
title = "IAM Policy for access to a Single S3 Bucket"
date = "2013-09-19T01:17:29-04:00"
tags = ["devops", "aws", "iam", "s3"]
type = "post"
aliases = [
    "/blog/2013/08/06/iam-policy-for-access-to-a-single-s3-bucket/"
]
+++

Assuming the bucket name is **my-bucket**<!--more-->
```javascript
{
  "Statement": [
    {
      "Action": [
        "s3:ListAllMyBuckets"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::*"
    },
    {
      "Action": "s3:*",
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::my-bucket", 
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

Source: http://andrewhitchcock.org/?post=325
