
## 1. Creating the Bucket

### Using the Minio Web Console:

1. **Log in** to your Minio Console (typically at `http://your-minio-server:9001`)
2. In the left sidebar, click on **Buckets**
3. Click the **Create Bucket** button
4. Enter the bucket name: `hills-house-media`
5. Click **Create Bucket**

### Using the Minio Client (mc):

```bash
# First, ensure mc is configured to connect to your Minio instance
mc alias set myminio https://storage.sbtl.dev YOUR_ACCESS_KEY YOUR_SECRET_KEY

# Create the bucket
mc mb myminio/hills-house-media
```

## 2. Setting Up Access Policy for the Bucket

### Make the Bucket Public (for media files):

1. In the Minio Console, go to **Buckets**
2. Click on the `hills-house-media` bucket
3. Click on **Manage**
4. Select the **Access Policy** tab
5. Choose **Public** from the dropdown, or use a custom policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": ["*"]},
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::hills-house-media/*"]
    }
  ]
}
```

1. Click **Add** or **Save**

### Using mc to set public policy:

```bash
mc anonymous set download myminio/hills-house-media
```

## 3. Configuring CORS for the Bucket

CORS configuration is essential for Payload CMS to upload files from the admin interface.

### Using the Minio Console:

1. Go to **Buckets**
2. Click on the `hills-house-media` bucket
3. Click on **Manage**
4. Select the **CORS Configuration** tab
5. Add a new CORS rule:

```json
{
  "CORSRules": [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
      "AllowedOrigins": [
        "http://localhost:3000", 
        "https://itshillshouse.com",
        "https://admin.itshillshouse.com",
        "http://localhost:8000"
      ],
      "ExposeHeaders": ["ETag", "Content-Length", "Content-Type"]
    }
  ]
}
```

1. Click **Save**

### Using mc to set CORS:

```bash
# Create a cors.json file with the above content
echo '{
  "CORSRules": [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
      "AllowedOrigins": ["http://localhost:3000", "https://itshillshouse.com", "https://admin.itshillshouse.com", "http://localhost:8000"],
      "ExposeHeaders": ["ETag", "Content-Length", "Content-Type"]
    }
  ]
}' > cors.json

# Apply the CORS configuration
mc admin bucket cors set myminio/hills-house-media cors.json
```

## 4. Creating a Dedicated User for Payload CMS

### Using the Minio Console:

1. Go to **Identity → Users**
2. Click **Create User**
3. Fill in:
   - **Username**: `hills-house-payload`
   - **Password**: Generate a strong password
4. Click **Save**
5. Note the **Access Key** and **Secret Key** that are generated

## 5. Creating a Policy for the User

### Using the Minio Console:

1. Go to **Identity → Policies**
2. Click **Create Policy**
3. Name it `hills-house-payload-policy`
4. Enter the policy JSON:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetBucketLocation"
      ],
      "Resource": [
        "arn:aws:s3:::hills-house-media"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::hills-house-media/*"
      ]
    }
  ]
}
```

1. Click **Save**

## 6. Assigning the Policy to the User

### Using the Minio Console:

1. Go to **Identity → Users**
2. Click on the `hills-house-payload` user
3. Go to the **Policies** tab
4. Click **Attach Policies**
5. Select the `hills-house-payload-policy`
6. Click **Attach**

## 7. Testing the Bucket Access

### Using the Minio Client:

```bash
# Configure mc with the new user credentials
mc alias set hillshouse https://storage.sbtl.dev HILLS_HOUSE_ACCESS_KEY HILLS_HOUSE_SECRET_KEY

# Test listing the bucket
mc ls hillshouse/hills-house-media

# Test uploading a file
mc cp test-image.jpg hillshouse/hills-house-media/test-image.jpg

# Test downloading a file
mc cp hillshouse/hills-house-media/test-image.jpg ./downloaded-test-image.jpg

# Test that the file is publicly accessible
curl -I https://storage.sbtl.dev/hills-house-media/test-image.jpg
```

## 8. Configuring Payload CMS to Use the Bucket

Add these environment variables to your Payload CMS project:

```
HILLS_HOUSE_MINIO_ENDPOINT=https://storage.sbtl.dev
HILLS_HOUSE_MINIO_ACCESS_KEY=your-hills-house-access-key
HILLS_HOUSE_MINIO_SECRET_KEY=your-hills-house-secret-key
HILLS_HOUSE_MINIO_PUBLIC_URL=https://storage.sbtl.dev/hills-house-media
```

Update your Payload configuration:

```typescript
// payload-cms/payload.config.ts
import { buildConfig } from 'payload/config';
import path from 'path';
import { S3Client } from '@aws-sdk/client-s3';

// S3/Minio configuration
const s3Client = new S3Client({
  region: 'us-east-1', // This can be any value for Minio
  endpoint: process.env.HILLS_HOUSE_MINIO_ENDPOINT,
  forcePathStyle: true, // Required for Minio
  credentials: {
    accessKeyId: process.env.HILLS_HOUSE_MINIO_ACCESS_KEY,
    secretAccessKey: process.env.HILLS_HOUSE_MINIO_SECRET_KEY,
  },
});

export default buildConfig({
  serverURL: process.env.PAYLOAD_PUBLIC_SERVER_URL,
  admin: {
    user: 'users',
  },
  collections: [
    // Your collections here
  ],
  upload: {
    limits: {
      fileSize: 10000000, // 10MB
    },
    // Configure S3/Minio storage
    useTempFiles: true,
    storage: {
      s3: {
        bucket: 'hills-house-media',
        client: s3Client,
        prefix: '', // Optional prefix for all files
      },
    },
    // This is important - it determines how URLs are generated
    staticURL: '/media',
    // This tells Payload how to generate public URLs for files
    generateFileURL: ({ filename }) => {
      return `${process.env.HILLS_HOUSE_MINIO_PUBLIC_URL}/${filename}`;
    },
  },
  typescript: {
    outputFile: path.resolve(__dirname, 'payload-types.ts'),
  },
});
```

## 9. Configuring Your Next.js App

Add this environment variable to your Next.js project:

```
NEXT_PUBLIC_MINIO_URL=https://storage.sbtl.dev/hills-house-media
```

Use it in your components:

```tsx
<Image 
  src={`${process.env.NEXT_PUBLIC_MINIO_URL}/${show.flyer.filename}`}
  alt={show.title}
  width={800}
  height={600}
/>
```

## 10. Troubleshooting Common Issues

### CORS Errors
If you see CORS errors in the browser console when uploading:
- Double-check your CORS configuration
- Ensure all domains (including localhost) are in the AllowedOrigins
- Verify that the correct HTTP methods are allowed

### Access Denied
If you get "Access Denied" errors:
- Verify the policy is correctly attached to the user
- Check that the bucket name is correct in all configurations
- Ensure the access key and secret key are correct

### File URLs Not Working
If files upload but URLs don't work:
- Check the `generateFileURL` function in your Payload config
- Verify that `HILLS_HOUSE_MINIO_PUBLIC_URL` is set correctly
- Ensure the bucket is set to public access for GET operations

## Summary

You've now set up a secure, dedicated Minio bucket for your Hills House Payload CMS project with:
- Proper bucket creation
- Secure but sufficient access policies
- CORS configuration for uploads
- A dedicated user with limited permissions
- Configuration for both Payload CMS and your Next.js frontend

This setup follows security best practices while ensuring all functionality works correctly.