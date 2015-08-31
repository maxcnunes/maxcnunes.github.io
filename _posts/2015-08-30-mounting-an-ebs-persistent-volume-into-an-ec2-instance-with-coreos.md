---
layout: post
title: Mounting an EBS persistent volume into an EC2 instance with CoreOS
categories:
- coreos
- aws
tags:
- coreos
- aws
- ec2
- ebs
- persistent
- volume
- mount
---

In the last weeks I have been working a lot with [AWS](https://aws.amazon.com/). At Bravi we are deploying most of our production services and applications on AWS through the [CloudFormation](https://aws.amazon.com/cloudformation/). Which really is a powerful tool to provision the whole infrastructure.

CloudFormation uses templates to specify the stack configuration. This template is a JSON-formatted text file which can be used to create a new stack or apply update changes to an existing stack.

Since it is my first experience with AWS and CloudFormation almost everything is new for me. And some times can take a long time of several attempts to figure out how to configure some stuff that are not really well documented in the AWS website.

That is why I'm going to try post here the solution I found for some of those problems.

I think the first problem I had was mounting an EBS persistent volume into an EC2 instance with CoreOS.

At the end the solution was pretty simple:

- create the volume using the `AWS::EC2::Volume`
- attach the volume into the ec2 instance through the `Volumes` property
- add a unit into the **cloud-config** to format the volume in case it is a new volume
- add another unit to mount the volume

```json
{
  ...

  "Resources": {
    "EBSVolume" : {
      "Type" : "AWS::EC2::Volume",
      "Properties" : {
        "Size" : "20",
        "AvailabilityZone" : "us-east-1b"
      },
      "DeletionPolicy" : "Snapshot"
    },

    "EC2Instance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "ImageId": { "Ref": "CoreOSAMI" },
        "InstanceType": "t2.small",
        "KeyName": { "Ref": "KeyPair" },
        "NetworkInterfaces": [
          {
            ...
          }
        ],
        "Volumes" : [
          {
            "VolumeId" : { "Ref" : "EBSVolume" },
            "Device" : "/dev/sdb"
          }
        ],
        "UserData" : {
          "Fn::Base64": {
            "Fn::Join": [ "", [
                "#cloud-config\n\n",
                "\n\n",
                "coreos:\n",
                "  update:\n",
                "    reboot-strategy: off\n",
                "  units:\n",
                "    - name: format-ebs-volume.service\n",
                "      command: start\n",
                "      content: |\n",
                "        [Unit]\n",
                "        Description=Formats the ebs volume if needed\n",
                "        Before=docker.service\n",
                "        [Service]\n",
                "        Type=oneshot\n",
                "        RemainAfterExit=yes\n",
                "        ExecStart=/bin/bash -c '(/usr/sbin/blkid -t TYPE=ext4 | grep /dev/xvdb) || (/usr/sbin/wipefs -fa /dev/xvdb && /usr/sbin/mkfs.ext4 /dev/xvdb)'\n",
                "\n",
                "    - name: data.mount\n",
                "      command: start\n",
                "      content: |\n",
                "        [Unit]\n",
                "        Description=Mount EBS Volume to /data\n",
                "        Requires=format-ebs-volume.service\n",
                "        After=format-ebs-volume.service\n",
                "        Before=docker.service\n",
                "        [Mount]\n",
                "        What=/dev/xvdb\n",
                "        Where=/data\n",
                "        Type=ext4\n",
                "\n",
                ""
              ]
            ]
          }
        }
      }
    }
  }
}
```
