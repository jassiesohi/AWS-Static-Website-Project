

# Version 0 - Verifiying all steps

# Manual AWS Static Website Setup Guide with CORS Testing

# Step 1: Create S3 Buckets

# Main Website Bucket (sohiverse.cloud)
1. Go to S3 Console: https://console.aws.amazon.com/s3/
2. Click "Create bucket"
3. Enter bucket name: `sohiverse.cloud`
4. Region: Choose your preferred region (remember it for CloudFront)
5. Uncheck "Block all public access"
6. Acknowledge the warning about making the bucket public
7. Keep other settings default
8. Click "Create bucket"

# WWW Redirect Bucket (www.sohiverse.cloud)
1. Create another bucket named `www.sohiverse.cloud`
2. Use the same settings as above

# Content Bucket (content.sohiverse.cloud)
1. Create another bucket named `content.sohiverse.cloud`
2. Use the same settings as above

 Step 2: Configure Static Website Hosting

# Main Bucket Configuration
1. Open the `sohiverse.cloud` bucket
2. Go to "Properties" tab
3. Scroll down to "Static website hosting"
4. Click "Edit"
5. Choose "Enable"
6. Set Index document: `index.html`
7. Set Error document: `error.html`
8. Save changes
9. Note the endpoint URL

http://sohiverse.cloud.s3-website-us-east-1.amazonaws.com


# WWW Bucket Configuration
1. Open the `www.sohiverse.cloud` bucket
2. Go to "Properties" tab
3. Enable static website hosting
4. Choose "Redirect requests"
5. Enter target bucket: `sohiverse.cloud`
6. Save changes

# Content Bucket Configuration
1. Open the `content.sohiverse.cloud` bucket
2. Enable static website hosting
3. Set Index document: `content.html`
4. Save changes

# Step 3: Configure Bucket Policies

