``` yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "PayFlow Shield - Compute Stack - Bastion Host + ALB"

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

  ProjectName:
    Type: String
    Default: payflow-shield

  BastionAMI:
    Type: String
    Default: ami-0bc7aabcf58d1e02a
    Description: "Amazon Linux 2023 AMI for ap-south-1"

  BastionInstanceType:
    Type: String
    Default: t3.micro
    Description: "Free tier eligible"

  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName
    Default: test-iam-new
    Description: "EC2 Key Pair for Bastion SSH access"

Resources:

  # ── BASTION HOST ─────────────────────────────────────
  BastionHost:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref BastionInstanceType
      ImageId: !Ref BastionAMI
      KeyName: !Ref KeyPairName
      SubnetId: !ImportValue
        "Fn::Sub": "${ProjectName}-${Environment}-PublicSubnetAZ1"
      SecurityGroupIds:
        - !ImportValue
          "Fn::Sub": "${ProjectName}-${Environment}-BastionSecurityGroupId"
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-bastion"

  # ── APPLICATION LOAD BALANCER ─────────────────────────
  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: !Sub "${ProjectName}-${Environment}-alb"
      Scheme: internet-facing
      Type: application
      Subnets:
        - !ImportValue
          "Fn::Sub": "${ProjectName}-${Environment}-PublicSubnetAZ1"
        - !ImportValue
          "Fn::Sub": "${ProjectName}-${Environment}-PublicSubnetAZ2"
      SecurityGroups:
        - !ImportValue
          "Fn::Sub": "${ProjectName}-${Environment}-ALBSecurityGroupId"
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-alb"

  # ── ALB TARGET GROUP ──────────────────────────────────
  ALBTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: !Sub "${ProjectName}-${Environment}-tg"
      Port: 8080
      Protocol: HTTP
      VpcId: !ImportValue
        "Fn::Sub": "${ProjectName}-${Environment}-VPCId"
      HealthCheckPath: /health
      HealthCheckIntervalSeconds: 30
      HealthyThresholdCount: 2
      UnhealthyThresholdCount: 3
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-tg"

  # ── ALB LISTENER ──────────────────────────────────────
  ALBListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref ApplicationLoadBalancer
      Port: 80
      Protocol: HTTP
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref ALBTargetGroup

Outputs:
  BastionHostId:
    Value: !Ref BastionHost
    Export:
      Name: !Sub "${ProjectName}-${Environment}-BastionHostId"

  BastionPublicIP:
    Value: !GetAtt BastionHost.PublicIp
    Export:
      Name: !Sub "${ProjectName}-${Environment}-BastionPublicIP"

  ALBArn:
    Value: !Ref ApplicationLoadBalancer
    Export:
      Name: !Sub "${ProjectName}-${Environment}-ALBArn"

  ALBDNSName:
    Value: !GetAtt ApplicationLoadBalancer.DNSName
    Export:
      Name: !Sub "${ProjectName}-${Environment}-ALBDNSName"

  ALBTargetGroupArn:
    Value: !Ref ALBTargetGroup
    Export:
      Name: !Sub "${ProjectName}-${Environment}-ALBTargetGroupArn"
```

Initially i forget to add key so once the stack is created i have added key pair information in yaml file and updated the stack i already created.

