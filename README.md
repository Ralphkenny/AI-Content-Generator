# Telex Groq Content Generator Integration

## Overview
This guide explains how to integrate **Groq AI** with **Telex** using an **AWS Lambda** function to generate content based on keywords provided by users. The Lambda function will communicate with the **Groq API** to generate responses based on user input (keywords), and the results will be sent back to Telex.

### Key Features:
- Accepts user-provided keywords from Telex.
- Sends the keyword to Groq Cloud to generate content.
- Returns the generated content back to Telex.

---

## Prerequisites

Before starting the integration, make sure you have:

1. **AWS Account**: To create and deploy Lambda functions.
2. **Telex Account**: To configure and test the integration in the Telex platform.
3. **Groq API Key**: For making authenticated requests to the Groq API.

---

## Setup Steps

### 1. Create the Lambda Function

1. **Log in to AWS Lambda**:
    - Navigate to [AWS Lambda Console](https://console.aws.amazon.com/lambda/).

2. **Create a New Lambda Function**:
    - Click **Create Function**.
    - Choose **Author from Scratch**.
    - Set a name for your function (e.g., `TelexGroqIntegration`).
    - Choose **Python 3.x** as the runtime.
    - Assign a basic execution role with necessary permissions (e.g., `AWSLambdaBasicExecutionRole`).

3. **Lambda Function Code**:
    Replace the default function code with the following:

    ```python
    import json
    import requests

    def lambda_handler(event, context):
        # Step 1: Extract the keyword from the Telex event (JSON)
        keyword = event.get('keyword')
        
        if not keyword:
            return {
                'statusCode': 400,
                'body': json.dumps('Error: No keyword provided')
            }

        # Step 2: Set up the Groq Cloud API details
        GROQ_API_URL = "https://api.groq.com/openai/v1/chat/completions"
        GROQ_API_KEY = "YOUR_GROQ_API_KEY"  # Replace with your actual Groq API key
        
        headers = {
            "Authorization": f"Bearer {GROQ_API_KEY}",
            "Content-Type": "application/json"
        }
        
        # Step 3: Prepare the data for Groq API
        data = {
            "model": "llama-3.3-70b-versatile",
            "messages": [{
                "role": "user",
                "content": keyword  # Send the user's keyword as the content
            }]
        }

        # Step 4: Send the request to Groq Cloud to generate content
        try:
            response = requests.post(GROQ_API_URL, json=data, headers=headers)
            
            # Log the response for debugging
            print(f"Response Status Code: {response.status_code}")
            print(f"Response Content: {response.text}")

            if response.status_code == 200:
                content = response.json().get("choices", [{}])[0].get("message", {}).get("content", "No content returned.")
                return {
                    'statusCode': 200,
                    'body': json.dumps({'content': content})
                }
            else:
                return {
                    'statusCode': response.status_code,
                    'body': json.dumps({'error': 'Failed to generate content from Groq', 'details': response.text})
                }
        except Exception as e:
            return {
                'statusCode': 500,
                'body': json.dumps({'error': str(e)})
            }
    ```

4. **Save and Deploy**:
    - After adding the code, click **Deploy** to save and deploy the Lambda function.

---

### 2. Create Lambda Function URL (for Telex Integration)

1. **Enable Lambda Function URL**:
    - Go to the **Function URL** section in the Lambda function dashboard.
    - Click **Create Function URL** to generate a URL endpoint for your Lambda function.

2. **Copy the URL**:
    - Copy the generated **Lambda Function URL**, which will be used in your Telex integration.

---

### 3. Update Telex Integration Configuration

1. **Telex Integration JSON**:
    The configuration JSON file for Telex will include the **Lambda Function URL**. The JSON structure will look like this:

    ```json
    {
        "integration_name": "Groq Content Generator",
        "app_url": "https://<lambda-function-url>.lambda-url.us-east-1.on.aws/",
        "logo_url": "https://my-application.com/logo.png",
        "integration_type": "External API Integration",
        "created_at": "02/21/2025",
        "updated_at": "02/21/2025",
        "target_url": "https://<lambda-function-url>.lambda-url.us-east-1.on.aws/",
        "author": "Layobright",
        "integration_category": "AI Content Generation",
        "tick_url": "https://my-support-page.com",
        "description": "Generates AI-powered content based on user-provided keywords, using Groq Cloud.",
        "key_features": "Generates content based on user keywords, customizable response length, AI-powered.",
        "settings": [
            {
                "field_label": "Keyword Length",
                "field_type": "Number",
                "default_value": "200"
            },
            {
                "field_label": "Time Interval",
                "field_type": "Text",
                "default_value": "10"
            }
        ]
    }
    ```

2. **Submit the JSON to Telex**:
    - Submit the JSON file to Telex as part of the integration setup.
    - Telex will use the **Lambda Function URL** to send user input (keywords) to your Lambda function.

---

### 4. Test the Integration

1. **Test the Lambda Function**:
    - Create a test event in AWS Lambda with the following payload:
    ```json
    {
        "keyword": "Explain the importance of fast language models"
    }
    ```

    - Click **Test** in the AWS Lambda console to verify that the function generates the content correctly and returns the expected output.

2. **Test in Telex**:
    - After the Lambda function is working, type a keyword in the **Telex channel**.
    - The request will be sent to the Lambda function, which will interact with Groq Cloud and return the generated content to Telex.

---

### 5. Debugging and Logs

1. **Check CloudWatch Logs**:
    - If the integration is not working as expected, check the **CloudWatch logs** to see detailed logs of the Lambda execution.
    - You can find logs related to **errors**, **responses**, and other useful debug information in CloudWatch.

2. **Groq Response Logs**:
    - You can also log the full **response content** from Groq Cloud to see what’s being returned. This will help you understand if there's an issue with the request or the response format.

---

## Conclusion

This integration setup allows you to use **Groq Cloud**’s powerful AI model to generate content dynamically based on keywords sent from Telex. By using **AWS Lambda** and **Telex’s API integration**, you can provide your users with engaging, AI-generated content in real-time.

---

## Troubleshooting

If you encounter any issues, check the following:
- **Lambda function permissions**: Ensure your Lambda function has the necessary permissions to make external HTTP requests.
- **API Key**: Double-check the Groq API key for validity and ensure it has the correct access levels.
- **Response Format**: Make sure the Groq API response format matches the expected structure for the Lambda function.

Let me know if you need any further assistance!
