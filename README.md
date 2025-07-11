# Secure Static Website Hosting with S3 and CloudFront (OAC)

This project demonstrates how to securely host a static website using Amazon S3 and serve it over HTTPS using Amazon CloudFront with Origin Access Control (OAC). The S3 bucket is fully private, and content is accessible only via CloudFront.

## Features

* Static HTML/CSS website hosted on Amazon S3
* Private S3 bucket with no public access
* CloudFront distribution using Origin Access Control (OAC)
* HTTPS support via default CloudFront certificate
* Fast, secure, globally cached content

## Tech Stack

| Service      | Purpose                               |
| ------------ | ------------------------------------- |
| Amazon S3    | Static website storage                |
| CloudFront   | Secure global CDN + HTTPS             |
| OAC (OAC ID) | Secure S3 access from CloudFront only |
| AWS CLI      | Infrastructure automation             |

## How to Deploy

### 1. Prepare Static Files

Clone your static website repo:

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

Ensure `index.html` and other assets are in the root or inside a folder you can target.

---

### 2. Create an S3 Bucket

```bash
aws s3api create-bucket \
  --bucket your-unique-bucket-name \
  --region us-east-1
```

### 3. Enable Static Website Hosting

```bash
aws s3 website s3://your-unique-bucket-name/ --index-document index.html
```

### 4. Upload Your Site

```bash
aws s3 sync . s3://your-unique-bucket-name --delete
```

---

### 5. Make the S3 Bucket Private

```bash
aws s3api put-public-access-block \
  --bucket your-unique-bucket-name \
  --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

aws s3api delete-bucket-policy --bucket your-unique-bucket-name
```

---

### 6. Create an Origin Access Control (OAC)

```bash
aws cloudfront create-origin-access-control \
  --origin-access-control-config '{
    "Name": "secure-oac-access",
    "Description": "Allow CloudFront access to S3",
    "SigningBehavior": "always",
    "SigningProtocol": "sigv4",
    "OriginAccessControlOriginType": "s3"
  }'
```

Copy the returned `Id` for later.

---

### 7. Create a CloudFront Distribution (with OAC)

Create a file named `cf-config.json`:

Then run:

```bash
aws cloudfront create-distribution \
  --distribution-config file://cf-config.json
```

---

### 8. Set the Correct Bucket Policy for OAC

```bash
aws s3api put-bucket-policy \
  --bucket your-unique-bucket-name \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "AllowCloudFrontServicePrincipalReadOnly",
        "Effect": "Allow",
        "Principal": { "Service": "cloudfront.amazonaws.com" },
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::your-unique-bucket-name/*",
        "Condition": {
          "StringEquals": {
            "AWS:SourceArn": "arn:aws:cloudfront::<account-id>:distribution/<distribution-id>"
          }
        }
      }
    ]
  }'
```

---

### 9. Test Your Website

Access your site via:

```
https://<your-distribution-id>.cloudfront.net
```

You should see your static website, now securely hosted and locked behind CloudFront.

---

## Folder Structure

```
secure-cloudfront-site/
├── index.html
├── about.html
├── assets/
├── cf-config.json
├── README.md
```

## Author

**Surya**
DevOps Engineer | Cloud Enthusiast
[LinkedIn](https://www.linkedin.com/) • [GitHub](https://github.com/)

## License

This project is open source under the [MIT License](LICENSE)