# Main Bucket Policy
1. Go to the `sohiverse.cloud` bucket
2. Click "Permissions" tab
3. Under "Bucket policy", click "Edit"
4. Paste this policy:

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::sohiverse.cloud/*"
        }
    ]
}


# Content Bucket Policy
1. Go to the `content.sohiverse.cloud` bucket
2. Click "Permissions" tab
3. Under "Bucket policy", click "Edit"
4. Paste this policy:

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPublicReadAccess",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::content.sohiverse.cloud/*",
            "Condition": {
                "StringEquals": {
                    "aws:Origin": "https://sohiverse.cloud"
                }
            }
        }
    ]
}


# Step 4: Configure CORS (Content Bucket)

1. Go to the `content.sohiverse.cloud` bucket
2. Click "Permissions" tab
3. Find "Cross-origin resource sharing (CORS)"
4. Click "Edit"
5. Paste this configuration:

[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET",
            "HEAD",
            "POST",
            "PUT"
        ],
        "AllowedOrigins": [
            "https://sohiverse.cloud"
        ],
        "ExposeHeaders": [
            "ETag",
            "x-amz-server-side-encryption",
            "x-amz-request-id",
            "x-amz-id-2"
        ],
        "MaxAgeSeconds": 3000
    }
]




 Step 5: Upload Website Files

# Main Website Files
1. Go to the `sohiverse.cloud` bucket
2. Click "Upload"
3. Upload `index.html` and `error.html` (use the HTML files from the first paste)
4. Maintain file permissions as is

# Content Files
1. Go to the `content.sohiverse.cloud` bucket
2. Click "Upload"
3. Upload `content.html` and `data.json` (use the files from the first paste)
4. Maintain file permissions as is

# Step 6: Set Up SSL Certificate

1. Go to AWS Certificate Manager (ACM)
2. Ensure you're in us-east-1 region (required for CloudFront)
3. Click "Request certificate"
4. Choose "Request a public certificate"
5. Add domain names:
   - sohiverse.cloud
   - *.sohiverse.cloud
6. Choose "DNS validation"
7. Click "Request"
8. Follow the DNS validation process
9. Wait for certificate validation

# Detailed Steps for CloudFront, Route 53, and Testing Setup

 Step 7: Create CloudFront Distributions

# 7A: Main Website Distribution (sohiverse.cloud)
1. Go to CloudFront Console: https://console.aws.amazon.com/cloudfront/
2. Click "Create Distribution"
3. Origin Settings:
   - Origin domain: Select the S3 website endpoint (it ends with s3-website-region.amazonaws.com)
   - Origin name: Keep default
   - Enable Origin Shield: No
   - Origin path: Leave empty
   - Name: Leave default
   - Enable Origin Shield: No
   - Additional settings: Leave defaults

4. Default Cache Behavior Settings:
   - Path pattern: Default (*)
   - Compress objects automatically: Yes
   - Viewer protocol policy: Redirect HTTP to HTTPS
   - Allowed HTTP methods: GET, HEAD
   - Restrict viewer access: No
   - Cache key and origin requests:
     - Cache policy: CachingOptimized
     - Origin request policy: CORS-CustomOrigin
   - Response headers policy: SimpleCORS

5. Distribution Settings:
   - Price class: Use Only North America and Europe
   - Alternate domain names (CNAMEs): sohiverse.cloud
   - Custom SSL certificate: Select your ACM certificate
   - Default root object: index.html
   - Standard logging: Off (or enable if needed)
   - IPv6: Enable

6. Click "Create Distribution"
7. Note down the Distribution Domain Name (e.g., d123456789.cloudfront.net)

# 7B: Content Distribution (content.sohiverse.cloud)
1. Create another distribution
2. Origin Settings:
   - Origin domain: Select content bucket website endpoint
   - Other origin settings: Same as main website

3. Default Cache Behavior Settings:
   - Path pattern: Default (*)
   - Compress objects automatically: Yes
   - Viewer protocol policy: Redirect HTTP to HTTPS
   - Allowed HTTP methods: GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE
   - Cache key and origin requests:
     - Cache policy: CachingDisabled
     - Origin request policy: AllViewer
   - Response headers policy: SimpleCORS

4. Distribution Settings:
   - Same as main website except:
   - Alternate domain names (CNAMEs): content.sohiverse.cloud

5. Click "Create Distribution"
6. Note down the Distribution Domain Name

# Step 8: Configure Route 53

# 8A: Create/Update Hosted Zone
1. Go to Route 53 Console: https://console.aws.amazon.com/route53/
2. Click "Hosted zones"
3. Either:
   - Create hosted zone (if new):
     - Click "Create hosted zone"
     - Domain name: sohiverse.cloud
     - Type: Public hosted zone
     - Click "Create"
   - Or select existing zone for sohiverse.cloud

4. Note the NS records - you'll need these at your domain registrar

# 8B: Create DNS Records
1. In your hosted zone:
2. Create record for main website:
   - Click "Create record"
   - Record name: Leave empty (apex domain)
   - Record type: A
   - Toggle "Alias" on
   - Route traffic to: 
     - Choose "CloudFront distribution"
     - Select your main website distribution
   - Routing policy: Simple routing
   - Click "Create records"

3. Create record for content subdomain:
   - Click "Create record"
   - Record name: content
   - Record type: A
   - Toggle "Alias" on
   - Route traffic to:
     - Choose "CloudFront distribution"
     - Select your content website distribution
   - Routing policy: Simple routing
   - Click "Create records"

# 8C: Update Domain Registrar
1. Go to your domain registrar (e.g., GoDaddy, Namecheap)
2. Find DNS/Nameserver settings
3. Update nameservers to match Route 53 NS records (all 4 of them)
4. Save changes

 Step 9: Testing and Verification

# 9A: Initial Testing
1. Wait 15-30 minutes for CloudFront distributions to deploy
2. Check distribution status in CloudFront console:
   - Status should be "Deployed"
   - State should be "Enabled"

# 9B: DNS Verification
1. Use dig or nslookup to verify DNS propagation:
   
   dig sohiverse.cloud
   dig content.sohiverse.cloud
   
2. Verify they point to your CloudFront distributions

# 9C: Website Testing
1. Test main website:
   - Open https://sohiverse.cloud
   - Verify SSL certificate (lock icon in browser)
   - Check if index.html loads properly
   - Try a non-existent page to test error.html

2. Test CORS functionality:
   - Open browser developer tools (F12)
   - Click "Load HTML Content" button
   - Verify in Network tab:
     - Request shows CORS headers
     - No CORS errors in console
   - Click "Load JSON Data" button
   - Verify data loads without CORS errors

3. Test overall functionality:
   - All buttons work
   - Content loads from content.sohiverse.cloud
   - No console errors
   - SSL secure on both domains

 Common Issues & Solutions

# CloudFront Issues:
1. 403 Forbidden:
   - Check bucket policy
   - Verify Origin settings
   - Ensure static website hosting is enabled

2. Steps to invalidate CloudFront cache:

1. Sign in to the AWS Management Console
2. Go to CloudFront console (https://console.aws.amazon.com/cloudfront/)
3. Select your distribution
4. Click on the "Invalidations" tab
5. Click "Create Invalidation"
6. Enter the path pattern(s):
   - Use `/*` to invalidate everything
   - Or specific paths like `/content.html`, `/data.json`
7. Click "Create Invalidation" to submit

# Common path patterns:
- `/*` - Invalidates all files
- `/images/*` - Invalidates all files in the images directory
- `/images/logo.png` - Invalidates a specific file
- `/*.html` - Invalidates all HTML files

# Using AWS CLI (alternative method)

# Get your distribution ID
aws cloudfront list-distributions --query 'DistributionList.Items[*].[Id,DomainName]' --output table

# Create invalidation for all files
aws cloudfront create-invalidation \
    --distribution-id YOUR_DISTRIBUTION_ID \
    --paths "/*"

# Check invalidation status
aws cloudfront get-invalidation \
    --distribution-id YOUR_DISTRIBUTION_ID \
    --id INVALIDATION_ID

# Create invalidation for specific files
aws cloudfront create-invalidation \
    --distribution-id YOUR_DISTRIBUTION_ID \
    --paths "/content.html" "/data.json" "/images/*"



3. CORS Errors:
   - Verify CORS configuration in content bucket
   - Check CloudFront behavior settings
   - Ensure headers are being forwarded

# DNS Issues:
1. Website not accessible:
   - Check if NS records match at registrar
   - Verify A records in Route 53
   - Wait for DNS propagation (up to 48 hours)

2. SSL Certificate Errors:
   - Ensure certificate is validated
   - Check if correct certificate is selected in CloudFront
   - Verify CNAME records match certificate domains

# Testing Checklist:
- [ ] Main website loads over HTTPS
- [ ] Content subdomain loads over HTTPS
- [ ] CORS requests succeed
- [ ] Error page works
- [ ] All interactive features function
- [ ] No console errors
- [ ] Both domains use valid SSL