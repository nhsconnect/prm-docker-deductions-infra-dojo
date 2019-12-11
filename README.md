# docker-deductions-infra-dojo

[Dojo](https://github.com/kudulab/dojo) docker image with tools needed to deploy infrastructure in scope of deductions team.

Tested and released images are published to dockerhub as [nhsdev/deductions-infra-dojo](https://hub.docker.com/r/nhsdev/deductions-infra-dojo)

## Usage
1. Setup docker.
2. Install [Dojo](https://github.com/kudulab/dojo) binary.
3. Provide a Dojofile at the root of the project:
```
DOJO_DOCKER_IMAGE="nhsdev/deductions-infra-dojo:<commit>"
```
4. Create and enter the container by running `dojo` at the root of project.

By default, current directory in docker container is `/dojo/work`.

### Access to AWS

In order to get sufficient access to work with terraform or AWS CLI:

Export your AWS credentials in shell (if you have credentials in `~/.aws/credentials` that will work too):
```
export AWS_ACCESS_KEY_ID=***********
export AWS_SECRET_ACCESS_KEY=**************************
unset AWS_SESSION_TOKEN
```

Enter docker container with terraform and AWS CLI by typing:
```
dojo
```
at the root of the project.

Assume role with elevated permissions:
```
eval $(aws-cli-assumerole -rmfa <role-arn> <your-username> <mfa-otp-code>)
```

Work with terraform as usual:
```
terraform init
terraform apply
```

If your session expires, exit the container to drop the temporary credentials and run `dojo` again.


# Specification

 * base image is alpine, to make this image as small as possible
 * terraform binary on the PATH
 * terraform plugins: aws, null, external, local, template.
 * packer to build MHS images
 * AWS CLI
 * Docker CLI 
 * `jq` to parse JSON from bash scripts
 * `dot` to generate infrastructure graphs from terraform
 * a minimal ssh and git setup - to clone terraform modules
