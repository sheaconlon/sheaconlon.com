# sheaconlon.com

## Deployment

To deploy the website from scratch, follow the [deployment instructions](deployment.md). The instructions walk you through the following:

1. setting up an Ubuntu virtual machine using AWS VPC and EC2
2. setting up DNS for your domain using AWS Route 53
3. setting up a site down notification using AWS SNS and Cloudwatch
4. setting up Apache for separate production and development sites, www to non-www redirection, "clean" URLs, and no server-side scripting

The requirements are a domain name, an Amazon Web Services account, and roughly $6 per month to spend on it. The result is a running instance of this website, or a different website, if you simply paste in different files at the end! The domain name is optional and you could use a t2.micro instead of a t2.nano EC2 instance to take advantage of free tier and pay near $0 per month.

## Workflow

To set up my preferred development workflow, follow the [workflow setup instructions](workflow_setup.md). The instructions walk you through the following:

1. getting Sublime Text 3 and the package Sublime SFTP
2. configuring Sublime SFTP and `sshd` (only on Mac OS) for remotely editing files on your EC2 instance
3. configuring `git` for separate production and development branches and `git`-based deployment