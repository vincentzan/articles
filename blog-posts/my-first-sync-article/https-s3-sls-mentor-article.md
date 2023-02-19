---
published: false
title: 'Ensure HTTPS-Secure Communication 🤝 🔑 with your S3 Bucket'
cover_image:
description: 'Easily configure your S3 buckets to allow only encrypted requests. Quickly check if your stack complies with sls-mentor! 📈'
tags: HTTPS, REST, S3, AWS
series: sls-mentor
canonical_url:
---
Serverless architectures are becoming increasingly popular for various reasons including performance, maintainability and cost. This shifts a lot of cybersecurity concerns to the cloud provider, but it surely does not mean you shouldn’t care about the security of your serverless app.

The app architecture is a key part of a secure serverless app—building separate, atomic Lambda functions, separating services etc. But it’s not enough, because the architecture doesn’t include protection methods or instrumentation agents such as [keys authentication or file transfer protocols](https://protectonce.com/the-10-best-practices-for-serverless-security/).

Hopefully, there is a simple action to take for improved security: setting up the right configurations for your AWS resources can help you increase the overall security of your app at low cost. [Sls-mentor](https://sls-mentor.dev/) can help you in that endeavour. It is an open source audit tool running directly from the terminal which will audit your resource stack. It will go through a list of good practice criteria and pinpoint you where there is room for improvement. 

More specifically, if you’re building a serverless app with AWS there’s a good chance you’re using S3 storage. As a data store, it is crucial to avoid security vulnerabilities with this resource.

First you should encrypt your S3 bucket on the server side so that the data is not readable if unfortunately data is maliciously retrieved from then. This can be done with simple, free S3-managed configuration (ServerSideEncryption-S3). From January 2023, this is done [by default](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-encryption.html) on all newly uploaded objects  on S3 buckets.

![meme data breach](./assets/meme-data-breach.gif 'meme data breach')

Now let’s say you run run an audit of your stack with sls-mentor and you realize that your S3 bucket is shamelessly managed through unencrypted requests. 😱 Don’t worry there is a quick fix! 🔧

This can be done simply by using the widespread HTTPS protocol 🤝 for posting requests to the database. Essentially, it uses transport layer encryption, a public-key cryptographic protocol—the encryption key is public but the decryption key is private. Hence, the request data is end-to-end encrypted. In turn, plain old HTTP requests are not encrypted thus are vulnerable to [man-in-the-middle](https://csrc.nist.gov/glossary/term/man_in_the_middle_attack) and eavesdropping attacks.

![meme https](./assets/meme-https.png 'meme https')

To enforce a configuration where only HTTPS requests will be allowed on your S3 bucket, you need to change its access policy. An access policy is made of a version, a name and a list of statements. Each element of the latter is an object that describes the different rules that apply to your bucket. Each rule is a set of property assignments: 
- effect sets if the statement will allow or deny access e.g. "allow";
- principal is the user or account that the statement applies to e.g. "awesome-user";
- action is the database actions (e.g. "s3:GetObject") in scope of the statement;
- the resource is the list of AmazonResourceNames (ARNs) of the objects. It is a long string that begins with "arn:aws:s3:::*" e.g. "arn:aws:s3:::my-stack-my-chicken-wings-bucket423e3a42f3-432f3j2";
- the condition is a [property](https://docs.aws.amazon.com/AmazonS3/latest/userguide/amazon-s3-policy-keys.html) that states when the policy applies : in our case we want to deny access when the aws:SecureTransport is false;
- you can tag your statement to group resources which share the same tag e.g. "food-bucket".

The thing above delimited by `---` is called a "front matter" and it allows us to keep control over our article in a very easy way. Just edit it with your own data and CI will handle the rest to publish it to dev.to!

You can also take advantage of [embedme](https://github.com/zakhenry/embedme) to extract your code from the markdown file and make sure that what you're displaying in the markdown is always up to date too e.g.

![Logo kumo](./assets/logo_kumo_carre.png 'Logo Kumo')

```ts
// code/demo-code.ts

interface A {
  hello: string;
}
```

# Found a typo?

If you've found a typo, a sentence that could be improved or anything else that should be updated on this blog post, you can access it through a git repository and make a pull request. Instead of posting a comment, please go directly to <REPO URL> and open a new pull request with your changes.
