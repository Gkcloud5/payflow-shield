
```
AWSTemplateFormatVersion: "2010-09-09"
Description: "PayFlow Shield - SQS Stack - Commission Job Queue + DLQ"

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

  ProjectName:
    Type: String
    Default: payflow-shield

Resources:

  # ── DEAD LETTER QUEUE ─────────────────────────────────
  # Catches jobs that fail 3 times
  CommissionDLQ:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: !Sub "${ProjectName}-${Environment}-commission-dlq"
      MessageRetentionPeriod: 1209600
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-commission-dlq"

  # ── MAIN COMMISSION QUEUE ─────────────────────────────
  # Receives all incoming commission jobs
  CommissionQueue:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: !Sub "${ProjectName}-${Environment}-commission-queue"
      VisibilityTimeout: 300
      MessageRetentionPeriod: 86400
      RedrivePolicy:
        deadLetterTargetArn: !GetAtt CommissionDLQ.Arn
        maxReceiveCount: 3
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-commission-queue"

Outputs:
  CommissionQueueUrl:
    Value: !Ref CommissionQueue
    Export:
      Name: !Sub "${ProjectName}-${Environment}-CommissionQueueUrl"

  CommissionQueueArn:
    Value: !GetAtt CommissionQueue.Arn
    Export:
      Name: !Sub "${ProjectName}-${Environment}-CommissionQueueArn"

  CommissionDLQUrl:
    Value: !Ref CommissionDLQ
    Export:
      Name: !Sub "${ProjectName}-${Environment}-CommissionDLQUrl"

  CommissionDLQArn:
    Value: !GetAtt CommissionDLQ.Arn
    Export:
      Name: !Sub "${ProjectName}-${Environment}-CommissionDLQArn"
```




