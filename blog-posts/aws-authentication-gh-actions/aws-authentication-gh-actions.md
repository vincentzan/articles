---
published: false
title: 'Configure Authentication to your AWS account in your GitHub actions CI'
cover_image: '../assets/background_image.png'
description: 'Easily authenticate your AWS account for your CI/CD pipeline! '
tags: CI, CD, Github actions, AWS
canonical_url: https://dev.to/vincentzan/configure-aws-authentication-on-github-actions
---

**TL;DR** Streamline your GitHub Actions CI workflow by addressing the challenge of configuring secure authentication to your AWS account. This article provides concise guidance on ensuring seamless integration while maintaining the highest standards of security.

I recently packaged my awesome, open-source, AWS stack auditing tool [SLS-mentor](https://www.sls-mentor.dev/) into a [github action](https://github.com/marketplace/actions/sls-mentor). On this occasion, I was confronted with the question of reproducibly configuring AWS authentication on a remote CI/CD job. In this article, I will explain to you how to proceed.

We are going to configure authentication to an AWS account in a GitHub Actions CI workflow using OIDC-standard short-term credentials authentication. This is particularly useful when you want to delegate the deployment of your AWS infrastucture as code to Github Actions. Compared to long lived credentials like AWS IAM user access keys, utilizing short-lived IAM credentials like OIDC offers a more secure alternative by reducing the exposure window and enhancing the overall resilience of the authentication process.

I will start by giving a short principle description of the authentication protocol. I you are just looking for a practical tutorial, you can directly jump to the next section.

## Dive into GH OIDC protocol

Now maybe you are wondering about what is happening under the hood when you set up this authentication method.

The OIDC protocol that we configure is a standard based upon the OAuth 2.0 protocol that provides secure identity certification.

![schema]('../../assets/schema.png' 'principle schema')

You need to create a trust relationship between GH OIDC provider (also called IdP for identity provider) and AWS IAM. Note that for Github actions it is a third party distinct from Github (called Digicert). This is what is done when you clicked “get thumbprint” on the setup. In response, the authorisation server responds with a public key (for the RS256 protocol). With the latter, the cloud provider can check (and only check) the validity of a given JWT token. When you create a trust relationship between IAM and an OIDC identity provider, you are trusting identities authenticated by that IdP (identified by their webiste url, or DNS) to have access to a certain scope AWS account. Each CI job (i.e. each instance of a Github virtual machine CI/CD runner) requests an OIDC token from GitHub's Identity provider, which responds with a unique, automatically generated JSON web token (JWT). When the job runs, the OIDC token is presented to the cloud provider (in our case, to the STS service, from which we ask short term credentials in order to assume a role as the github runner). To validate the token, the cloud provider first checks it authenticity and second it checks if the OIDC token's subject and other claims are a match for the conditions that were preconfigured on the cloud role's OIDC trust definition. For example, the token will check the audience and the subjects as we have set in the Trust Policy.

To fully authenticate your GitHub action runner with your AWS account, we are going to follow 3 main steps :

1. Configure an OIDC trust relationship between AWS IAM and Github Actions
2. Create a minimal IAM role for the CI workflow to assume.
3. Configure the authentication in your CI workflow.

## Configure an OIDC trust relationship between AWS IAM and Github Actions

First you want to tell AWS IAM to acknowledge the Github identity provider as a secure Identity Certificating Authority, that is an Identity Provide (IdP).

- Go to the AWS IAM console and under identity providers click “Add provider” under provider type specify OIDC (last generation, JSON-based while SAML is XML based)
- For provider URL, enter the URL of the GitHub OIDC IdP for this solution: [https://token.actions.githubusercontent.com](https://token.actions.githubusercontent.com/)
- Click on “Get thumbprint”
- Audience, enter [sts.amazonaws.com](http://sts.amazonaws.com/): ![tuto1]('../assets/tuto1.png' 'congigure Github identity provider')

## Create a minimal IAM role for the CI workflow to assume.

- Click on the newly created identity provider
- Click on “Assign role”
- Create new role
- In the Audience list, select [sts.amazonaws.com](http://sts.amazonaws.com/)
- Fill in the Github organization (note that AWS has inferred that we are configuring GitHub there).

On the next page you need to select the minimal role you need your CI/CD to assume in order to The least privilege principle in AWS IAM emphasizes assigning the minimal permissions necessary for a specific task to prevent over-authorization. This practice enhances security by reducing the attack surface and minimizing the potential impact of compromised credentials.

- Knowing this, select the role you want your CI/CD to be able to assume
- Check the trust policy of the role : it should read as follows :

![tuto2]('../assets/tuto2.png' 'configure trust policy')

```yaml
'Condition':
  {
    'StringEquals':
      {
        'token.actions.githubusercontent.com:aud': 'sts.amazonaws.com',
        'token.actions.githubusercontent.com:sub': 'repo:octo-org/octo-repo:ref:refs/heads/octo-branch',
      },
  }
```

## Configure the authentication in your CI workflow.

Now going back to the Github actions jobs in your CI/CD workflow. How do make it authenticate using the process we have just set up?

To do so, we are going to leverage the power of the Github actions marketplace. Indeed, AWS provides us with the code to authenticate the runner machine with the AWS client wrapped in a Github action simply available from the Github actions marketplace. You just need to add this action as a step to your workflow. Don’t forget to carefully replace with the role with the arn of the role you just created.

```yaml
- name: Configure AWS Credentials
	uses: aws-actions/configure-aws-credentials@v3
	with:
		role-to-assume: arn:aws:iam::111122223333:role/GitHubAction-AssumeRoleWithAction #change to reflect your IAM role’s ARN
		role-session-name: GitHub_to_AWS_via_FederatedOIDC
```

Finally, add the following permissions to your job

```yaml
permissions:
  id-token: write # This is required for requesting the JWT
  contents: read # This is required for actions/checkout
```

That’s it, you are all set!

## Sources:

https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services

https://github.com/aws-actions/configure-aws-credentials#configure-aws-credentials-for-github-actions

https://datatracker.ietf.org/doc/html/rfc6749.html#section-1.4

https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc_verify-thumbprint.html

https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html

https://auth0.com/docs/get-started/applications/signing-algorithms
