# Serverless Email Marketing Application

This repository contains the code and setup instructions for a serverless email marketing application built using AWS services. The application allows users to submit email addresses via an HTML form and sends marketing emails using AWS SES. The architecture leverages AWS API Gateway, EventBridge, Lambda, S3, SES, and Route 53.

## Architecture

<img src="Diagram.png.png" alt="drawing" width="500"/>

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Setup Instructions](#setup-instructions)
  - [1. Create S3 Bucket](#1-create-s3-bucket)
  - [2. Setup SES](#2-setup-ses)
  - [3. Create Lambda Function](#3-create-lambda-function)
  - [4. Setup API Gateway](#4-setup-api-gateway)
  - [5. Setup EventBridge](#5-setup-eventbridge)
  - [6. Setup Route 53](#6-setup-route-53)
  - [7. Create HTML Form](#7-create-html-form)
- [Adding Contacts](#adding-contacts)
- [Email Template](#email-template)
- [How It Works](#how-it-works)
- [License](#license)

## Features

- Serverless architecture
- Secure email submission via API Gateway
- Asynchronous email processing with EventBridge
- Marketing emails sent using SES
- HTML form for email submission

## Requirements

- AWS Account
- AWS CLI configured
- Basic knowledge of AWS services
- Python 3.8+

## Setup Instructions

### 1. Create S3 Bucket

1. Go to the S3 console.
2. Click on "Create bucket".
3. Enter a unique bucket name and select your region.
4. Click "Create bucket".

### 2. Setup SES

1. Go to the SES console.
2. Verify your email address or domain.
3. Note down your verified email address.

### 3. Create Lambda Function

1. Go to the Lambda console.
2. Click "Create function".
3. Choose "Author from scratch".
4. Enter the function name, e.g., `EmailMarketingFunction`.
5. Choose Python 3.8 as the runtime.
6. Create a new role with basic Lambda permissions.
7. Click "Create function".

**Lambda Function Code**:
```python
import json
import boto3
import os

ses = boto3.client('ses')
s3 = boto3.client('s3')

def lambda_handler(event, context):
    email = event['queryStringParameters']['email']
    bucket_name = os.environ['BUCKET_NAME'] # Replace with your bucket name!
    template_key = os.environ['TEMPLATE_KEY'] # Replace with your contacts file!
    
    # Get the email template from S3
    template_obj = s3.get_object(Bucket=bucket_name, Key=template_key)
    template_body = template_obj['Body'].read().decode('utf-8')
    
    response = ses.send_email(
        Source=os.environ['SOURCE_EMAIL'],
        Destination={
            'ToAddresses': [email],
        },
        Message={
            'Subject': {
                'Data': 'Welcome to Our Marketing Campaign',
                'Charset': 'UTF-8'
            },
            'Body': {
                'Html': {
                    'Data': template_body,
                    'Charset': 'UTF-8'
                }
            }
        }
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps('Email sent successfully!')
    }
```
8. Deploy the Lambda function

### 4. Setup API Gateway
1. Go to the API Gateway console.
2. Click "Create API".
3. Choose "HTTP API".
4. Click "Build".
5. Enter a name for the API.
6. Click "Next".
7. Under "Integrations", select "Lambda Function".
8. Select your Lambda function.
9. Click "Next" and then "Create".

### 5. Setup EventBridge
1. Go to the EventBridge console.
2. Click "Create rule".
3. Enter a name for the rule.
4. Under "Event source", select "AWS services".
5. Choose "API Gateway" and the API you created.
6. Add a target and select your Lambda function.
7. Click "Create".

### 6. Setup Route 53
1. Go to the Route 53 console.
2. Click "Create hosted zone".
3. Enter your domain name.
4. Note down the name servers.
5. Update your domain registrar with the new name servers.

### 7. Create HTML Form
1. Create an index.html file with the following content:
```
<!DOCTYPE html>
<html>
<head>
    <title>Email Marketing Signup</title>
</head>
<body>
    <form action="https://your-api-id.execute-api.your-region.amazonaws.com/default/your-lambda-function-name" method="get">
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```
2. Upload the index.html file to your S3 bucket.
3. Configure the bucket for static website hosting.

### Adding Contacts
```
Firstname,Email
John,example1@example.com
Emily,example2@example.com
James,example3@example.com
```
### Email Template
```
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to Our Marketing Campaign</title>
</head>
<body>
    <h1>Welcome!</h1>
    <p>Thank you for signing up for our marketing campaign. Stay tuned for exciting updates and offers!</p>
</body>
</html>
```
Upload this email_template.html file to your S3 bucket.

### How It Works
1. The user submits their email via the HTML form.
2. The form makes a GET request to API Gateway.
3. API Gateway triggers the Lambda function.
4. The Lambda function sends a marketing email using SES.
5. EventBridge monitors and logs the events for troubleshooting.