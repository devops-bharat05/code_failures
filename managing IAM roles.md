## Problem Statement: I am managing IAM roles and have given 10 students full access to IAM. I want them to be able to create any role and delete roles if required. However, I want to restrict them from deleting IAM roles created by other students. How can I achieve this?

To achieve this, you can manage the permissions for your students using **IAM policies** and **resource-based policies**. Here’s how you can restrict them from deleting IAM roles created by other students while still allowing them full access to create and delete roles for themselves:

### Steps to implement:

1. **Create a custom IAM policy** that grants the students permissions to manage IAM roles, but with a condition to prevent deletion of roles created by other students. You can use the `aws:RequestTag` and `aws:PrincipalTag` conditions to ensure that students can only delete roles they created.

2. **Tagging roles**: You need to ensure that when students create roles, they add a tag (e.g., `createdBy: student-name`) to identify who created the role. This way, the student can only delete roles that have their own user ID (or a tag with their user name) as the `createdBy` tag.

3. **IAM Policy Example**:

   Create an IAM policy that allows role creation and deletion, but restricts deletion of roles that don't have the student's tag (i.e., roles created by others). Here's an example policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:AttachRolePolicy",
                "iam:PutRolePolicy",
                "iam:DeleteRole",
                "iam:PassRole"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Deny",
            "Action": "iam:DeleteRole",
            "Resource": "*",
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:RequestTag/createdBy": "${aws:PrincipalTag/Name}"
                }
            }
        }
    ]
}
```

### Explanation:
- The first `Allow` statement gives students full access to create roles, attach policies, delete roles, and pass roles.
- The second `Deny` statement ensures that students can **only delete roles** that have a `createdBy` tag that matches their IAM user name (`${aws:PrincipalTag/Name}`). If the role was created by another student, they won't be able to delete it.
- The `StringNotEqualsIfExists` condition means that if the `createdBy` tag exists, the student's `Name` tag must match for them to be able to delete the role.

### Implementing the `createdBy` tag:

1. **Create roles with the `createdBy` tag**: When a student creates a role, they should ensure that they add their IAM username as a tag (e.g., `createdBy: student-name`). You can enforce this by using a `Condition` in your IAM policies or through automation tools like Lambda to add this tag during role creation.

2. **Use `aws:PrincipalTag/Name`**: This ensures that only the student with the same `Name` tag can delete the role they created.

### Example Scenario:
- Student A creates a role and tags it with `createdBy: StudentA`.
- Student B creates a role and tags it with `createdBy: StudentB`.
- Student A will be able to delete the role tagged with `createdBy: StudentA`, but **cannot delete** the role created by Student B (`createdBy: StudentB`).

By following this approach, you ensure that students can create and manage their roles, but they won’t be able to delete roles created by others.
