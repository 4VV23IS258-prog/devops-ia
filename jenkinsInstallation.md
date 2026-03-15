# Jenkins Installation Guide

1.  **Jenkins Installation**: Begin the setup process for the Jenkins automation server.
2.  **Download Installer**: Download the .msi installer from the official site: [www.jenkins.io/download/](https://www.jenkins.io).
3.  **Destination Folder**: Configure the installer by choosing the destination folder where Jenkins will be installed.
4.  **Service Logon**: Select the logon type **"Run service as LocalSystem"** and click next.
5.  **Port Selection**: Test an available port and assign it to Jenkins to host its service through the port.
6.  **Java Path**: Choose the path where the Java Development Kit (JDK) is installed. (Note: Jenkins supports JDK 17, 21, and 25).
7.  **Custom Setup**: Choose how features are handled from the tree, select to start the service after installation, and enable a firewall exception for Jenkins running on the assigned port.
8.  **Install**: Click **"Install"** to begin the installation process.
9.  **Initial Unlock**: In a web browser, go to `localhost:assigned_port_for_jenkins`. It will redirect to the login page. The initial administrator credentials are located at: `C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword`.
10. **Install Git Plugin**: Navigate to **Manage Jenkins** > **Plugins** > **Available plugins**, search for **"Git"** (verified by "Installed on 96% of controllers"), and install it.
