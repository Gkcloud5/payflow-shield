
```
AWSTemplateFormatVersion: "2010-09-09"
Description: "PayFlow Shield - Network Stack - VPC, Subnets, IGW, NAT, Route Tables"

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]
    Description: Deployment environment

  ProjectName:
    Type: String
    Default: payflow-shield
    Description: Project name used for tagging

# ─────────────────────────────────────────
# RESOURCES
# ─────────────────────────────────────────
Resources:

  # ── VPC ──────────────────────────────
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-vpc"

  # ── INTERNET GATEWAY ─────────────────
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-igw"

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  # ── PUBLIC SUBNETS ───────────────────
  PublicSubnetAZ1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: ap-south-1a
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-public-az1"

  PublicSubnetAZ2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: ap-south-1b
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-public-az2"

  # ── PRIVATE APP SUBNETS ──────────────
  PrivateAppSubnetAZ1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.11.0/24
      AvailabilityZone: ap-south-1a
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-private-app-az1"

  PrivateAppSubnetAZ2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.12.0/24
      AvailabilityZone: ap-south-1b
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-private-app-az2"

  # ── PRIVATE DB SUBNETS ───────────────
  PrivateDBSubnetAZ1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.21.0/24
      AvailabilityZone: ap-south-1a
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-private-db-az1"

  PrivateDBSubnetAZ2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.22.0/24
      AvailabilityZone: ap-south-1b
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-private-db-az2"

  # ── NAT GATEWAY ──────────────────────
  NATGatewayEIP:
    Type: AWS::EC2::EIP
    DependsOn: AttachGateway
    Properties:
      Domain: vpc

  NATGateway:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt NATGatewayEIP.AllocationId
      SubnetId: !Ref PublicSubnetAZ1
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-nat"

  # ── ROUTE TABLES ─────────────────────
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-public-rt"

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnetAZ1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnetAZ1
      RouteTableId: !Ref PublicRouteTable

  PublicSubnetAZ2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnetAZ2
      RouteTableId: !Ref PublicRouteTable

  PrivateRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-private-rt"

  PrivateRoute:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NATGateway

  PrivateAppSubnetAZ1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateAppSubnetAZ1
      RouteTableId: !Ref PrivateRouteTable

  PrivateAppSubnetAZ2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateAppSubnetAZ2
      RouteTableId: !Ref PrivateRouteTable

  PrivateDBSubnetAZ1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateDBSubnetAZ1
      RouteTableId: !Ref PrivateRouteTable

  PrivateDBSubnetAZ2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateDBSubnetAZ2
      RouteTableId: !Ref PrivateRouteTable

# ─────────────────────────────────────────
# OUTPUTS — exported for other stacks
# ─────────────────────────────────────────
Outputs:
  VPCId:
    Value: !Ref VPC
    Export:
      Name: !Sub "${ProjectName}-${Environment}-VPCId"

  PublicSubnetAZ1:
    Value: !Ref PublicSubnetAZ1
    Export:
      Name: !Sub "${ProjectName}-${Environment}-PublicSubnetAZ1"

  PublicSubnetAZ2:
    Value: !Ref PublicSubnetAZ2
    Export:
      Name: !Sub "${ProjectName}-${Environment}-PublicSubnetAZ2"

  PrivateAppSubnetAZ1:
    Value: !Ref PrivateAppSubnetAZ1
    Export:
      Name: !Sub "${ProjectName}-${Environment}-PrivateAppSubnetAZ1"

  PrivateAppSubnetAZ2:
    Value: !Ref PrivateAppSubnetAZ2
    Export:
      Name: !Sub "${ProjectName}-${Environment}-PrivateAppSubnetAZ2"

  PrivateDBSubnetAZ1:
    Value: !Ref PrivateDBSubnetAZ1
    Export:
      Name: !Sub "${ProjectName}-${Environment}-PrivateDBSubnetAZ1"

  PrivateDBSubnetAZ2:
    Value: !Ref PrivateDBSubnetAZ2
    Export:
      Name: !Sub "${ProjectName}-${Environment}-PrivateDBSubnetAZ2"
```