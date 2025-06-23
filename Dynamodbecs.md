You're absolutely right — **as of late 2023**, **Amazon DynamoDB now supports resource-based policies** (also known as **table-level policies**) for fine-grained access control.

This is a **major enhancement** in how we can manage permissions on DynamoDB tables, especially useful when you want to restrict access based on specific IAM principals like roles, users, or services such as ECS tasks.

---

## ✅ Yes, You Can Use Resource-Based Policies on DynamoDB Tables

AWS introduced **resource-based policies for DynamoDB tables**, allowing you to define who (IAM principal) can perform what actions on a specific table. This works similarly to S3 bucket policies and allows for centralized permission management at the resource level.

---

## 🎯 Goal: Allow Only One ECS Service Access to a DynamoDB Table Using a Resource Policy

Here's how to do it:

---

### 🔧 Step-by-Step Setup

#### 1. Identify the ECS Task Role ARN
Your ECS task must run under an IAM role. You'll reference this role's ARN in your DynamoDB resource policy.

Example:
```
arn:aws:iam::123456789012:role/MyEcsTaskExecutionRole
```

#### 2. Create the DynamoDB Resource Policy

Go to the **DynamoDB Console > Tables > [your-table] > Permissions**, and add a **Resource Policy** like this:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/MyEcsTaskExecutionRole"
      },
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:Scan",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:region:123456789012:table/MyTable"
    }
  ]
}
```

> ⚠️ Replace `123456789012`, `MyEcsTaskExecutionRole`, `MyTable`, and `region` with your actual AWS account ID, role name, table name, and region.

#### 3. Remove Other IAM Policies That Allow Access

To ensure that **only** this service has access:

- Do not attach DynamoDB permissions directly to other IAM roles.
- Remove any IAM managed or inline policies that allow broad access to this table.
- Optionally use SCPs (Service Control Policies) in AWS Organizations if needed.

---

## 📌 Important Notes

| Feature | Status |
|--------|--------|
| Resource-based policies supported on DynamoDB? | ✅ Yes (since late 2023) |
| Supported Actions | All standard DynamoDB data plane operations |
| Scope | Per table only (not global tables or indexes yet) |
| Conflicts with IAM policies? | No – **both are evaluated** using least privilege logic |
| Best Practice | Combine with IAM role scoped to ECS task |

---

## 🧠 When to Use Resource-Based Policies vs IAM Policies

| Use Case | Recommended Method |
|---------|---------------------|
| Centralized control over access to a single DynamoDB table | ✅ Resource-based policy |
| Fine-grained access per user/role/service | ✅ IAM policy |
| Multiple services need different access levels to same table | ✅ Both IAM + Resource policies |

---

## 🔍 Example Use Case

You have two ECS services (`service-a`, `service-b`) and one DynamoDB table (`users-table`). You want:

- `service-a` to have full access.
- `service-b` to have read-only access.

✅ You can achieve this cleanly with a **single resource-based policy** on the DynamoDB table:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/ServiceARole"
      },
      "Action": "dynamodb:*",
      "Resource": "arn:aws:dynamodb:region:123456789012:table/users-table"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/ServiceBRole"
      },
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:Scan",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:region:123456789012:table/users-table"
    }
  ]
}
```

---

## ✅ Summary

- ✅ DynamoDB now supports **resource-based policies**.
- ✅ You can allow only one ECS service to access a DynamoDB table using a **table-level policy**.
- ✅ Ensure your ECS task runs with a dedicated IAM role.
- ✅ Combine with IAM best practices for secure, manageable access.

Let me know if you'd like help writing or validating your exact policy!
