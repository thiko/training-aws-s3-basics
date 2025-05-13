# Amazon S3 Exercises

## Exercise 1: Creating and Configuring an S3 Bucket

### Objective
Learn how to create an Amazon S3 bucket and perform basic configurations.

### Steps

1. **Create a bucket**
   - Sign in to the AWS Management Console
   - Navigate to the S3 service
   - Click "Create bucket"
   - Enter a globally unique name: `training-[YourName]-[Date]`
   - Select region `eu-central-1` (Frankfurt)
   - Keep "Block all public access" enabled (we'll modify this later)
   - Keep the remaining default settings
   - Click "Create bucket"

2. **Configure bucket properties**
   - Select your newly created bucket
   - Go to the "Properties" tab
   - Enable "Static website hosting"
   - Select "Host a static website" and enter `index.html` as the index document
   - Save the settings

3. **Upload files**
   - Create a simple `index.html` file locally with the following content:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>My S3 Exercise</title>
     </head>
     <body>
         <h1>Welcome to my S3 Exercise!</h1>
         <p>Date: [Current Date]</p>
     </body>
     </html>
     ```
   - Go to the "Objects" tab and click "Upload"
   - Upload the `index.html` file
   - Also upload an image of your choice

4. **Adjust bucket policy**
   - Disable "Block all public access" in the bucket properties and confirm
   - Go to the "Permissions" tab
   - Click on "Bucket policy"
   - Insert the following policy (replace `[YourBucketName]` with your actual bucket name):
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Sid": "PublicReadForGetBucketObjects",
           "Effect": "Allow",
           "Principal": "*",
           "Action": "s3:GetObject",
           "Resource": "arn:aws:s3:::[YourBucketName]/*"
         }
       ]
     }
     ```
   - Save the policy
   
   > **NOTE**: Making S3 buckets publicly accessible as demonstrated in this exercise is NOT recommended for production environments. In real-world scenarios, you should use Amazon CloudFront with an Origin Access Identity (OAI) to securely serve content from S3 without making the bucket publicly accessible.

5. **Check the website**
   - Return to the "Properties" tab
   - Scroll down to "Static website hosting"
   - Open the displayed website URL in a new browser tab
   - Verify that your website is displayed correctly

### Verification
- Your static website is accessible through the S3 website URL
- The bucket policy allows public read access
- The uploaded files are accessible via the URL

---

## Exercise 2: Versioning, Storage Classes, and Lifecycle Policies

### Objective
Learn how to configure versioning, different storage classes, and lifecycle policies for S3 buckets.

### Steps

1. **Enable versioning**
   - Navigate to your S3 bucket
   - Go to the "Properties" tab
   - Scroll to "Bucket Versioning"
   - Click "Edit" and enable versioning
   - Save the changes

2. **Create multiple versions**
   - Go to the "Objects" tab
   - Open your `index.html` file in a text editor and modify the content:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>My S3 Exercise - Version 2</title>
     </head>
     <body>
         <h1>Welcome to my S3 Exercise!</h1>
         <p>Date: [Current Date]</p>
         <p>This is version 2 of the file.</p>
     </body>
     </html>
     ```
   - Upload the edited file with the same name
   - Repeat this process with another change for version 3

3. **Manage versions**
   - Enable "Show versions" in the Objects view
   - Verify that multiple versions of `index.html` are displayed
   - Download an older version to confirm that the original content is preserved
   - Restore an older version by selecting it and choosing "Delete" for the newest version

4. **Configure storage classes**
   - Upload a new file (e.g., `test-file.txt`) with the standard storage
   - Select the file and click "Actions" → "Edit storage class"
   - Change the storage class to "S3 Standard-IA" (Infrequent Access)
   - Save the change

5. **Create a lifecycle policy**
   - Go to the "Management" tab
   - Click on "Create lifecycle rule"
   - Name the rule (e.g., "Archive-and-Delete-Rule")
   - Select "Apply to all objects in the bucket"
   - Configure the following actions:
     - After 30 days: Transition to S3 Standard-IA
     - After 90 days: Transition to S3 Glacier
     - After 365 days: Expire current versions of objects AND Permanently delete noncurrent versions of objects
   - Additionally enable "Delete expired object delete markers"
   - Create the rule

### Verification
- Versioning is enabled and multiple versions of a file exist
- Different storage classes have been configured for objects
- A lifecycle policy has been successfully set up

---

## Exercise 3: Pre-Signed URLs and Access Control

### Objective
Learn how to grant temporary access to private S3 objects using pre-signed URLs and configure advanced access control.

### Steps

1. **Set up a private bucket**
   - Create a new bucket named `private-training-[YourName]-[Date]`
   - Make sure "Block all public access" is enabled
   - Upload some test files (images, PDFs, or TXT files)

2. **Create a pre-signed URL using the AWS CLI**
   - Open a terminal or command prompt
   - Run the following command (replace the placeholders accordingly):
     ```bash
     aws s3 presign s3://private-training-[YourName]-[Date]/[FileName] --expires-in 300
     ```
   - The command returns a URL that is valid for 5 minutes (300 seconds)

3. **Test the pre-signed URL**
   - Copy the generated URL
   - Open a browser and paste the URL
   - Verify that you can access the file
   - Wait more than 5 minutes and try to access the URL again
   - Confirm that access is denied after the time expires

4. **Set up CORS configuration**
   - Navigate to "Cross-origin resource sharing (CORS)"
   - Click "Edit"
   - Add the following CORS configuration:
     ```json
     [
       {
         "AllowedHeaders": ["*"],
         "AllowedMethods": ["GET"],
         "AllowedOrigins": ["*"],
         "ExposeHeaders": []
       }
     ]
     ```
   - Save the configuration

   > **Understanding CORS**: Cross-Origin Resource Sharing (CORS) is a security feature implemented by browsers that restricts web pages from making requests to a different domain than the one that served the original page. By configuring CORS on your S3 bucket, you're allowing specified origins (websites) to load resources from your bucket. This is essential when you want to host assets like images, fonts, or scripts on S3 that will be used by web applications hosted on different domains.

5. **Create a pre-signed URL for upload**
   - Create a simple Java application `GeneratePresignedUrlExample.java`:
     ```java
      public class GeneratePresignedUrlExample {

         public static void main(String[] args) {

            var preSignBuilder = S3Presigner.builder()
                     .region(Region.EU_CENTRAL_1)
                     .credentialsProvider(DefaultCredentialsProvider.create());
            // optionally: use a ProfileCredentialsProvider to use profiles

            try (var presigner = preSignBuilder.build()) {

                  // Bucket und Key definieren
                  String bucketName = "private-training-[YourName]-[Date]";
                  String keyName = "uploaded-via-presigned-url.txt";

                  PutObjectRequest objectRequest = PutObjectRequest.builder()
                        .bucket(bucketName)
                        .key(keyName)
                        .build();

                  // Generate Pre-signed URL for upload (PUT)
                  // Expires in 5 Minutes
                  PutObjectPresignRequest presignRequest = PutObjectPresignRequest.builder()
                        .signatureDuration(Duration.ofMinutes(5))
                        .putObjectRequest(objectRequest)
                        .build();

                  var presignedUrl = presigner.presignPutObject(presignRequest).url().toString();

                  System.out.printf("Pre-signed upload URL (valid for %d minutes): \n", presignRequest.signatureDuration().toMinutes());
                  System.out.println(presignedUrl);

            }
         }
      }
     ```
   - You can copy the dependencies from: https://github.com/thiko/training-aws-s3-presigned-url
   - Run the script or use the AWS CLI to generate an upload URL
   - Test the upload with a REST client like Postman or curl
   ```sh
        # Create a test file
        echo "This is a test file uploaded via pre-signed URL" > test-upload.txt

        # Upload the file using curl
        curl -X PUT -T test-upload.txt "YOUR_PRESIGNED_URL_HERE"

        # If the upload is successful, you should see an empty response with status code 200
        # You can verify the upload in the AWS console by checking your S3 bucket
    ```

### Verification
- You can grant temporary access to private objects via pre-signed URLs
- You understand the effect of CORS
- You can create pre-signed URLs for both downloads and uploads

---

### Additional Task (if time permits)

**Setting up replication between buckets**

1. Create a third bucket in a different region (e.g., `eu-west-1`)
2. Enable versioning for both buckets
3. Configure a replication rule from the original to the new bucket
4. Test the replication by uploading files and verifying their appearance in the destination bucket

This additional task demonstrates how data replication across regions can be configured for disaster recovery or global availability.