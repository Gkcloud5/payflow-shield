
```
Internet → ALB only (port 80, 443)
ALB → App EC2 only (port 8080)
App EC2 → RDS only (port 5432)
App EC2 → outbound internet via NAT (for updates)
Bastion → App EC2 only (port 22)
```

``` yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "PayFlow Shield - Security Groups Stack"

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

  ProjectName:
    Type: String
    Default: payflow-shield

Resources:

  # ── ALB SECURITY GROUP ───────────────────────────────
  # Accepts traffic from the internet on port 80 and 443
  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "ALB - accepts internet traffic"
      VpcId: !ImportValue
        "Fn::Sub": "${ProjectName}-${Environment}-VPCId"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-alb-sg"

  # ── APP SECURITY GROUP ───────────────────────────────
  # Accepts traffic only from the ALB
  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "App EC2 - accepts traffic from ALB only"
      VpcId: !ImportValue
        "Fn::Sub": "${ProjectName}-${Environment}-VPCId"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 8080
          ToPort: 8080
          SourceSecurityGroupId: !Ref ALBSecurityGroup
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          SourceSecurityGroupId: !Ref BastionSecurityGroup
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-app-sg"

  # ── DB SECURITY GROUP ────────────────────────────────
  # Accepts traffic only from App EC2
  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "RDS - accepts traffic from App EC2 only"
      VpcId: !ImportValue
        "Fn::Sub": "${ProjectName}-${Environment}-VPCId"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 5432
          ToPort: 5432
          SourceSecurityGroupId: !Ref AppSecurityGroup
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-db-sg"

  # ── BASTION SECURITY GROUP ───────────────────────────
  # SSH access — restrict this to your IP in production
  BastionSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Bastion Host - SSH access"
      VpcId: !ImportValue
        "Fn::Sub": "${ProjectName}-${Environment}-VPCId"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-bastion-sg"

Outputs:
  ALBSecurityGroupId:
    Value: !Ref ALBSecurityGroup
    Export:
      Name: !Sub "${ProjectName}-${Environment}-ALBSecurityGroupId"

  AppSecurityGroupId:
    Value: !Ref AppSecurityGroup
    Export:
      Name: !Sub "${ProjectName}-${Environment}-AppSecurityGroupId"

  DBSecurityGroupId:
    Value: !Ref DBSecurityGroup
    Export:
      Name: !Sub "${ProjectName}-${Environment}-DBSecurityGroupId"

  BastionSecurityGroupId:
    Value: !Ref BastionSecurityGroup
    Export:
      Name: !Sub "${ProjectName}-${Environment}-BastionSecurityGroupId"
```