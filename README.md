!!!THIS TUTORIAL ASSUMES YOU HAVE BASIC CLOUD KNOWLEGDE!!!

In this tutorial, you create and configure a Lambda function that resizes images added to an Amazon Simple Storage Service (Amazon S3) bucket. When you add an image file to your bucket, Amazon S3 invokes your Lambda function. The function then creates a thumbnail version of the image and outputs it to a different Amazon S3 bucket.

The runtime used here is Python 3.9. Youcan follow the documentation from AWS at the end if you prefer a different Lambda Runtime.

Below are the thiings we need to do to accomplish this:

- Create source and destination Amazon S3 buckets and upload a sample image.
In my case, I named my source s3 bucket "source-bucket-237" and my output bucket "source-bucket-237-resized". Be sure to name your buckets like this for your lambda logic gto function properly. Once your buclets are set up, upload a test image into the source bucket (source-bucket-237), we will need this later dot testing

- Create a Lambda function that resizes an image and outputs a thumbnail to an Amazon S3 bucket.
To create out lambda function, we need to get a few things ready. Let's create our lambda execution role and a policy and attach the policy to the role.
I name my policy "LambdaS3Policy" and my role "LambdaS3Role"

use this custom policy while creating the above policy:


    ----------------------------------------------

    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:PutLogEvents",
                "logs:CreateLogGroup",
                "logs:CreateLogStream"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::*/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::*/*"
        }
    ]
}

----------------------------------------------

when done, attaxch the policy to the lambda role(LambdaS3Role) created above.

Here's where it gets intetresting. We need to create a deployment package.

in your IDE create a folder for this project. 

mkdir auto-process-pictures-with-lambda && cd auto-process-pictures-with-lambda

Once in your project directory, create a file for your lambda code : sudo vi lambda_function.py

Past this python code in the file created and save 

    -----------------------------------------
    
import boto3
import os
import sys
import uuid
from urllib.parse import unquote_plus
from PIL import Image
import PIL.Image
            
s3_client = boto3.client('s3')
            
def resize_image(image_path, resized_path):
  with Image.open(image_path) as image:
    image.thumbnail(tuple(x / 2 for x in image.size))
    image.save(resized_path)
            
def lambda_handler(event, context):
  for record in event['Records']:
    bucket = record['s3']['bucket']['name']
    key = unquote_plus(record['s3']['object']['key'])
    tmpkey = key.replace('/', '')
    download_path = '/tmp/{}{}'.format(uuid.uuid4(), tmpkey)
    upload_path = '/tmp/resized-{}'.format(tmpkey)
    s3_client.download_file(bucket, key, download_path)
    resize_image(download_path, upload_path)
    s3_client.upload_file(upload_path, '{}-resized'.format(bucket), 'resized-{}'.format(key))
    

    -----------------------------------------------
    


While making sure you are in the same directory(auto-process-pictures-with-lambda) above create a subdirectory called "package" and install the PIL and AWS SDK for python. Copying and pasting the below commands creates the filder and installs the dependencies:

mkdir package
pip install \
--platform manylinux2014_x86_64 \
--target=package \
--implementation cp \
--python-version 3.9 \
--only-binary=:all: --upgrade \
pillow boto3

Now that the dependencies have been created in our working directory, we need to zip the whole working directory and this is what we will use as our lambda code:

Use the following command to cd back to tyhe parent folder and zip the folder content.

cd package
zip -r ../lambda_function.zip .
cd ..
zip lambda_function.zip lambda_function.py



- Configure a Lambda trigger that invokes your function when objects are uploaded to your source bucket.

Now let's create our lambda function in the same region as our two buckets. I called my bucket "CreateThumbnail"

assign the LambdaS3Role created above as the lambda execution role.

Select python3.9 for Runtime

change timout to 1 minute

add memory of lambda to 256

Architecture x86_64

For the lambda code, we need to upload the file we zipped above after we installed the dependencies. This is goingg to be your lambda_function.zip file

Once you upload the file, save it.

Set your s3 "source-bucket-237" to be the trigger for your lambda, select "All object create events" and save


- Test your function, first with a dummy event, and then by uploading an image to your source bucket.

Follow the AWS Documentation posted below to test your function to make sure its working properly. upload a few pictures in your sources buuclet aand check your lambda events under "monitor" section and then confirm in the resized(output) bucket that the smaller(Thumbnail) piictures are in there.


- By completing these steps, you’ll learn how to use Lambda to carry out a file processing task on objects added to an Amazon S3 bucket. You can complete this tutorial using the AWS Command Line Interface (AWS CLI) or the AWS Management Console.


CONGRATULATIONS, YOU HAVE SUCCESFULLY DEPLOYED A SOLUTION TO AUTOMATICALLY PROCESS FILES UPLOADED TO S3 WIITH LAMBDA AND UPLOAD THEM IN A DIFFERENT BUCKET.

Links: 
https://docs.aws.amazon.com/lambda/latest/dg/with-s3-tutorial.html



