# AWS CloudFormation Nested Stacks

## Overview

AWS CloudFormation Nested Stacks allow you to break complex infrastructure into smaller, reusable templates. They help modularize infrastructure and enable teams to work independently on different components.

## Real-World Use Cases

* Microservices: Each service deployed via its own nested template.
* Enterprise infra: Reuse common templates (e.g., VPC, IAM roles).
* CI/CD pipelines: Manage environments (dev/test/prod) with shared base infra.

---

## Stack Architecture

```
MainTemplate.yml (Root Stack)
├── VPCStack.yml (VPC + 1 subnet)
├── AppStack.yml (EC2 instance in subnet)
└── DBStack.yml (can be expanded later)
```

---

## MainTemplate.yml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  VPCStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/templates/VPCStack.yml

  AppStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: VPCStack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/templates/AppStack.yml
      Parameters:
        VpcId: !GetAtt VPCStack.Outputs.VpcId
        SubnetId: !GetAtt VPCStack.Outputs.PublicSubnet
```

---

## VPCStack.yml (VPC + 1 Public Subnet)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: VPC with a single public subnet

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: MyVPC

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: PublicSubnet

Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: VPCID

  PublicSubnet:
    Value: !Ref PublicSubnet
    Export:
      Name: PublicSubnetID
```

---

## AppStack.yml (EC2 in subnet)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  VpcId:
    Type: String
  SubnetId:
    Type: String

Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0abcdef1234567890
      InstanceType: t2.micro
      NetworkInterfaces:
        - AssociatePublicIpAddress: true
          DeviceIndex: 0
          SubnetId: !Ref SubnetId
```

---

## Key Concepts

| Concept            | Description                        |
| ------------------ | ---------------------------------- |
| **Nested Stack**   | A stack within a parent stack      |
| **TemplateURL**    | S3 path to nested template         |
| **Outputs/Params** | Used to pass data between stacks   |
| **DependsOn**      | Controls order of stack deployment |

---

## Best Practices

* Store nested templates in versioned S3 paths
* Validate and test templates individually
* Keep output values consistent and meaningful
* Use exports for cross-stack references

---

## Summary

Nested stacks in CloudFormation make infrastructure more modular, manageable, and reusable — ideal for real-world team collaboration and scalable architectures.
