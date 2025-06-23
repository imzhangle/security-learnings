Yes, **using a Key Policy is the primary and recommended way to restrict access to AWS KMS (Key Management Service)**. However, there are also other methods you can use in conjunction with key policies to further secure your KMS keys.

---

## ✅ **1. Use Key Policies (Primary Method)**

A **KMS Key Policy** controls:
- Who (IAM users, roles, accounts) can use the key.
- What actions they can perform (e.g., `kms:Encrypt`, `kms:Decrypt`).
- Under what conditions (e.g., IP address, encryption context).

### 🔐 Example Key Policy Snippet:

```json
{
  "Version": "2012-10-17",
  "Id": "key-default-1",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow specific role to decrypt",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/MyRole"
      },
      "Action": [
        "kms:Decrypt",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    }
  ]
}
```

> ⚠️ Note: The **root principal (`arn:aws:iam::ACCOUNT_ID:root`)** is often included by default for account-level administration. You can replace or restrict this as needed.

---

## ✅ **2. Use IAM Policies**

In addition to key policies, you can use **IAM policies** attached to users, roles, or groups to control access.

However, IAM policies **only work if the key policy allows access to the principal first**.

### 📌 Example IAM Policy for KMS:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt"
      ],
      "Resource": "arn:aws:kms:region:123456789012:key/abcd123a-a123-45cd-efgh-678ijklmno90"
    }
  ]
}
```

---

## ✅ **3. Use Resource-Based Policies (Not Available for KMS)**

Unlike S3 or Lambda, **KMS does not support resource-based policies** like bucket policies or function policies. Instead, it uses **Key Policies** as the main mechanism.

---

## ✅ **4. Use Conditions in Key Policies**

You can add **conditions** to key policies to further restrict access:

- Based on **IP address**
- Based on **encryption context**
- Based on **AWS service** using the key
- Require **SSL/TLS**

### 🛡 Example Condition:

```json
"Condition": {
  "IpAddress": {
    "aws:SourceIp": "203.0.113.0/24"
  }
}
```

---

## ✅ **5. Use Grant-based Access (Programmatic Only)**

You can use **grants** to allow temporary or programmatic access to a key without modifying the key policy.

- Useful for cross-account access or short-term permissions.
- Can be created via the KMS API (`CreateGrant`).

---

## ✅ **6. Cross-Account Access**

To allow access from another AWS account:
- Create a key policy that allows access from the external account.
- Then, in the external account, create an IAM policy that references the key ARN.

---

## ✅ Best Practices to Restrict Access to KMS Keys

| Practice | Description |
|---------|-------------|
| Principle of Least Privilege | Only allow required actions (`kms:Decrypt`, `kms:Encrypt`, etc.) |
| Use Key Policies | Always start here – they are mandatory |
| Combine with IAM Policies | For fine-grained control within the account |
| Enable CloudTrail logging | Monitor all KMS usage for audit and security |
| Rotate keys | Enable automatic key rotation where possible |
| Deny access by default | Explicitly deny access unless explicitly allowed |
| Use VPC Endpoint (KMS VPC endpoint) | Prevent traffic from going over public internet |

---

## ✅ Summary

| Control Mechanism | Can Be Used? | Notes |
|------------------|--------------|-------|
| Key Policy        | ✅ Yes       | Primary method |
| IAM Policy        | ✅ Yes       | Must align with key policy |
| Resource Policy   | ❌ No        | Not supported |
| Grants            | ✅ Yes       | Programmatic/temporary access |
| VPC Endpoint      | ✅ Yes       | Network isolation |
| SCPs              | ❌ No direct | Can block KMS creation but not key access |

---

Would you like an example policy for a specific use case (like restricting Lambda access to KMS, or allowing cross-account access)?
