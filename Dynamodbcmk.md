To set up **Customer Master Keys (CMKs)** for use with **Amazon DynamoDB**, you're likely referring to using **AWS Key Management Service (KMS)** CMKs to encrypt your DynamoDB tables. This is part of the encryption-at-rest feature provided by AWS.

Here's a step-by-step guide to setting up and using a Customer Master Key (CMK) from AWS KMS to encrypt your DynamoDB table:

---

## 🔐 Step 1: Create a CMK in AWS KMS

1. **Open the AWS KMS Console**:
   - Go to [https://console.aws.amazon.com/kms](https://console.aws.amazon.com/kms)

2. **Create a new CMK**:
   - Choose **"Create key"**
   - Select **"Symmetric** (default and recommended for DynamoDB)
   - Add **Key administrators**: IAM users or roles that can manage the key
   - Add **Key users**: IAM users, roles, or services (like DynamoDB) that are allowed to use the key
   - Set key policies as needed (advanced)
   - Review and create

> Make note of the **ARN** of the CMK — you'll need it when creating or updating your DynamoDB table.

---

## 🗂️ Step 2: Enable CMK Encryption on Your DynamoDB Table

### Option A: When Creating a New Table

Use the AWS Console, CLI, or SDK to specify the CMK during creation.

#### ✅ Using AWS Console:
1. Open the [DynamoDB Console](https://console.aws.amazon.com/dynamodb)
2. Click **"Create table"**
3. Fill in the table name and primary key
4. Under **"Encryption at rest"**, select **"AWS KMS using your own CMK"**
5. Paste the ARN of your CMK
6. Complete the setup

#### ✅ Using AWS CLI:
```bash
aws dynamodb create-table \
    --table-name MyTable \
    --attribute-definitions AttributeName=id,AttributeType=S \
    --key-schema AttributeName=id,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --sse-specification Enabled=true,SSEType=KMS,KMSMasterKeyId=<your-cmk-arn>
```

---

### Option B: Enable CMK Encryption on an Existing Table

You cannot change the encryption settings of an existing DynamoDB table directly. However, you can:

1. **Export the data** (e.g., via AWS Backup, DAX, or scanning/exporting)
2. **Delete the old table**
3. **Recreate the table** with the same structure and enable CMK-based encryption
4. **Import the data**

Alternatively, if you initially created the table without specifying a CMK, but used **AWS-owned or AWS-managed KMS keys**, you can still enforce encryption using **SSE-S3** or switch to **AWS KMS CMK** by re-creating the table.

---

## 🔒 Step 3: IAM Permissions

Ensure that the IAM role or user interacting with DynamoDB has permissions to use the CMK.

Add the following permission in your IAM policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt",
                "kms:GenerateDataKey"
            ],
            "Resource": "arn:aws:kms:<region>:<account-id>:key/<key-id>"
        }
    ]
}
```

Also ensure that the **DynamoDB service-linked role** has access to the CMK.

---

## 🧪 Step 4: Verify Encryption Settings

After creating or updating the table, verify the encryption status:

### Using AWS CLI:
```bash
aws dynamodb describe-table \
    --table-name MyTable \
    --query 'Table.SSEDescription'
```

The output should show:
```json
{
    "Status": "ENABLED",
    "SSEType": "KMS",
    "KMSMasterKeyArn": "arn:aws:kms:..."
}
```

---

## ✅ Summary

| Step | Description |
|------|-------------|
| 1 | Create a CMK in AWS KMS |
| 2 | Use the CMK ARN when creating a DynamoDB table |
| 3 | Ensure proper IAM/KMS permissions |
| 4 | Verify encryption settings |

---

Let me know if you'd like this done via CloudFormation, Terraform, or SDK examples (Python/Boto3, etc.).
