1. LAB starten und alle aws-cli credentials im Terminal updaten. siehe unter learner lab details
2. Ubuntu ami der region `us-east-1` überprüfen. Sollte **ami-0360c520857e3138f** sein, falls nicht muss du es im unteren Teil "EC2 erstellen" anpassen.
3. Git klonen:
   `git clone https://github.com/taher-alsaegh/Flask-Python-E-Commerce-Website.git`
4. In aws-infra wechseln:
   `cd aws-infra`
5. SSH Key Pair erstellen:
   `ssh-keygen -t rsa -b 4096 -f ~/YOUR-PATH/devops_rsa`
6. Den pup key in beide cloud-init files hinzufügen. siehe aws-infra folder
7. Berechtigung des pub keys auf read only setzen:
   `chmod 400 devops_rsa`
8. aws-infra ausführen (muss im directory ausgeführt werden wo alle cloud-init files sind):
   `chmod +x aws-infra; ./aws-infra`

```bash
#!/usr/bin/env bash
# ===== Settings =====
REGION="us-east-1"
AZ="us-east-1a"

VPC_CIDR="10.11.5.0/24"
SUBNET_DMZ_CIDR="10.11.5.192/26"   # public
SUBNET_PRIV_CIDR="10.11.5.0/26"    # private

# ===== VPC =====
VPC_ID=$(aws ec2 create-vpc \
  --region "$REGION" \
  --cidr-block "$VPC_CIDR" \
  --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value=proj-vpc}]" \
  --query 'Vpc.VpcId' --output text)
aws ec2 modify-vpc-attribute --region "$REGION" --vpc-id "$VPC_ID" --enable-dns-hostnames
aws ec2 modify-vpc-attribute --region "$REGION" --vpc-id "$VPC_ID" --enable-dns-support
echo "VPC: $VPC_ID"

# ===== Subnets =====
SUBNET_DMZ_ID=$(aws ec2 create-subnet \
  --region "$REGION" --vpc-id "$VPC_ID" \
  --cidr-block "$SUBNET_DMZ_CIDR" \
  --availability-zone "$AZ" \
  --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=dmz-public-10.11.5.192-26}]" \
  --query 'Subnet.SubnetId' --output text)
aws ec2 modify-subnet-attribute --region "$REGION" --subnet-id "$SUBNET_DMZ_ID" --map-public-ip-on-launch

SUBNET_PRIV_ID=$(aws ec2 create-subnet \
  --region "$REGION" --vpc-id "$VPC_ID" \
  --cidr-block "$SUBNET_PRIV_CIDR" \
  --availability-zone "$AZ" \
  --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=default-private-10.11.5.0-26}]" \
  --query 'Subnet.SubnetId' --output text)
echo "Subnets: DMZ=$SUBNET_DMZ_ID  PRIV=$SUBNET_PRIV_ID"

# ===== Internet Gateway =====
IGW_ID=$(aws ec2 create-internet-gateway \
  --region "$REGION" \
  --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=proj-igw}]" \
  --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --region "$REGION" --internet-gateway-id "$IGW_ID" --vpc-id "$VPC_ID"
echo "IGW: $IGW_ID"

# ===== NAT Gateway + Elastic IP (im DMZ) =====
EIP_ALLOC_ID=$(aws ec2 allocate-address \
  --region "$REGION" --domain vpc \
  --tag-specifications "ResourceType=elastic-ip,Tags=[{Key=Name,Value=proj-nat-eip}]" \
  --query 'AllocationId' --output text)

NAT_GW_ID=$(aws ec2 create-nat-gateway \
  --region "$REGION" \
  --subnet-id "$SUBNET_DMZ_ID" \
  --allocation-id "$EIP_ALLOC_ID" \
  --tag-specifications "ResourceType=natgateway,Tags=[{Key=Name,Value=proj-nat}]" \
  --query 'NatGateway.NatGatewayId' --output text)
echo "NAT GW: $NAT_GW_ID (warten bis available...)"
aws ec2 wait nat-gateway-available --region "$REGION" --nat-gateway-ids "$NAT_GW_ID"

# ===== Route Tables =====
# Public RT: 0.0.0.0/0 -> IGW, assoc DMZ
RT_PUBLIC_ID=$(aws ec2 create-route-table \
  --region "$REGION" --vpc-id "$VPC_ID" \
  --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=proj-rt-public}]" \
  --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --region "$REGION" --route-table-id "$RT_PUBLIC_ID" --destination-cidr-block 0.0.0.0/0 --gateway-id "$IGW_ID"
aws ec2 associate-route-table --region "$REGION" --subnet-id "$SUBNET_DMZ_ID" --route-table-id "$RT_PUBLIC_ID"

# Private RT: 0.0.0.0/0 -> NAT, assoc Private
RT_PRIVATE_ID=$(aws ec2 create-route-table \
  --region "$REGION" --vpc-id "$VPC_ID" \
  --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=proj-rt-private}]" \
  --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --region "$REGION" --route-table-id "$RT_PRIVATE_ID" --destination-cidr-block 0.0.0.0/0 --nat-gateway-id "$NAT_GW_ID"
aws ec2 associate-route-table --region "$REGION" --subnet-id "$SUBNET_PRIV_ID" --route-table-id "$RT_PRIVATE_ID"
echo "RouteTables: PUBLIC=$RT_PUBLIC_ID PRIVATE=$RT_PRIVATE_ID"

# ===== Security Groups =====
# webshop_sg: TCP 22,80 von überall; egress alle
SG_WEBSHOP_ID=$(aws ec2 create-security-group \
  --region "$REGION" \
  --group-name "webshop_sg" \
  --description "Allow SSH/HTTP from anywhere" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --region "$REGION" --group-id "$SG_WEBSHOP_ID" \
  --ip-permissions '[
    {"IpProtocol":"tcp","FromPort":22,"ToPort":22,"IpRanges":[{"CidrIp":"0.0.0.0/0","Description":"SSH"}]},
    {"IpProtocol":"tcp","FromPort":80,"ToPort":80,"IpRanges":[{"CidrIp":"0.0.0.0/0","Description":"HTTP"}]}
  ]'
aws ec2 authorize-security-group-egress --region "$REGION" --group-id "$SG_WEBSHOP_ID" \
  --ip-permissions '[{"IpProtocol":"-1","IpRanges":[{"CidrIp":"0.0.0.0/0","Description":"All outbound"}]}]'

# webserveronly: TCP 22,3306 nur intern (ganze VPC 10.11.5.0/24); egress alle
SG_WEBSERVERONLY_ID=$(aws ec2 create-security-group \
  --region "$REGION" \
  --group-name "webserveronly" \
  --description "SSH/MySQL only from internal subnets" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --region "$REGION" --group-id "$SG_WEBSERVERONLY_ID" \
  --ip-permissions "[
    {\"IpProtocol\":\"tcp\",\"FromPort\":22,\"ToPort\":22,\"IpRanges\":[{\"CidrIp\":\"$VPC_CIDR\",\"Description\":\"SSH internal\"}]},
    {\"IpProtocol\":\"tcp\",\"FromPort\":3306,\"ToPort\":3306,\"IpRanges\":[{\"CidrIp\":\"$VPC_CIDR\",\"Description\":\"MySQL internal\"}]}
  ]"
aws ec2 authorize-security-group-egress --region "$REGION" --group-id "$SG_WEBSERVERONLY_ID" \
  --ip-permissions '[{"IpProtocol":"-1","IpRanges":[{"CidrIp":"0.0.0.0/0","Description":"All outbound"}]}]'
echo "SGs: webshop=$SG_WEBSHOP_ID  webserveronly=$SG_WEBSERVERONLY_ID"

# ===== Network ACLs (Allow-All) + Associations =====
# Public NACL -> DMZ
NACL_PUB_ID=$(aws ec2 create-network-acl \
  --region "$REGION" --vpc-id "$VPC_ID" \
  --tag-specifications "ResourceType=network-acl,Tags=[{Key=Name,Value=proj-acl-public-allowall}]" \
  --query 'NetworkAcl.NetworkAclId' --output text)
aws ec2 create-network-acl-entry --region "$REGION" --network-acl-id "$NACL_PUB_ID" \
  --ingress --rule-number 100 --protocol -1 --rule-action allow --cidr-block 0.0.0.0/0
aws ec2 create-network-acl-entry --region "$REGION" --network-acl-id "$NACL_PUB_ID" \
  --egress --rule-number 100 --protocol -1 --rule-action allow --cidr-block 0.0.0.0/0
aws ec2 associate-network-acl --region "$REGION" --network-acl-id "$NACL_PUB_ID" --subnet-id "$SUBNET_DMZ_ID"

# Private NACL -> Private
NACL_PRIV_ID=$(aws ec2 create-network-acl \
  --region "$REGION" --vpc-id "$VPC_ID" \
  --tag-specifications "ResourceType=network-acl,Tags=[{Key=Name,Value=proj-acl-private-allowall}]" \
  --query 'NetworkAcl.NetworkAclId' --output text)
aws ec2 create-network-acl-entry --region "$REGION" --network-acl-id "$NACL_PRIV_ID" \
  --ingress --rule-number 100 --protocol -1 --rule-action allow --cidr-block 0.0.0.0/0
aws ec2 create-network-acl-entry --region "$REGION" --network-acl-id "$NACL_PRIV_ID" \
  --egress --rule-number 100 --protocol -1 --rule-action allow --cidr-block 0.0.0.0/0
aws ec2 associate-network-acl --region "$REGION" --network-acl-id "$NACL_PRIV_ID" --subnet-id "$SUBNET_PRIV_ID"
echo "NACLs: PUBLIC=$NACL_PUB_ID PRIVATE=$NACL_PRIV_ID"

# =====Create EC2 Webshop=====
aws ec2 run-instances \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=webserver}]' \
  --region us-east-1 \
  --image-id ami-0360c520857e3138f \
  --count 1 \
  --instance-type t3.micro \
  --subnet-id "$SUBNET_DMZ_ID" \
  --private-ip-address 10.11.5.201 \
  --security-group-ids "$SG_WEBSHOP_ID" \
  --user-data file://cloud-init-webshop.yml

# =====Create EC2 Database=====
aws ec2 run-instances \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=database}]' \
  --region us-east-1 \
  --image-id ami-0360c520857e3138f \
  --count 1 \
  --instance-type t3.micro \
  --subnet-id "$SUBNET_PRIV_ID" \
  --private-ip-address 10.11.5.5 \
  --security-group-ids "$SG_WEBSERVERONLY_ID" \
  --user-data file://cloud-init-database.yml

# ===== Summary =====
echo
echo "DONE"
cat <<EOF
VPC_ID=$VPC_ID
SUBNET_DMZ_ID=$SUBNET_DMZ_ID
SUBNET_PRIV_ID=$SUBNET_PRIV_ID
IGW_ID=$IGW_ID
NAT_GW_ID=$NAT_GW_ID
RT_PUBLIC_ID=$RT_PUBLIC_ID
RT_PRIVATE_ID=$RT_PRIVATE_ID
SG_WEBSHOP_ID=$SG_WEBSHOP_ID
SG_WEBSERVERONLY_ID=$SG_WEBSERVERONLY_ID
NACL_PUB_ID=$NACL_PUB_ID
NACL_PRIV_ID=$NACL_PRIV_ID
EOF
```

6. Per SCP den Private Key auf den Webserver kopieren (Not secure)
   `scp -i <PRV-KEY> <PRV-KEY> ubuntu@<IP-WEBSERVER>:`
