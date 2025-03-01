## Basic Policy Structure
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:Action1", "s3:Action2"],
      "Resource": ["arn:aws:s3:::bucket-name"]
    }
  ]
}
```

## Common Policy Actions

### Bucket-Level Actions
| Action | Description |
|--------|-------------|
| `s3:ListBucket` | List objects in bucket |
| `s3:GetBucketLocation` | Get bucket region |
| `s3:CreateBucket` | Create new buckets |
| `s3:DeleteBucket` | Delete buckets |
| `s3:PutBucketPolicy` | Set bucket policies |
| `s3:GetBucketPolicy` | Read bucket policies |
| `s3:PutBucketCORS` | Set CORS configuration |
| `s3:GetBucketCORS` | Read CORS configuration |

### Object-Level Actions
| Action | Description |
|--------|-------------|
| `s3:GetObject` | Download/read objects |
| `s3:PutObject` | Upload/write objects |
| `s3:DeleteObject` | Delete objects |
| `s3:ListMultipartUploadParts` | List parts of multipart upload |
| `s3:AbortMultipartUpload` | Cancel multipart upload |

### Admin Actions
| Action | Description |
|--------|-------------|
| `s3:*` | All S3 actions (admin) |
| `admin:*` | All admin actions |

## Resource Patterns

### Bucket Resources
```
"Resource": ["arn:aws:s3:::bucket-name"]
```

### Object Resources
```
"Resource": ["arn:aws:s3:::bucket-name/*"]
```

### Multiple Buckets
```
"Resource": [
  "arn:aws:s3:::bucket1",
  "arn:aws:s3:::bucket2"
]
```

### All Buckets (Admin Only)
```
"Resource": ["arn:aws:s3:::*"]
```

## Common Policy Templates

### Read-Only Access
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket", "s3:GetBucketLocation"],
      "Resource": ["arn:aws:s3:::bucket-name"]
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::bucket-name/*"]
    }
  ]
}
```

### Read-Write Access
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket", "s3:GetBucketLocation"],
      "Resource": ["arn:aws:s3:::bucket-name"]
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": ["arn:aws:s3:::bucket-name/*"]
    }
  ]
}
```

### Public Access Policy (Bucket Policy)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": ["*"]},
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::bucket-name/*"]
    }
  ]
}
```

### CMS Application Access
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket", "s3:GetBucketLocation"],
      "Resource": ["arn:aws:s3:::bucket-name"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListMultipartUploadParts",
        "s3:AbortMultipartUpload"
      ],
      "Resource": ["arn:aws:s3:::bucket-name/*"]
    }
  ]
}
```

## CORS Configuration Template
```json
{
  "CORSRules": [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
      "AllowedOrigins": ["http://localhost:3000", "https://yourdomain.com"],
      "ExposeHeaders": ["ETag", "Content-Length", "Content-Type"]
    }
  ]
}
```

## Command Line Examples

### Set Policy Using mc
```bash
# Save policy to file
echo '{POLICY_JSON}' > policy.json

# Apply policy to user
mc admin policy add myminio policy-name policy.json

# Assign policy to user
mc admin policy set myminio policy-name user=username
```

### Set Bucket Policy Using mc
```bash
# Make bucket public for downloads
mc anonymous set download myminio/bucket-name

# Apply custom bucket policy
mc anonymous set-json /path/to/policy.json myminio/bucket-name
```

### Set CORS Using mc
```bash
# Save CORS config to file
echo '{CORS_JSON}' > cors.json

# Apply CORS config
mc admin bucket cors set myminio/bucket-name cors.json
```