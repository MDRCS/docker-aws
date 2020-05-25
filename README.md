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
    - save the key (swarm-cluster.pem)
    - click on see instance

    $ get IPv4 Public IP -> 3.235.222.47

    + SSH login into the EC2 instance as user “ec2-user”.
    $ chmod 400 swarm-cluster.pem # change permissions of swarm-cluster.pem to be read by the command-line
    $ ssh -i "swarm-cluster.pem" ec2-user@3.235.222.47

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


    - Tutorial : making a cluster of docker containers -> 1 manager and 2 nodes :

    # swarm mode commands :
![](./static/docker-swarm-command.png)

    Run the following command to initialize Docker Swarm mode.
    # check the private ip -> 172.31.66.147
    $ docker swarm init --advertise-addr 172.31.66.147:2377
    $ docker info
    $ docker info | grep Swarm # check weather is active or not.

    $ docker node ls # check how many nodes there is


# Manager Status :
![](./static/status.png)
![](./static/manager_status.png)

    -> The manager nodes are added for a different reason than the worker nodes. The manager nodes are added
       to make the Swarm more fault tolerant and the worker nodes are added to add capacity to the Swarm.

    The command to add a worker node may also be found using the following command.
    $ docker swarm join-token worker

    The command to add a manager node may be found using the following command.
    $ docker swarm join-token manager

    ++ A reason for adding a worker node is that the service tasks scheduled on some of the nodes are not running and are
       in Allocated state. A reason for adding a manager node is that another manager node has become unreachable.


    # Create EC2 Instances for 1 Manager & 2 Slaves
        Manager -> 3.235.222.47
        Slave-1 -> 3.235.78.101
        Slave-2 -> 3.235.224.114

### + before creating EC2 Instances, we should create a security group where we will allow certains ports :

![](./static/group_security.png)

    + Each time you create weather manager or slave choose this security group it's IMPORTANT!!.

    # Add Slaves to the swarm Cluster :
    # Manager :
    $ docker swarm join-token worker # command to get token to add slave nodes

    # Slave 1 :
    $ ssh -i "swarm-cluster.pem" ec2-user@3.235.78.101
    $ docker swarm join --token SWMTKN-1-0z9xm6tcpldjbpergpzkxzeb7oqzoqd9p7lejn1aswipzv4vn7-bmxtfbmd7rsfcuqeyljhbznrn 172.31.66.147:2377

    # Slave 2 :
    $ ssh -i "swarm-cluster.pem" ec2-user@3.235.224.114
    $ docker swarm join --token SWMTKN-1-0z9xm6tcpldjbpergpzkxzeb7oqzoqd9p7lejn1aswipzv4vn7-bmxtfbmd7rsfcuqeyljhbznrn 172.31.66.147:2377

    # Manager
    $ docker node ls

    # create a Docker service using the docker service create command, which becomes available only if the Swarm mode is enabled.
      Using Docker image alpine, which is a Linux distribution, create two replicas and ping the docker.com domain from the service containers.

    $ docker service create --replicas 2 --name helloworld alpine ping docker.com
    $ docker service ls
    $ docker service inspect helloworld
    $ docker service ps helloworld

    # check replicas on slaves nodes :

    # Slave-1
    $ docker ps
        CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
        70c80cb48db4        alpine:latest       "ping docker.com"   2 minutes ago       Up 2 minutes                            helloworld.2.79bmb8bvqwnfezaqnxasix000

    # Slave-2
    $ docker ps
        CONTAINER ID        IMAGE                            COMMAND             CREATED             STATUS                           PORTS               NAMES
        f0a29ddbfffa        alpine:latest                    "ping docker.com"   3 minutes ago       Up 3 minutes                                         helloworld.1.wwb4xosfd6y1t9wrsphwyc47a

    -> in the manager node you won't find any docker container.

    # Manager
    $ docker service rm helloworld
    $ docker service inspect helloworld
    []

    # Promoting a Worker Node to Manager
    # Manager
    $ docker node ls
    $ docker node promote ip-172-31-73-139.ec2.internal
    $ docker node ls

        v3vevlzef8jjrpnmgifj94cvg     ip-172-31-73-139.ec2.internal   Ready               Active              Reachable           19.03.6-ce

    # Demoting a Manager Node to Worker
    $ docker node demote ip-172-31-73-139.ec2.internal

    #making a Worker Node Leave the Swarm
    #SALVE-2
    $ docker swarm leave

    # Modifying Node Availability
    $ docker node update --availability drain ip-172-30-5-108.ec2.internal ip-172-30-5-108.ec2.internal

![](./static/status.png)

    # update Availability
    $ docker node update --availability drain ip-172-30-5-108.ec2.internal ip-172-30-5-108.ec2.internal

    # Remove a Node
    $ docker node rm ip-172-30-5-108.ec2.internal
