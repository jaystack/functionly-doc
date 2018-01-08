## Serverless limitations

Currently we are using [serverless](https://www.npmjs.com/package/serverless) for our [AWS Lambda](https://aws.amazon.com/lambda) projects. For now **we are fairly blocked** by some limitations:

- **[Cloudformation templates are limited to 200 resources](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)**. Issues are [here](https://github.com/serverless/serverless/issues/2387) and [here](https://github.com/serverless/serverless/issues/3411), but they hasn't be fixed for a while. Despite we are using a [plugin](https://github.com/dougmoscrop/serverless-plugin-split-stacks), **we can't add any feature as lambda anymore**.
- **[Inline role policies are limited to 10,240 chars](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_iam-limits.html)**. This is a [known issue](https://github.com/serverless/serverless/issues/2508), and we are still struggling with this. The only solution is to open roles generally, which **causes security issues**.
- **[Disk space and lambda deployment packages are limited](http://docs.aws.amazon.com/lambda/latest/dg/limits.html)**. Our package sizes are **very close to the limit**, and we have to perform the garbage clearing on disk via jenkins.

## Solution: [functionly](https://www.npmjs.com/package/functionly)

We've been working on a solution for a while, especially after realizing these limitations. [Functionly](https://www.npmjs.com/package/functionly) brings the functional approach even to NodeJS hosted and serverless conventional microservices, and it solves the limitations above. You can read more [here](https://github.com/jaystack/functionly-doc/blob/master/wut.md).
