## Setting up a simple webpage on EC2 with a custom URL

In this blog we will create a basic static site accessed through a custom domain name.  You can purchase domains from many sites including [GoDaddy](http://godaddy.com/), [Google](https://domains.google/) etc.

### Setting up EC2

Once you have a domain, we can start creating an EC2 instance.  We first need to create a security group with the necessary ports to allow SSH and web hosting access.

```shell script
# create a security group called 'my-sg' after deleting it if it already exists
group_name=$(aws ec2 describe-security-groups --query 'SecurityGroups[?GroupName==`my-sg`]' --output=text)
if [[ ! -z "$group_name" ]]; then
  aws ec2 delete-security-group --group-name "my-sg"
fi

aws ec2 create-security-group --group-name "my-sg" --description "My security group"

# configure port permissions
SECURITY_GROUP_ID=$(aws ec2 describe-security-groups --group-name "my-sg" --query "SecurityGroups[*].GroupId" --output=text)
aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol tcp --port 80 --cidr 0.0.0.0/0
```

We create a single t2-micro EC2 instance and get its public IP.

```shell script
aws ec2 run-instances \
    --image-id ami-0274e11dced17bb5b \
    --count 1 \
    --instance-type t2.micro \
    --key-name my_key \
    --security-group-ids ${SECURITY_GROUP_ID}

# you should wait about 30 seconds for the instance to boot up
PUBLIC_IP=$(aws ec2 describe-instances \
            --filters Name=instance-state-name,Values=running \
            --query "Reservations[*].Instances[*].PublicIpAddress" \
            --output=text)
```

We will use a Nginx Docker image as our web server.  The following commands will install Docker on the EC2 instance and start the Nginx server.  Ensure you specify the correct name of your key pair file. 

```shell script
ssh -T -i ~/.ssh/my_key.pem -o StrictHostKeyChecking=no ec2-user@${PUBLIC_IP} << EOF
    sudo yum update -y && \
    sudo amazon-linux-extras install docker -y && \
    sudo service docker start && \
    sudo usermod -a -G docker ec2-user
EOF

ssh -i ~/.ssh/my_key.pem -o StrictHostKeyChecking=no ec2-user@${PUBLIC_IP} "docker run --rm -d -p 80:80 nginx"
```

We should now be able to navigate to this IP address (`echo ${PUBLIC_IP}`) in a browser.  If everything has worked you will see a message from Nginx.

We can customise this message by running something like:
```shell script
ssh -T -i ~/.ssh/my_key.pem -o StrictHostKeyChecking=no ec2-user@${PUBLIC_IP} "docker exec \$(docker ps -q) sh -c \"echo 'Welcome to my site' > /usr/share/nginx/html/index.html\""
```

### Using a custom URL

Let's now setup our domain to access this site.  We will use Amazon Route 53 as our DNS service.  We first need to create a hosted zone.  Set the `${DOMAIN}` variable to the domain name you bought e.g. `DOMAIN=example.com`.

```shell script
aws route53 create-hosted-zone --name ${DOMAIN} --caller-reference "$(date)"
ZONE_ID=$(aws route53 list-hosted-zones --query 'HostedZones[?Name==`'${DOMAIN}'.`].Id' --output=text)
```

We then need to update the nameservers in the domain name provider's settings.  For example, in GoDaddy you can find the nameservers under the DNS Management page in My Domains.  Select the Custom type and enter the four nameservers which you can get by running:
```shell script
aws route53 list-resource-record-sets \
    --hosted-zone-id ${ZONE_ID} \
    --query 'ResourceRecordSets[?Type==`NS`].ResourceRecords' \
    --output text | sed 's/.$//'
```

Finally, we need to create two new record sets of type "A" to allow both `example.com` and `www.example.com` to map to the IP address of our EC2 instance.

```shell script
aws route53 change-resource-record-sets \
    --hosted-zone-id ${ZONE_ID} \
    --change-batch '{"Changes": [{"Action": "CREATE", "ResourceRecordSet": {"Name": "www.'${DOMAIN}'", "SetIdentifier": "www record", "Type": "A", "Region": "eu-west-2", "TTL": 300, "ResourceRecords": [{"Value": "'${PUBLIC_IP}'"}]}}]}'

aws route53 change-resource-record-sets \
    --hosted-zone-id ${ZONE_ID} \
    --change-batch '{"Changes": [{"Action": "CREATE", "ResourceRecordSet": {"Name": "'${DOMAIN}'", "SetIdentifier": "blank record", "Type": "A", "Region": "eu-west-2", "TTL": 300, "ResourceRecords": [{"Value": "'${PUBLIC_IP}'"}]}}]}'
```

It can take anything from 30 minutes to 48 hours for the DNS records to update.

---
[Home](../index.md)
