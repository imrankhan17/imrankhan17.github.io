## Running an Apache web server using Docker on EC2

Create an EC2 instance via the AWS Console or with the following CLI command:

```bash
aws ec2 run-instances \
    --image-id ami-0274e11dced17bb5b \
    --count 1 \
    --instance-type t2.micro \
    --key-name my_key \
    --security-group-ids sg-0d7ed0aa18a3dba3f
```

You should use your own ID for the `--security-group-ids` flag and ensure it includes a inbound rule to allow connections through port 80.  Instructions for this can be found in step 6 [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-LAMP.html).

Connect to the instance: `ssh -i ~/.ssh/my_key.pem ec2-user@18.130.136.192`

Update packages: `sudo yum update -y`

Install Docker: `sudo amazon-linux-extras install docker -y`

Start Docker: `sudo service docker start`

Try running `docker ps`.  You should see an error.  To allow the `ec2-user` username to execute Docker commands without using `sudo`, run: `sudo usermod -a -G docker ec2-user`

`exit` out of the EC2 instance and `ssh` back in.  Try running `docker ps` again.

Pull the Apache HTTP server image: `docker pull httpd`

Start the web server: `docker run -d -p 80:80 httpd`

Then go to a browser and enter the public IP address.  You should see a header saying `It works!`.

To edit this page, log into the Docker container: `docker exec -it cedb8a298984 bash`.  The container ID can be found using `docker ps`.

Go to the `/usr/local/apache2/htdocs/` directory where you can edit the `index.html` file.

---
[Home](../index.md)
