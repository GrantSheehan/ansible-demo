## Overview
Downloads the `ethereum/go-ethereum` repo, verifies the signature on the commit, builds the docker image, uploads 3 built image to the Docker Hub registry, and creates a VPC and an autoscaling group, and and runs the image from a public Docker Hub registry.

## Instructions/Caveats

To run the playbook, you'll need to `pip install boto boto3 botocore awscli` and run `ansible-playbook playbook.yml -i inventory` after creating a public Docker Hub registry and updating the vars in `vars.yml`.

1. If you want to build and push the docker repo, you'll need to create a public repository on Docker Hub named "ethereum" and update the `dockerhub_` credentials in the `vars.yml` file.
2. You'll want to update the vars.yml file where appropriate, with your own EC2 key name, for example.
3. In order to verify the git commit in the `go-ethereum` repo, you'll need to have the ethereum maintainer's public key in your PGP keychain. The tag being used here was signed by Péter Szilágy. You can find a copy of his public key [here](https://ethereum.github.io/go-ethereum/downloads/)
4. The playbook is written in a way that allows you to switch regions but in testing I believe the CentOS atomic image the AMI filter is looking for only exists in us-east-1, and doesn't support the latest Elastic Network Adapter, which means it's not compatible with the latest generation of instances. I imagine the RHEL version has better compatibility and is more widely available, though.

## Changes I'd Make With More Time
1. I'd update the playbook to create a kubernetes cluster instead of just doing `docker run` in the instance's user_data script.
2. In this instance, I'm simply compiling the Ethereum project's Dockerfile and running it. If we needed to templatize our own Dockerfile, include secrets, integrate more closely with k8s, etc., I'd use `ansible-container` to manage the image lifecycle.
2. I'd push the image to a private repository. If we were on AWS, I'd probably use an ECR repository since [k8s supports pulling from them natively](https://kubernetes.io/docs/concepts/containers/images/#using-aws-ec2-container-registry). I'd obviously need to create a role and EC2 instance profile with an attached policy that had the required permissions outlined in the doc.
3. Depending on security requirements, I'd refactor the network to be more secure and fit in with whatever parameters they're intended to be used. I'm guessing in production, you wouldn't want each Ethereum node to get its own NAT'd IP, you'd want to use some VPC NACLs, and have a more restricted set of inbound and output ranges on the security groups, etc.
4. Drop SSH from the instance, security group, and NACLs to make the instances immutable. I left those things in in case you wanted SSH into the instance and snoop around.
5. Compile and distribute custom AMIs for the kubernetes hosts.
6. Add load-balancing, AZ-level outage redundancy.
