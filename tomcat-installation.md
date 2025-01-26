# Tomcat Installation and Deployment on Amazon Linux EC2 Instance

## Steps for Setting Up Tomcat on EC2 Instance

### 1. Launch an EC2 Instance
1. Log in to your AWS Management Console.
2. Create a new EC2 instance:
   - Choose Amazon Linux as the operating system.
   - Select the instance type (e.g., t2.micro for free tier eligibility).
   - Configure instance details, storage, and security groups.
   - Ensure the security group allows access on port **8080** (Tomcat default port).
3. Launch the instance and connect to it via SSH.

### 2. Connect to the Server
1. Use SSH to connect to the EC2 instance:
   ```bash
   ssh -i <your-key-pair.pem> ec2-user@<public-ip-address>
   ```

### 3. Install Required Software
1. Update the system:
   ```bash
   sudo yum update -y
   ```
2. Install Java (required for Tomcat):
   ```bash
   sudo yum install java -y
   ```
3. Verify Java installation:
   ```bash
   java --version
   ```

### 4. Download and Install Tomcat
1. Go to the [Tomcat official website](https://tomcat.apache.org/) and copy the link for the latest version's **tar.gz** file.
2. Use the `wget` command to download the Tomcat package:
   ```bash
   wget <tomcat-tar-gz-link>
   ```
3. Extract the downloaded file:
   ```bash
   tar -xvf apache-tomcat-<version>.tar.gz
   ```
4. Navigate to the extracted directory:
   ```bash
   cd apache-tomcat-<version>
   ```

### 5. Start Tomcat
1. Go to the `bin` directory:
   ```bash
   cd bin
   ```
2. Start Tomcat:
   ```bash
   ./startup.sh
   ```
3. Check if Tomcat is running by visiting the public IP address of your EC2 instance in a browser with port **8080**:
   ```
   http://<public-ip>:8080
   ```
   You should see the Tomcat homepage.

### 6. Configure User Access for Manager and Host Manager
1. Navigate to the `conf` directory:
   ```bash
   cd ../conf
   ```
2. Open the `tomcat-users.xml` file using a text editor:
   ```bash
   sudo vim tomcat-users.xml
   ```
3. Add the following user roles and credentials:
   ```xml
   <role rolename="manager-gui"/>
   <role rolename="admin-gui"/>
   <user username="admin" password="password" roles="manager-gui,admin-gui"/>
   ```
4. Save and exit the file.
5. Restart Tomcat to apply changes:
   ```bash
   ./shutdown.sh
   ./startup.sh
   ```

### 7. Allow Remote Access to Manager App
1. Open the `context.xml` files for `manager` and `host-manager`:
   ```bash
   sudo vim ../webapps/manager/META-INF/context.xml
   sudo vim ../webapps/host-manager/META-INF/context.xml
   ```
2. Locate the following line and comment it out or modify it to allow all IPs:
   ```xml
   <!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
          allow="127\..*" /> -->
   ```
   Or modify it as needed to allow specific IP ranges.
3. Save and restart Tomcat.

### 8. Deploy Applications
1. Copy your `.war` or `.jar` file to the `webapps` directory:
   ```bash
   sudo cp <your-app>.war ../webapps/
   ```
2. The application will automatically be deployed, and you can access it using:
   ```
   http://<public-ip>:8080/<your-app>
   ```

## Automating Deployment Using Jenkins

### 1. Set Up Jenkins
1. Install Jenkins on the EC2 instance or another server.
2. Install the **Deploy to Container** plugin in Jenkins.

### 2. Configure Jenkins Project
1. Go to your Jenkins dashboard and create a new project.
2. In the project configuration:
   - Under **Post-Build Actions**, select **Deploy war/ear to a container**.
   - Provide the following details:
     - **WAR/EAR files**: Specify the path to the `.war` file (e.g., `**/*.war`).
     - **Context path**: Specify the application context path.
     - **Container**: Select **Tomcat 9.x**.
     - Provide the Tomcat server URL, username, and password.
3. Save the configuration.

### 3. Trigger Deployment
1. Build the project in Jenkins.
2. The `.war` file will be deployed to Tomcat automatically.
3. Verify the application on your browser:
   ```
   http://<public-ip>:8080/<context-path>
   ```

## Logs and Troubleshooting
- Tomcat logs are stored in the `logs` directory, specifically in the `catalina.out` file:
  ```bash
  cd ../logs
  tail -f catalina.out
  ```
- Use logs to debug deployment and application issues.

This guide covers both manual and automated deployment of applications on Tomcat. Ensure to secure your EC2 instance and Tomcat configuration to prevent unauthorized access.

