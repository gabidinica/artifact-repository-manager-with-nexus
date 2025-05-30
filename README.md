# Install and Configure Nexus on a Cloud Server

This guide walks you through installing and configuring **Nexus Repository Manager** from scratch on a cloud server using **DigitalOcean**.

> **Prerequisites:**  
Nexus requires a server with higher memory allocation. It is recommended to use a droplet with **at least 8GB of RAM and 4 CPUs**.

---

## 🛠️ Prerequisites

- A DigitalOcean account: [https://cloud.digitalocean.com](https://cloud.digitalocean.com)
- An SSH key added to GitHub or GitLab, which will be used to access your droplet
- Terminal access (macOS/Linux shell or Windows PowerShell/Git Bash)

## 🧱 Step-by-Step Setup

### 1. Create a DigitalOcean Droplet and secure the server with Firewall

1. Go to [DigitalOcean → Droplets](https://cloud.digitalocean.com/droplets) → **Create Droplet**
2. Choose:
   - **Image**: A Linux distribution (e.g., Ubuntu)
   - **Plan**: Basic → Regular → **8GB / 4 CPUs**
   - **Region**: Choose your closest location
   - **Authentication**: Use your existing **SSH key**
3. After the droplet is created, click on its name.
4. Go to **Networking → Firewalls → Create Firewall**
5. Add:
   - **Name**: e.g., `nexus-firewall`
   - **Inbound Rule**:
     - **Type**: SSH
     - **Port**: 22
     - **Source**: Remove all, and **add your own IP address only**
6. Click **Create Firewall**
7. Assign the firewall to your droplet under the **Droplets tab**
8. To connect to the droplet from your computer copy the IP address and into your terminal type: ssh root@IP_ADDRESS

⚙️ Initial Server Setup
 1. Update the server packages: apt update
 2. Check if Java is installed: java
 3. To be compatible with our project, it's needed the installation of Java 17 so copy the option: apt install openjdk-17-jdk -y
 4. Confirm the installation: java -version

📦 Install Nexus Repository Manager
 1. Navigate to the /opt directory: cd /opt
 2. Go to your browser and search for ‘Nexus download’ and open the link from Sonatype > copy the address from Unix archive. Use wget to download: 
   - wget https://download.sonatype.com/nexus/3/nexus-3.80.0-01-unix.tar.gz
 3. Check the downloaded file: ls
 4. Extract the Nexus archive: tar -zxvf YOUR_NEXUS_TAR_NAME

-----

👤 Create new User on Nexus with relevant permissions

 
## We’re gonna create a new user which is gonna be used by Nexus service so we won’t use the root user for it.

 1. In the terminal, in the same folder /opt, add a new user: adduser nexus
   - set a password
   - Fill in the **Full Name** as **nexus**
   - Press Enter to skip other fields
 2. Check permissions for Nexus and Sonatype folders: ls -l
 3. Change ownership of the Nexus and Sonatype folders:
   > chown -R nexus:nexus nexus-3.80.0-01
   > chown -R nexus:nexus sonatype-work
 4. Verify changes: ls -l
 5. We need to set the Nexus configuration to run as the nexus user. To do this, edit the Nexus config file: **vim NEXUS_FOLDER_NAME/bin/nexus.rc**
 6. Inside the file, set: run_as_user="nexus"
 7. Now switch to nexus user to run the service: **su - nexus**
 8. Start the Nexus service: **nexus/NEXUS_FOLDER_NAME/bin/nexus start**
 9. Verify Nexus is running:
   > ps aux | grep nexus
   > netstat -lntp
   - You’ll see that Nexus is running on port 8081 so we if want to access Nexus from browser, we need to allow the firewall to allow inbound through this port.
 10. Go to your DigitalOcean Firewall settings and add an Inbound rule:
     - **Type**: Custom TCP
     - **Port**: 8081
     - **Source**: All Sources
     - Save the firewall settings
 11. ✅ Access Nexus in Your Browser: Go to: http://YOUR_IP_ADDRESS:8081 and you should see the Nexus Repository Manager UI.
  
## 🔐 Access the Nexus Admin UI
  
  1. Click on "Sign in" (top-right corner)
  2. Use the following default credentials:
	- Username: admin
 	- Password: Found inside: cat /opt/sonatype-work/nexus3/admin.password
          and copt the displayed password and paste into the login page
  You now have full access to the Nexus Repository Manager UI.

------

📦 Java Gradle Project: Build JAR & Upload to Nexus

To allow your Gradle project to upload JARs to Nexus, you need a dedicated Nexus user with specific permissions.
 1. After you’ve logged in with the admin user, go to **Security - Users - Create a new user**.
 Fill in:
   - User ID: your-upload-user
   - First Name / Last Name: (any)
   - Email: (optional)
   - Password: set your secure password
   - Status: Active
   - Roles: assign nx-anonymous temporarily
   - Click Create user
 2.  Create a Custom Role
   - Go to Security → Roles → Create role
   - Role type: Nexus Role
   - Role ID: nexus-java
   - Role name: nexus-java
 Applied Privileges:
   - Click Modify applied privileges
   - Search and select: nx-repository-view-maven2-maven-snapshots-*    (This provides read/write access to Maven2 snapshot repositories.)
   - Click Save to create the role
 3. Assign the Role to the User
   - Go to Security → Users
   - Edit the user you created
   - Remove the nx-anonymous role
   - Assign the new nexus-java role
   - Save the changes
✅ Your user now has permission to upload artifacts to your Nexus Maven snapshot repository.

  Clone the project from your GitHub/GitLab repository and open it in IntelliJ IDEA (or your preferred IDE):
  -  git clone https://github.com/your-username/java-gradle-app.git
  -  cd java-gradle-app

  Open the project in IntelliJ IDEA and ensure the following line exists in your build.gradle file (it already should be present): **apply plugin: 'maven-publish'**
   In the same file, please check publishing section.  
   Publications is the jar file configuration that we’re gonna upload and repositories is the repo where we’re gonna upload the jar file.
   On the repositories, on URL line: you need to update with your [nexus id]:[nexus port]. You can copy from your browser. The jar file will be found in Nexus on **Repositories -> maven-snapshots** folder.
   1. Create a file in the root of your project named: **gradle.properties**
   2. Add the following:
    - repoUser=your-upload-username
    - repoPassword=your-upload-password
  Replace your-upload-username and your-upload-password with the credentials of the Nexus user you previously created.
  3. In your terminal, in the project folder, we need to build the project: gradle build
   In IntelliJ if you check there will be created a new **build** folder and if you expand it and go to libs, there will be the jar file.
  4. Type: ls and check that build folder is displayed.
  5. To upload the jar file execute: **gradle publish** 
✅ If everything is configured correctly, the JAR will be published to:
    http://<your-nexus-ip>:8081/#browse/browse:maven-snapshots
  
-------

 ☕ Java Maven Project: Build JAR & Upload to Nexus

 Clone your Maven project and open it in IntelliJ IDEA:
  -  git clone https://github.com/your-username/java-maven-app.git
  -  cd java-maven-app
 Open the project in IntelliJ.
  
  1. Update pom.xml for Deployment
     - Open pom.xml
     - Locate the <distributionManagement> section.
     - Update the URL to your Nexus server’s Maven snapshots repository: [nexus id]:[nexus port]
  
  2. Configure Maven Credentials
  - Navigate to your Maven config folder:
    - ls -a | grep .m2
    - ls .m2/
    - cd .m2
  - Create or edit settings.xml: vim settings.xml
  Add your Nexus credentials:
   <settings>
  <servers>
    <server>
      <id>nexus-snapshots</id>
      <username>your_username</username>
      <password>your_user_password</password>
    </server>
  </servers>
</settings>
  Replace your_username and your_user_password with the Nexus user you created earlier. Save and exit.
 
  3. Build and Deploy Your JAR
  From the project folder:
    - ls
    - mvn package
    - ls target
  You will see the generated JAR file inside target.
  To deploy (upload) the JAR to Nexus: **mvn deploy**
  
  4. Verify in Nexus
   Open your browser: http://<your-nexus-ip>:<port-id>
   Navigate to Browse (top menu).
   Select the maven-snapshots repository and confirm that your uploaded JAR appears there.
 



