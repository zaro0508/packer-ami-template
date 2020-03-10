# packer-ami-template
A template for quickly getting a new packer AWS AMI project started.

## Naming
**IMPORTANT**: Our naming convention is `packer-<image name>` (i.e. packer-base-ubuntu-bionic).
Please name your repo accordingly.  This naming convention helps us locate packer repos and
their corresponding builds in github and travis.

## Development

### Setup
* Install packer with [provided script](install_packer.sh). General install instructions are
in [packer docs](https://www.packer.io/intro/getting-started/install.html)
* Install [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

### Validate a template
Choose an ImageName such as "MyImage" and run
```
packer validate -var 'ImageName=my-test-image' template.json
```

### AWS Access
To run a build you must have an AWS account and access to EC2.

* Request an IAM account in [Imagecentral](https://github.com/Sage-Bionetworks/imagecentral-infra)
* Change password and set up MFA
* Create a Keypair
* Add your access code and secrety key to `~/.aws/credentials`, using a profile such as "imagecentral.jsmith"
* Authenticate with `awsmfa`, for example `awsmfa -i imagecentral.jsmith -t jsmith@imagecentral`
* Finally, get the correct role ARN for the PackerServiceRole then add the following:
```
[profile packer-service-imagecentral]
region = us-east-1
role_arn = *****
source_profile = jsmith@imagecentral
```

Now you will be able to build an image and deploy it to Imagecentral.

### Manual AMI Build
If you would like to test building an AMI run:
```
cd src
packer build -var AwsProfile=packer-service-imagecentral -var AwsRegion=us-east-1 -var ImageName=my-test-image src/template.json
```

Packer will do the following:
* Create a temporary EC2 instance, configure it with shell/ansible/puppet/etc. scripts.
* Create an AMI from the EC2
* Delete the EC2

__Note__: Packer deploys a new AMI to the AWS account specified by the AwsProfile

### Image Accessability
This project is setup to build publicly accessible images.  To change it to
build private images please refer to the [packer documentation](https://packer.io/docs/builders/amazon-ebs.html)
for `ami_users` and `snapshot_users`options.

### Testing
As a pre-deployment step we syntatically validate our packer json
files with [pre-commit](https://pre-commit.com).

Please install pre-commit, once installed the file validations will
automatically run on every commit.  Alternatively you can manually
execute the validations by running `pre-commit run --all-files`.

### CI Workflow
The workflow to provision AWS AMI is done using pull requests.
Just make changes with PRs and when th PR is merged a packer build
will kick off which will build the image and deploys it to AWS.

Packer will do the following:
* Create a temporary EC2 instance, configure it with shell/ansible/puppet/etc. scripts.
* Create an AMI from the EC2
* Delete the EC2

__Note__: The image will automatically be named gitrepo-branch (i.e. MyRepo-master)

### Versioning
Versions are managed by git tags. When the tag is pushed travis will build
an AMI version with that tag number.

__Note__: The image will automatically be named gitrepo-tag (i.e. MyRepo-v1.0.0)

### Searching
List the built images by using the AWS CLI:
```
aws ec2 describe-images --owners 867686887310 --filters Name=tag:Name,Values=my-test-image
```

## Contributions
Contributions are welcome.

Requirements:
* Install [pre-commit](https://pre-commit.com/#install) app
* Clone this repo
* Run `pre-commit install` to install the git hook.

## Deployments
Travis runs packer which temporarily deploys an EC2 to create an AMI.

## Continuous Integration
We have configured Travis to deploy updates.

## Issues
* https://sagebionetworks.jira.com/projects/IT

## Builds
We use travis CI to automatically build and deploy images. Setup a travis ci build
and add the AWS deployment credentials to the travis environment variables.

## Secrets
* We use the [AWS SSM](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html)
to store secrets for this project.
