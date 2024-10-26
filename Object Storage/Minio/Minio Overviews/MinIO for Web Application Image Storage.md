## Core Concepts
```javascript
// MinIO is an object storage server compatible with Amazon S3 API
// Think of it as your own private S3-like service
```
MinIO provides S3-compatible object storage, meaning you can store and retrieve files using an API similar to Amazon S3. This is perfect for applications needing to store user-uploaded content like images, while maintaining control over your storage infrastructure.

## Installation & Setup

### Docker Setup (Recommended)
```bash
# Pull and run MinIO container
docker run -p 9000:9000 -p 9001:9001 \
  --name minio \
  -v /path/to/data:/data \
  -e "MINIO_ROOT_USER=admin" \
  -e "MINIO_ROOT_PASSWORD=supersecret" \
  minio/minio server /data --console-address ":9001"
```
This Docker command sets up MinIO server locally. Port 9000 is for API operations, while 9001 provides access to the web console. The volume mount (-v) ensures data persists even if the container is removed. Environment variables set up the initial admin credentials.

### Express Backend Setup
```bash
# Install required packages
npm install minio multer

# Basic MinIO client setup
const Minio = require('minio');

const minioClient = new Minio.Client({
  endPoint: 'localhost',  // Your MinIO server address
  port: 9000,
  useSSL: false,         // Set to true if using HTTPS
  accessKey: 'admin',    // From your MinIO setup
  secretKey: 'supersecret'
});
```
This initializes the MinIO client in your Express backend. Multer is needed for handling multipart/form-data (file uploads). The client configuration connects to your MinIO server using the credentials you set during setup.

### Upload Handler (Express)
```javascript
// Utility function to upload file to MinIO
const uploadToMinio = async (file, bucketName) => {
  try {
    // Check if bucket exists, create if it doesn't
    const bucketExists = await minioClient.bucketExists(bucketName);
    if (!bucketExists) {
      await minioClient.makeBucket(bucketName);
    }

    // Generate unique filename
    const filename = `${Date.now()}-${file.originalname}`;
    
    // Upload file
    await minioClient.putObject(
      bucketName,
      filename,
      file.buffer,
      file.size,
      file.mimetype
    );

    // Return the file URL
    return `http://localhost:9000/${bucketName}/${filename}`;
  } catch (error) {
    throw new Error(`MinIO upload failed: ${error.message}`);
  }
};

// Express route handler
app.post('/upload', multer().single('image'), async (req, res) => {
  try {
    const fileUrl = await uploadToMinio(req.file, 'images');
    res.json({ url: fileUrl });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```
This is your main upload handler. The utility function checks for bucket existence (creates if needed), generates a unique filename using timestamp, and uploads the file. The Express route uses multer to parse the incoming file and returns the URL where the file can be accessed. This URL can then be stored in your PostgreSQL database.

### React Frontend
```javascript
// Example upload component
const ImageUpload = () => {
  const handleUpload = async (event) => {
    const file = event.target.files[0];
    const formData = new FormData();
    formData.append('image', file);

    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData,
      });
      const data = await response.json();
      // Store URL in Redux or state management
      console.log('Image URL:', data.url);
    } catch (error) {
      console.error('Upload failed:', error);
    }
  };

  return (
    <input 
      type="file" 
      accept="image/*" 
      onChange={handleUpload} 
    />
  );
};
```
A basic React component that handles file selection and upload. It creates a FormData object (necessary for file uploads) and sends it to your Express backend. The returned URL can be stored in Redux state or used directly in your application.

### Configuration
```javascript
// MinIO CORS configuration (run in MinIO client)
mc admin config set minio cors='["*"]'

// Better CORS configuration for production
mc admin config set minio cors='[
  {
    "origin": "https://yourapp.com",
    "methods": ["GET", "PUT", "POST"],
    "allowedHeaders": ["*"]
  }
]'
```
CORS configuration is crucial for allowing your frontend to interact with MinIO. The first example is permissive (good for development), while the second is more secure for production use, limiting access to your specific domain.

### Production Considerations
```javascript
// Environment variables setup
const minioClient = new Minio.Client({
  endPoint: process.env.MINIO_ENDPOINT,
  port: parseInt(process.env.MINIO_PORT),
  useSSL: process.env.NODE_ENV === 'production',
  accessKey: process.env.MINIO_ACCESS_KEY,
  secretKey: process.env.MINIO_SECRET_KEY
});
```
Production setup should always use environment variables for sensitive information. This example shows how to configure the MinIO client using environment variables, enabling different configurations for development and production environments.

### Common Operations
```javascript
// Delete file
await minioClient.removeObject(bucketName, filename);

// Get file metadata
const stat = await minioClient.statObject(bucketName, filename);

// Generate temporary URL (for private buckets)
const presignedUrl = await minioClient.presignedGetObject(
  bucketName, 
  filename, 
  24 * 60 * 60  // URL expires in 24 hours
);
```
These are common operations you'll need: deleting files (useful for user account deletion or image updates), getting file metadata (size, type, etc.), and generating temporary URLs for private files. Presigned URLs are especially useful when you want to restrict file access but temporarily allow it for authorized users.

### Error Handling
```javascript
// Implement proper error handling
const handleMinioError = (error) => {
  if (error.code === 'NoSuchBucket') {
    // Handle bucket not found
  } else if (error.code === 'NoSuchKey') {
    // Handle file not found
  } else {
    // Handle other errors
  }
};
```
Proper error handling is crucial for a robust application. This pattern shows how to handle common MinIO errors, allowing your application to gracefully handle issues like missing buckets or files.