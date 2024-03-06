# Deploy your Static Website to S3 Using GitHub Actions

## Create IAM user for deploying

The first thing we need to do is create a new IAM user. This user will be controll access your S3 bucket. We need this user’s credentials for our pipeline.

- Navigate to IAM and from the menu on the left, click on Users
- Choose Add User to create a new user
- Give your user a name (e.g. s3access) and grant it programmatic access
- Next click on Permissions . We will a attach a policy to the user by clicking on ‘Attach existing policies directly”. From the provided list, select AmazonS3FullAccess (this gives s3access fullaccess to S3 bucket)

![add user](/images/user_create.jpg)

- Once the process is complete, you need to generage access keys for communicating github with AWS. Go to IAM > Users > you_username > Security Credentials and add push Create access keys, after download them of save in text file.

![add user](/images/access_key_gen.jpg)

## Create S3 bucket
From the console, navigate to Amazon S3 and click on create bucket
Complete the following actions:

- Enter a bucket name: choose a unique name that mimics a website
- Select an AWS region: Choose a region close to you
![add user](/images/create_s3_bucket.jpg)
- Deselect Block all public access and acknowledge the warning.
![add user](/images/s3_full_access.jpg)

- Click on create bucket

We aren’t done, once your bucket is set up you have to ensure that it can host a website.

Click on your brand new bucket and choose the properties tab. Scroll all the way down to Static Website Hosting, click on edit, and perform the following actions:

- Enable website hosting
- Under hosting type, choose Host a static website
- Under the index document enter ‘index.html’. The error document is optional, you do not have to fill it in.
- Save changes
![add user](/images/stacit_hosting_enable.jpg)

Next, we will set up a bucket policy

In AWS, policies are used to define permissions for resources. They regulate how users interact with resources.

Select your bucket once again and click on the permissions tab. Under Bucket Policy, click on edit and past the following code: 
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
        }
    ]
}
```
YOUR_BUCKET_NAME - need to replace with your bucket name

## Configure github actions
Now that we have created the user, we need to use the credentials in our github account

- Go back to your github repository, and click on Settings

- From the left hand menu, scroll down to Security and click on Secrets and variables, When the dropdown opens, select Actions
- Add three secretes:
    - AWS_S3_BUCKET - bucket name
    - AWS_ACCESS_KEY_ID - access key
    - AWS_SECRET_ACCESS_KEY - secter key

![add user](/images/github_secrets.jpg)

## Add webisite code and github actions code to repository

Now we can add our website code and setup github actions. 
- Open your project root directory in your preferred text editor.
- Create a folder and name it .github (please make sure the .(dot) is part of your .github name), inside the .github folder create a new folder and call it workflows, create a file called main.yml inside .github/workflows folder
- Enter the following code in your main.yml
```yaml
name: Upload Website

on:
  push:
    branches:
    - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v1

    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Deploy static site to S3 bucket
      run: aws s3 sync ./website_code/ s3://${{ secrets.AWS_S3_BUCKET }} --delete
```

The workflow code triggers GitHub action whenever you push on the main branch. The GitHub action once triggered checkout the branch, then configures AWS credentials so that it can use the AWS CLI. `${{ secrets.AWS_ACCESS_KEY_ID }}` and `${{ secrets.AWS_SECRET_ACCESS_KEY }}` fetches its values from the secrets we created earlier. It then syncs your website_code folder to your S3 bucket. Push your code to your repository. After pushig we can see that deploy is successful:

![add user](/images/action_success.jpg)

Now we can check if out website is working, go to you S3 in AWS console and than Propertis and Static website hosting secting, where you can find the publick link for your website
![add user](/images/s3_public_url.jpg)

Click and check: 
![add user](/images/website_work_check.jpg)