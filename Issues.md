Common Issues Faced & Resolutions

1. AccessDenied from S3

Cause: Bucket was private, but CloudFront didn’t have permission.
Fix: Added OAC and correct bucket policy with AWS:SourceArn matching CloudFront distribution.

2. NoSuchBucket during policy updates

Cause: Minor typos in bucket name.
Fix: Double-checked with aws s3 ls and re-ran with correct name.

3. CloudFront still showing AccessDenied after setup

Cause: OAC ID was missing or incorrect in the CloudFront config.
Fix: Recreated distribution with proper OriginAccessControlId.

4. Forgetting to disable before deleting CloudFront

Fix: Ran update-distribution to disable before deletion.
