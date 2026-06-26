
For db password we are going to use Systems manager parameter store

`AWS Console → Systems Manager → Parameter Store → Create Parameter`

after that used security manager to create secret key

```
AWSTemplateFormatVersion: "2010-09-09"

Description: "PayFlow Shield - Database Stack - RDS PostgreSQL Single-AZ (dev)"

  

Parameters:

  Environment:

    Type: String

    Default: dev

    AllowedValues: [dev, staging, prod]

  

  ProjectName:

    Type: String

    Default: payflow-shield

  

  DBInstanceClass:

    Type: String

    Default: db.t3.micro

  

  DBName:

    Type: String

    Default: payflowdb

  

  DBUsername:

    Type: String

    Default: payflowadmin

  

Resources:

  

  # ── SECRETS MANAGER ───────────────────────────────────

  DBSecret:

    Type: AWS::SecretsManager::Secret

    Properties:

      Name: !Sub "${ProjectName}/${Environment}/db-password"

      Description: "PayFlow Shield RDS Master Password"

      GenerateSecretString:

        SecretStringTemplate: !Sub '{"username": "${DBUsername}"}'

        GenerateStringKey: "password"

        PasswordLength: 32               # 32 is stronger than 16

        ExcludeCharacters: '"@/\'

      Tags:

        - Key: Name

          Value: !Sub "${ProjectName}-${Environment}-db-secret"

  

  # ── DB SUBNET GROUP ───────────────────────────────────

  DBSubnetGroup:

    Type: AWS::RDS::DBSubnetGroup

    Properties:

      DBSubnetGroupDescription: "PayFlow Shield DB Subnet Group"

      SubnetIds:

        - !ImportValue

          "Fn::Sub": "${ProjectName}-${Environment}-PrivateDBSubnetAZ1"

        - !ImportValue

          "Fn::Sub": "${ProjectName}-${Environment}-PrivateDBSubnetAZ2"

      Tags:

        - Key: Name

          Value: !Sub "${ProjectName}-${Environment}-db-subnet-group"

  

  # ── RDS POSTGRESQL INSTANCE ───────────────────────────

  RDSInstance:

    Type: AWS::RDS::DBInstance

    DependsOn: DBSecret

    DeletionPolicy: Delete

    Properties:

      DBInstanceIdentifier: !Sub "${ProjectName}-${Environment}-postgres"

      DBInstanceClass: !Ref DBInstanceClass

      Engine: postgres

      EngineVersion: "15.12"            # ← Updated

      MasterUsername: !Ref DBUsername

      MasterUserPassword: !Sub          # ← Bug fixed here

        - "{{resolve:secretsmanager:${SecretArn}:SecretString:password}}"

        - SecretArn: !Ref DBSecret

      DBName: !Ref DBName

      DBSubnetGroupName: !Ref DBSubnetGroup

      VPCSecurityGroups:

        - !ImportValue

          "Fn::Sub": "${ProjectName}-${Environment}-DBSecurityGroupId"

      AllocatedStorage: "20"

      StorageType: gp2

      MultiAZ: false

      PubliclyAccessible: false

      BackupRetentionPeriod: 0

      DeletionProtection: false

      Tags:

        - Key: Name

          Value: !Sub "${ProjectName}-${Environment}-postgres"

  

  # ── SECRET RDS ATTACHMENT ─────────────────────────────

  SecretRDSAttachment:

    Type: AWS::SecretsManager::SecretTargetAttachment

    Properties:

      SecretId: !Ref DBSecret

      TargetId: !Ref RDSInstance

      TargetType: AWS::RDS::DBInstance

  

Outputs:

  RDSEndpoint:

    Value: !GetAtt RDSInstance.Endpoint.Address

    Export:

      Name: !Sub "${ProjectName}-${Environment}-RDSEndpoint"

  

  RDSPort:

    Value: !GetAtt RDSInstance.Endpoint.Port

    Export:

      Name: !Sub "${ProjectName}-${Environment}-RDSPort"

  

  DBName:

    Value: !Ref DBName

    Export:

      Name: !Sub "${ProjectName}-${Environment}-DBName"

  

  DBSecretArn:

    Value: !Ref DBSecret

    Export:

      Name: !Sub "${ProjectName}-${Environment}-DBSecretArn"
```
