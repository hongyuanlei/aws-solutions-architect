## S3 Access

#### S3 Access Status

- Public

Everyone has access to one or more of the following: List Objects, Write Objects, Read and write permissions.

- Object can be public

The bucket is not public but anyone with appropriate permissions can grant public access to objects.

- Bucket and objects not public

The bucket and objects do not have any public access.

- Only authorized users of this account

Access is isolated to **IAM users** and **roles in this account** and **AWS service principals** because there is a policy that grants public access.

#### Public access setting

Amazon S3 provides Block Public Access settings for buckets and accounts to help you manage public access to Amazon S3 resources.

By default, new buckets and objects don't allow public access, but users can modify bucket policies or object permissions to allow public access.

Amazon S3 Block Public Access provides settings that override these policies and permissions so that you can limit public access to these resources.

With Amazon S3 Block Public Access, account administrators and bucket owners can easily set up centralized controls to limit public access to their Amazon S3 resources that are enforced regardless of how the resources are created.

#### Access control list

Access control lists (ACLs) are one of the resource-based access policy options that you can use to manage access to your buckets and objects.

You can use ACLs to grant basic read/write permissions to other AWS accounts. There are limits to managing permissions using ACLs.

For example, you can grant permissions only to other AWS accounts; you cannot grant permissions to users in your account. You cannot grant conditional permissions, nor can you explicitly deny permissions. ACLs are suitable for specific scenarios.

For example, if a bucket owner allows other AWS accounts to upload objects, permissions to these objects can only be managed using object ACL by the AWS account that owns the object.


#### Bucket policy


In its most basic sense, a policy contains the following elements:

- Resources

Buckets and objects are the Amazon S3 resources for which you can allow or deny permissions. In a policy, you use the Amazon Resource Name (ARN) to identify the resource.

- Actions

For each resource, Amazon S3 supports a set of operations. You identify resource operations that you will allow (or deny) by using action keywords.

- Effect

What the effect will be when the user requests the specific action—this can be either allow or deny.

If you do not explicitly grant access to (allow) a resource, access is implicitly denied. You can also explicitly deny access to a resource, which you might do in order to make sure that a user cannot access it, even if a different policy grants access.

- Principal

The account or user who is allowed access to the actions and resources in the statement. In a bucket policy, the principal is the user, account, service, or other entity who is the recipient of this permission.

The following example bucket policy shows the preceding common policy elements. The policy allows Dave, a user in account Account-ID, s3:GetObject, s3:GetBucketLocation, and s3:ListBucket Amazon S3 permissions on the examplebucket bucket.

```
{
    "Version": "2012-10-17",
    "Id": "ExamplePolicy01",
    "Statement": [
        {
            "Sid": "ExampleStatement01",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::Account-ID:user/Dave"
            },
            "Action": [
                "s3:GetObject",
                "s3:GetBucketLocation",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::examplebucket/*",
                "arn:aws:s3:::examplebucket"
            ]
        }
    ]
}
```

#### CORS configuration