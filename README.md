# + Design docker swarm clusters on Amazon Web Services

    - Getting Started with Amazon EC2 :

    $ login aws

    go to https://console.aws.amazon.com/ec2/v2/home?region=us-east-1
    - launch an instance
    - go to marketplace
    - Next, choose an Amazon Machine Image (AMI)
    - choose Amazon ECS-Optimized Amazon Linux 2 AMI
    - choose t2.micro
    - Next. In Configure Instance Details, specify the number of instances as 1.
    - Select a network or click on Create New VPC to `create a new VPC`.
    - Select a subnet or click on Create New Subnet to create a new subnet.
    - Select Enable for Auto-Assign Public IP. Click on Next.
    - From Add Storage, select the default settings and click on Next
    - In Add Tags, no tags need to be added. Click on Next.
    - From Configure Security Group, add a security group to allow all traffic of any protocol in all port ranges from any source (0.0.0.0/0).
    - Click on Review and Launch and subsequently click on Launch.
    - Select a key pair and click on Launch Instances in the Select an Existing Key Pair or Create a New Key Pair dialog as `EC2` and click on download it.
    - save the key (EC2.pem)
    - click on see instance

    $ get IPv4 Public IP -> 34.203.194.177

    + SSH login into the EC2 instance as user “ec2-user”.
    $ chmod 400 EC2.pem # change permissions of EC2.pem to be read by the command-line
    $ ssh -i "EC2.pem" ec2-user@34.203.194.177

    # docker start
    $ docker run -d -p 8080:80 --name helloapp tutum/hello-world
    $ docker ps

    $ docker port helloapp # get port of a container
    $ curl localhost:8080

    # check Public DNS (IPv4)
    $ http://ec2-34-203-194-177.compute-1.amazonaws.com:8080/

    $ docker stop helloapp
    $ docker rm helloapp
    4 docker images



    + This section covers the following topics:

        • Setting the environment
        • Initializing the Docker Swarm mode
        • Joining nodes to the Swarm cluster
        • Testing the Swarm cluster
        • Promoting a worker node to manager
        • Demoting a manager node to worker
        • Making a worker node leave the Swarm cluster
        • Making A worker node rejoin the Swarm cluster
        • Making a manager node leave the Swarm cluster
        • Reinitializing a Swarm
        • Modifying node availability
        • Removing a node


