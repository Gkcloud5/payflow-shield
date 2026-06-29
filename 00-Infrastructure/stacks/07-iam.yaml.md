
### IAM role for EC2 worker:

* Read message from SQS
* Write results to RDS
* Fetch DB password from secret manager
* Write logs to cloudwatch


```
AWSTemplateFormatVersion: "2010-09-09"
Description: "PayFlow Shield - IAM Stack - EC2 Worker Role"

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

  ProjectName:
    Type: String
    Default: payflow-shield

Resources:

  # ── EC2 WORKER IAM ROLE ───────────────────────────────
  WorkerRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub "${ProjectName}-${Environment}-worker-role"
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole

  # ── WORKER POLICY ─────────────────────────────────────
  WorkerPolicy:
    Type: AWS::IAM::Policy
    Properties:
      PolicyName: !Sub "${ProjectName}-${Environment}-worker-policy"
      Roles:
        - !Ref WorkerRole
      PolicyDocument:
        Version: "2012-10-17"
        Statement:

          # SQS — read and delete messages from commission queue
          - Effect: Allow
            Action:
              - sqs:ReceiveMessage
              - sqs:DeleteMessage
              - sqs:GetQueueAttributes
              - sqs:ChangeMessageVisibility
            Resource: !ImportValue
              "Fn::Sub": "${ProjectName}-${Environment}-CommissionQueueArn"

          # Secrets Manager — fetch DB password only
          - Effect: Allow
            Action:
              - secretsmanager:GetSecretValue
            Resource: !ImportValue
              "Fn::Sub": "${ProjectName}-${Environment}-DBSecretArn"

          # CloudWatch — write logs and metrics
          - Effect: Allow
            Action:
              - logs:CreateLogGroup
              - logs:CreateLogStream
              - logs:PutLogEvents
              - cloudwatch:PutMetricData
            Resource: "*"

  # ── INSTANCE PROFILE ──────────────────────────────────
  # Attaches the role to EC2 instances
  WorkerInstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      InstanceProfileName: !Sub "${ProjectName}-${Environment}-worker-profile"
      Roles:
        - !Ref WorkerRole

Outputs:
  WorkerRoleArn:
    Value: !GetAtt WorkerRole.Arn
    Export:
      Name: !Sub "${ProjectName}-${Environment}-WorkerRoleArn"

  WorkerInstanceProfileArn:
    Value: !GetAtt WorkerInstanceProfile.Arn
    Export:
      Name: !Sub "${ProjectName}-${Environment}-WorkerInstanceProfileArn"

  WorkerInstanceProfileName:
    Value: !Ref WorkerInstanceProfile
    Export:
      Name: !Sub "${ProjectName}-${Environment}-WorkerInstanceProfileName"
```