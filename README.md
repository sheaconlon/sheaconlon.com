# sheaconlon.com
## Deploy the website
If an option is not specified, then accept the default.
1. Go to AWS EC2.
2. Switch to region `US West`.
3. Create a VPC.
    1. Name it `sheaconlon.com-vpc`.
    2. Choose CIDR block `10.0.0.0/16`.
    3. Create a subnet.
        1. Name it `sheaconlon.com-only-subnet`.
        2. Put it in `sheaconlon.com-vpc`.
        3. Choose the CIDR block `10.0.0.0/16`.
4. Launch an instance.
    1. Choose the image `Ubuntu Server 16.04 LTS`.
    2. Choose the instance type `t2.nano`.
    3. Choose as the network `sheaconlon.com-vpc`.
    4. Choose as the subnet `sheaconlon.com-only-subnet`.
    5. Enable termination protection.
    6. Enable detailed monitoring.
    7. Add tag with key `Name` and value `sheaconlon.com-instance`.
    8. Create a new security group with name `sheaconlon.com-security-group`.
    9. Allow SSH, HTTP, and HTTPS from anywhere.
    10. Create a new key pair names `sheaconlon.com-keypair`. Do not lose or release the key file.
