# Amazon EKS AMI Build Specification

This repository contains resources and configuration scripts for building a
custom Amazon EKS AMI with [HashiCorp Packer](https://www.packer.io/). This is
the same configuration that Amazon EKS uses to create the official Amazon
EKS-optimized AMI [awslabs/amazon-eks-ami](https://github.com/awslabs/amazon-eks-ami).

## CheetahDigital Customizations
For CheetahDigital usage we make several modification from the original repository form 
github [awslabs/amazon-eks-ami](https://github.com/awslabs/amazon-eks-ami/releases)

- Create `rpms` directory and add CrowdStrike rpm pkg which can be found [HERE](./files/rpms)
- Modify the `install-worker.sh` script which can be found [HERE](./scripts/install-worker.sh#262):
  - Installs and enable:
  	- AWS SSM
  	- CrowdStrike
  	- Qualys
  - Custom log rotate and cron for Stellar log
  
  > **_NOTE:_** Refer to from line number 254 until 316

- Modify the packer config file `eks-worker-al2.json` which can be found [HERE](./eks-worker-al2.json)
  - Add PACKER_PROFILE & PACKER_REGION in `variables`.
    ```json
      "variables": {
        "aws_region": "{{env `PACKER_REGION`}}",
        "aws_profile": "{{env `PACKER_PROFILE`}}",
      ...
    ```

  - Remove aws_* key references since we using aws_profile instead :
    ```json
    "aws_access_key_id": "{{env `AWS_ACCESS_KEY_ID`}}",
    "aws_secret_access_key": "{{env `AWS_SECRET_ACCESS_KEY`}}",
    "aws_session_token": "{{env `AWS_SESSION_TOKEN`}}",
    ```

  - Add profile in `builders` section.
    ```json
      "builders": [
        {
          "type": "amazon-ebs",
          "region": "{{user `aws_region`}}",
          "profile": "{{user `aws_profile`}}",
      ...
    ```

## Upgrade code
The way to upgrade the code by download the releases tar file form github [awslabs/amazon-eks-ami](https://github.com/awslabs/amazon-eks-ami/releases).
Then take note above changes and sync the customization that we had above.

***Change Log:-***

For more info , read CHANGELOG.MD [HERE](./CHANGELOG.md)

| Date | Referenced Release Version | Docker Version | Containerd version |
|------|----------------------------|----------------|--------------------|
| Sept 18 2020 | v20200904 | | |
| Jul 9 2020   | v20200618 | | |
| Nov 12 2020  | v20201007 | | |
| Jan 06 2021  | v20201211| | |
| Mar 26 2021  | v20210322| 19.03.13ce-1.amzn2 | |
| Nov 22 2021  | v20211109| 20.10.7-5.amzn2 | |
| Jan 17 2022  | v20220112| 20.10.7-5.amzn2| |
| Apr 28 2022  | v20220421| 20.10.13-2.amzn2 | |
| Sept 27 2022 | v20220421| 20.10.17-1.amzn2 | |
| Jan 13 2023  | v20230105| 20.10.17-1.amzn2.0.1 | 1.6.6-1.amzn2.0.2|
| Feb 09 2023  | v20230203| 20.10.17-1.amzn2.0.1 | 1.6.6-1.amzn2.0.2|
| May 09 2023  | v20230501|  ||
## Modify Choice Parameter
To modify the AWS_ACCOUNT, AWS_REGION & EKS_VERSION open [Jenkinsfile](./Jenkinsfile) and add into the parameters choice section accordingly.


## Building the AMI

Go to this build [URL](https://cd-jenkins.cheetahmail.com/job/aws_infra/job/packer/job/eks-ami/)
Click on Build with Parameters
Choose your target aws environment/account, region and eks version from the drop-down and click Build.

Region remark:
- us-east-1 = US
- us-east-2 = US
- ap-southeast-2 = Australia
- ap-northeast-1 = Japan
- eu-central-1 = EMEA

## AWS CLI to find the ami
```bash
aws ec2 describe-images \
--owners self \
--filters 'Name=name,Values=amazon-eks-node-1.16-v20201001' 'Name=state,Values=available' \
--query 'reverse(sort_by(Images, &CreationDate))[:1].ImageId' \
--profile cheetahdigital-edp-au-prod \
--region ap-southeast-2 \
--output text

# Note: modify the filter, profile and region accordingly
```

## Security

For security issues or concerns, please do not open an issue or pull request on GitHub. Please report any suspected or confirmed security issues to AWS Security https://aws.amazon.com/security/vulnerability-reporting/

## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.
