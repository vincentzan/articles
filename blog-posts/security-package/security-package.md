---
published: true
title: 'New sls-mentor Security Package'
description: 'Enhancing the Security of AWS Serverless Applications with sls-mentor'
tags: security, AWS, serverless
series: sls-mentor
canonical_url: https://dev.to/vincentzan/new-sls-mentor-security-package
---

In the realm of cloud computing, serverless has emerged as a popular paradigm, offering a pay-as-you-go model and the ability to scale applications dynamically. However, with such flexibility comes the potential for security vulnerabilities. To address this need, sls-mentor, an open-source tool, has been developed to audit and improve the security of your AWS Serverless applications.

**New Security Rules for Enhanced Protection**

We are excited to announce the release of new security rules for sls-mentor to further enhance the security of your serverless applications. These rules cover IAM role policies, SQS queues, and RDS instances.

**IAM Role Policies:**

- **No IAM Role Policy with wildcard Resource:** Using wildcards in IAM role policies can grant excessive access to resources, making it easier for attackers to exploit. To mitigate this risk, we recommend using specific resources in IAM policies instead of wildcards.

**SQS Queues:**

- **Encrypt your SQS queues:** SQS queues store messages, making them potential targets for unauthorized access. To safeguard your message data, encrypt your SQS queues. SQS supports server-side encryption, ensuring that the data is encrypted on the server before it is stored in the queue.

**RDS Instances:**

- **Encrypt your RDS instances:** RDS databases manage sensitive data, and encrypting them is crucial for protecting this information. sls-mentor checks for unencrypted RDS instances and provides instructions on how to enable encryption.

**Using sls-mentor to Enhance Security**

In addition to the newly introduced security rules, sls-mentor already includes a comprehensive set of security checks that cover various aspects of AWS Serverless applications. These include checks for insecure IAM role permissions, vulnerable Lambda function configurations, and unencrypted CloudWatch logs. By leveraging these existing checks and the newly added ones, developers can thoroughly assess and strengthen the security posture of their serverless applications.

To utilize sls-mentor and benefit from its security enhancement capabilities, follow these simple steps:

1. **Install sls-mentor:** Open your terminal and run the following command to install sls-mentor:

```bash
  npm install sls-mentor
```

2. **Run sls-mentor:** With your AWS credentials configured, run the following command to initiate the security analysis:

```bash
  npx sls-mentor@latest --report
```

3. **Review the security report:** The sls-mentor tool will generate a comprehensive report in your current directory, named `.sls-mentor/index.html`. This report provides insights into the security posture of your serverless applications, highlighting areas for improvement and offering actionable recommendations. **Join the Community and Contribute** We are always striving to enhance sls-mentor’s capabilities and welcome contributions from the community. If you have serverless or serverful knowledge to share, feel free to join our GitHub repository and contribute to our ongoing development efforts. Your contributions will directly impact the security of serverless applications worldwide.

**Conclusion** By incorporating sls-mentor into your serverless development workflow, you can proactively identify and address potential security vulnerabilities, ensuring the safety and integrity of your applications in the cloud. Join us in strengthening the security posture of serverless deployments and make your cloud infrastructure more resilient to attacks.
