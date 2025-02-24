# WireGuard + Immich in Docker 

This repository provides a comprehensive guide for deploying **WireGuard** and **Immich** using Docker, ensuring a secure and efficient environment. **WireGuard** is utilized to establish a high-performance VPN, enhancing secure remote access, while **Immich** serves as an advanced AI-powered media management platform. The documentation covers the step-by-step configuration of essential components, including **PostgreSQL**, **Redis**, and **Immich's machine learning services**, ensuring a fully optimized and containerized deployment for seamless operation.
![Immich Photo](https://i.ibb.co/zWmnJxx3/docker.png)
# What is WireGuard?

WireGuard is a next-generation **VPN (Virtual Private Network)** protocol designed for **simplicity, speed, and security**. Unlike traditional VPN solutions, WireGuard utilizes **state-of-the-art cryptographic principles** to provide a lightweight yet highly secure networking solution. It is engineered to be **faster, more efficient, and easier to configure** than legacy VPN protocols like OpenVPN or IPSec.

Built directly into the Linux kernel and available across multiple platforms, WireGuard is ideal for securing network traffic, enabling **remote access, site-to-site connectivity, and privacy-focused networking**. Its streamlined codebase enhances **performance and auditability**, making it a preferred choice for both individuals and enterprises.

For more information, visit the official website: [**https://www.wireguard.com/**](https://www.wireguard.com/)

![Immich Photo](https://upload.wikimedia.org/wikipedia/commons/9/98/Logo_of_WireGuard.svg)
# What is Immich?

Immich is an advanced, self-hosted media management platform designed for efficient photo and video backup, retrieval, and organization. Originally conceived as a personal project to provide a seamless alternative to proprietary cloud-based solutions, Immich prioritizes **performance, ease of use, and privacy**. Unlike traditional gallery applications, Immich is engineered as a high-performance backup tool with a **native mobile application**, enabling users to effortlessly upload, manage, and view their media. By leveraging **modern technologies** and a **containerized architecture**, it ensures scalability and adaptability to diverse hosting environments.

For more information I recommend visiting the official website: [**https://immich.app/**](https://immich.app/)

![Immich Photo](https://immich.app/img/immich-logo-inline-light.png)



# Step by Step

This guide will walk you through setting up **WireGuard** and **Immich** using **Docker** on a **Linux** environment. Linux is recommended for its compatibility and ease of use.

If you are using **Windows**, it is highly recommended to use **Windows Subsystem for Linux (WSL)** to follow these steps efficiently. You can find the official installation guide for WSL here:

🔗 **[Install WSL on Windows](https://learn.microsoft.com/en-us/windows/wsl/install)**

Now, let's begin the setup process. 

## Preparing environment and Installing Docker

 **- Update and Upgrade system:**

    sudo apt update && sudo apt upgrade -y
   
 **- Install Docker and Docker Compose:**

> Add Docker's official GPG key:

    sudo apt install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    

> Add the repository to Apt sources:

    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update

> Install the lastest version of docker:

     sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
     
> Verify that the installation is successful

     sudo docker run hello-world

> Create a new directory:

     mkdir test && cd test/

> Copy the Repository

     git clone https://github.com/AitorConS/WireGuard-Immich.git 

## Configure WireGuard and Immich
Before deploying the containers, we will configure two essential files: **`docker-compose.yml`** and **`.env`** (for Immich). The **`docker-compose.yml`** file defines the structure of our deployment, specifying the services, networks, and volumes for **WireGuard, Immich, PostgreSQL, and Redis**. The **`.env`** file allows us to configure Immich's environment variables, such as database credentials, storage paths, and optional settings, making it easier to customize the deployment without modifying the compose file directly. Properly setting up these files ensures a smooth and flexible installation process.

> **Note:** On **Linux**, the `.env` file is a **hidden file** due to the leading dot (`.`) in its name. To view it in the terminal, use: ***ls -la***

**.ENV (Configuration of Immich):**

    # The location where your uploaded files are stored
    UPLOAD_LOCATION=./library
    # The location where your database files are stored
    DB_DATA_LOCATION=./postgres
    
    # To set a timezone, uncomment the next line and change Etc/UTC to a TZ identifier from this list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
    # TZ=Etc/UTC
    
    # The Immich version to use. You can pin this to a specific version like "v1.71.0"
    IMMICH_VERSION=release
    
    # Connection secret for postgres. You should change it to a random password
    # Please use only the characters `A-Za-z0-9`, without special characters or spaces
    DB_PASSWORD=postgres
    
    # The values below this line do not need to be changed
    ###################################################################################
    DB_USERNAME=postgres
    DB_DATABASE_NAME=immich

**Docker-Compose.yml (For Plug And Play):**

    wireguard:
        image: linuxserver/wireguard
        container_name: wireguard
        cap_add:
	        - NET_ADMIN
	        - SYS_MODULE
	    environment:
		    - PUID=1000
	        - PGID=1000
	        - TZ=America/Mendoza
	        - SERVERURL=34.17.88.161  # Change with your external Server IP
	        - SERVERPORT=51820  # You can change that to another port but not recommendable
	        - PEERS=2  # Number of devices  
	        - PEERDNS=auto  
	        - INTERNAL_SUBNET=10.13.13.0  #optional but I don't recommend to change it
        volumes:
	        - ./wireguard:/config # Config path of Wireguard
	        - /lib/modules:/lib/modules
	        - /usr/src:/usr/src
        ports:
	        - 51820:51820/udp
        sysctls:
	        - net.ipv4.conf.all.src_valid_mark=1
        networks:
	        - my_custom_network
        restart: unless-stopped
# Start Services
Once all services have been properly configured in the **`docker-compose.yml`** and **`.env`** files, we can proceed to launch the containers. To do this, we will use **Docker Compose**, which will automatically set up and manage all the defined services.

Run the following command in the project directory to start the deployment:

    sudo docker compose -d
If everything has been set up correctly, you will find the **configuration files** of the VPN and **QR codes** inside the `wireguard/` directory. Each peer’s configuration will be located in its respective subdirectory within `wireguard/peers/`.

These files contain the necessary details for connecting clients to the WireGuard VPN. You can use the **QR codes** to easily configure the VPN on mobile devices by scanning them with the WireGuard app u can see them executing the command: 

    sudo docker logs wireguard 

To access **Immich**, you must first **connect to the WireGuard VPN**. If you are unsure how to do this, you can follow this tutorial:

🔗 **[How to Connect to a WireGuard VPN](https://www.tp-link.com/latam/support/faq/3989/)**

Before connecting, you need to determine the **IP address of the Immich container**. Run the following command to list the container names along with their assigned IPs:

    docker ps -q | xargs docker inspect -f '{{.Name}} - {{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}} 

Once you have identified the **Immich container's IP**, open your browser **after activating the VPN** and enter:

    http://<immich-container-ip>:2283 

This will allow you to access the **Immich web interface** and begin using the platform.

⚠️ **Important Notice:** The **first user** you create in Immich will be assigned **Administrator privileges**. Make sure to **remember the password**, as this account will have full control over the platform's settings and user management.




