# workrelationship2024



So to recap what was discussed.

 

If you run the command

 

git-remote-codecommit

 

and get an error that the file is not found, you will need to delete the file appdata\local\first_login.txt then log off and on the Workspace. This will re-run a logon script that configures your Workspace and will add the path to the file to you PATH variable.

 

To configure SSO, you need to run the command aws configure sso and complete the questions.

 

SSO Session Name: saas

SSO Start URL:  https://d-9c670aab85.awsapps.com/start/#/?tab=accounts

SSO region: eu-west-2

SSO registration scopes: sso:account:access

 

Once that is completed you can replace the file d:\Users\%username%\.aws\config with the one stored at \\iswinprowinmgmt\software$\config

 

 

Then you can login with the command

 

aws sso login --profile=ce-sandpit

 

I have modified the file so that if you were to use the command

 

aws s3 ls --profile ce-com

 

It will use the ReadOnly permissions. Once you have been granted admin rights you can then use the following profiles to access using admin rights.

 

Aws s3 ls –profile ce-com-admin

 

Start in the ce-sandpit account. When you start using git to work with you CodeCommit repository, type the following command to set your default account

 

    #!/bin/bash
    #switch to rootuser
    sudo -i
    # Update and install dependencies
    sudo yum update -y
    sleep 10
    sudo yum install -y unzip
    sleep 5
    # Download AWS CLI v2
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip"
    sleep 5
    unzip /tmp/awscliv2.zip -d /tmp
    sleep 5 
    sudo /tmp/aws/install
    sleep 5
    # Clean up installation files
    rm -rf /tmp/awscliv2.zip /tmp/aws
    sleep 5  # Wait for 5 seconds after cleanup
 # Create destination directory
    sudo mkdir -p /opt/modelizeIT
    sudo mkdir -p /tmp/modelizeit
    sudo chmod -R 777 /tmp/modelizeit

    # Download the file from S3 using aws cli
    aws s3 cp s3://sas-sandbox-staging/ModelizeIT/ /tmp/modelizeit --recursive

    # Extract the file
    sudo tar -xvzf /tmp/modelizeit/apache-maven-3.9.9-bin.tar.gz -C /opt/modelizeIT

    # Verrifiy this code

    user_data = <<-EOF
              #!/bin/bash

              DEST_DIR="/opt/modelizeIT"
      
              sudo yum update -y
              sleep 5
              sudo yum install -y unzip
              sleep 5
             
              curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip"
              sleep 5
              unzip /tmp/awscliv2.zip -d /tmp
              sleep 5 
              sudo /tmp/aws/install
              
             
              sudo rm -rf /tmp/awscliv2.zip /tmp/aws
              sleep 10 

              sudo chmod -R 777 /tmp/modelizeit
              sleep 10

              aws s3 cp s3://saas-sandbox-staging/ModelizeIT/modelizeIT-AnalysisServer.tgz /tmp/modelizeit/
              sleep 10
              sudo tar -xvzf /tmp/modelizeit/modelizeIT-AnalysisServer.tgz -C /opt/
              sleep 10

    
              cd $DEST_DIR/bin
              chmod +x RejuvenApptor-start.sh modelizeIT-start.sh Gatherer-UI.sh Gatherer-JobRunner.sh
              sh RejuvenApptor-start.sh
              sh modelizeIT-start.sh
              sh Gatherer-JobRunner.sh
              sh Gatherer-UI.sh
             
              rm -f /tmp/modelizeIT-AnalysisServer.tgz
              EOF


#Updated Version of the above code 

#!/bin/bash

DEST_DIR="/opt/modelizeIT"

sudo yum update -y
sudo yum install -y unzip

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip"
unzip /tmp/awscliv2.zip -d /tmp
sudo /tmp/aws/install

sudo rm -rf /tmp/awscliv2.zip /tmp/aws

sudo chmod -R 777 /tmp/modelizeit

aws s3 cp s3://saas-sandbox-staging/ModelizeIT/modelizeIT-AnalysisServer.tgz /tmp/modelizeit/
sudo tar -xvzf /tmp/modelizeit/modelizeIT-AnalysisServer.tgz -C /opt/

cd $DEST_DIR/bin
sudo chmod +x RejuvenApptor-start.sh modelizeIT-start.sh Gatherer-UI.sh Gatherer-JobRunner.sh
sudo sh $DEST_DIR/bin/RejuvenApptor-start.sh
sudo sh $DEST_DIR/bin/modelizeIT-start.sh
sudo sh $DEST_DIR/bin/Gatherer-JobRunner.sh
sudo sh $DEST_DIR/bin/Gatherer-UI.sh

sudo rm -f /tmp/modelizeIT-AnalysisServer.tgz

#This is for the powershell installation of modelizeit 


resource "aws_instance" "windows_instance" {
  ami                         = "ami-0c55b159cbfafe1f0"  # Replace with the desired Windows Server AMI ID
  instance_type               = var.instance_type
  subnet_id                   = var.subnet_id
  key_name                    = var.key_pair_name
  vpc_security_group_ids      = [var.security_group_id]
  associate_public_ip_address = true

  user_data = <<-EOF
    <powershell>
    # Set execution policy to allow script running
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine -Force

    # Define the destination directory
    $DEST_DIR = "C:\\modelizeIT"

    # Create the directory if it doesn't exist
    if (-Not (Test-Path -Path $DEST_DIR)) {
        New-Item -Path $DEST_DIR -ItemType Directory
    }

    # Install Chocolatey package manager
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

    # Install 7-Zip using Chocolatey
    choco install 7zip -y

    # Download and install AWS CLI
    $InstallerPath = "$env:TEMP\\AWSCLIV2.msi"
    Invoke-WebRequest -Uri "https://awscli.amazonaws.com/AWSCLIV2.msi" -OutFile $InstallerPath
    Start-Process msiexec.exe -ArgumentList "/i $InstallerPath /quiet" -Wait
    Remove-Item -Path $InstallerPath

    # Configure AWS CLI (ensure that AWS credentials are available)
    # aws configure set aws_access_key_id YOUR_ACCESS_KEY_ID
    # aws configure set aws_secret_access_key YOUR_SECRET_ACCESS_KEY
    # aws configure set default.region YOUR_REGION

    # Download the application package from S3
    $TempFilePath = "$env:TEMP\\modelizeIT-AnalysisServer.zip"
    aws s3 cp "s3://saas-sandbox-staging/ModelizeIT/modelizeIT-AnalysisServer.zip" $TempFilePath

    # Extract the application package
    & "C:\\Program Files\\7-Zip\\7z.exe" x $TempFilePath -o$DEST_DIR

    # Remove the temporary file
    Remove-Item -Path $TempFilePath

    # Navigate to the application's bin directory
    Set-Location -Path "$DEST_DIR\\bin"

    # Execute the startup scripts
    .\\RejuvenApptor-start.ps1
    .\\modelizeIT-start.ps1
    .\\Gatherer-JobRunner.ps1
    .\\Gatherer-UI.ps1

    # Enable RDP access
    Set-ItemProperty -Path 'HKLM:\\System\\CurrentControlSet\\Control\\Terminal Server' -Name 'fDenyTSConnections' -Value 0

    # Allow RDP through Windows Firewall
    Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

    # Set the Administrator password (ensure to replace 'YourSecurePassword' with a strong password)
    $password = ConvertTo-SecureString "YourSecurePassword" -AsPlainText -Force
    Set-LocalUser -Name "Administrator" -Password $password
    </powershell>
  EOF

  tags = {
    Name = "WindowsModelizeITInstance"
  }
}

#the steps to push code on code commit
Create a repo make sure add a readme.md file 
clone the GRC https
and run git clone .................... make sure you add the profile within the coppied GRC url
git clone codecommit::eu-west-2://ce-sandbox-admin@saas-sandbox-terraform-two
   38  git branch
   39  ls
   40  cd saas-sandbox-terraform-two/
   41  git branch
   42  code .
   43  pwd
   44  cd ../
   45  git clone codecommit::eu-west-2://ce-sandbox-admin@Saas-terraform-code
   46  cd Saas-terraform-code/
   47  ls
   48  code .
   49  ls
   50  ls
   51  ls
   52  code .
   53  history



   Ansible

   sudo apt update && sudo apt install -y ansible
[modelizeIT]
target_server ansible_host=192.168.1.10 ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/id_rsa

---
- name: Deploy ModelizeIT Application
  hosts: modelizeIT
  become: yes
  tasks:

    - name: Update system packages
      yum:
        name: "*"
        state: latest

    - name: Install unzip utility
      yum:
        name: unzip
        state: present

    - name: Download AWS CLI v2
      get_url:
        url: "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip"
        dest: "/tmp/awscliv2.zip"

    - name: Extract AWS CLI installer
      unarchive:
        src: /tmp/awscliv2.zip
        dest: /tmp/
        remote_src: yes

    - name: Install AWS CLI
      command: "/tmp/aws/install"

    - name: Clean up AWS CLI installation files
      file:
        path: "/tmp/awscliv2.zip"
        state: absent

    - name: Set permissions for the working directory
      file:
        path: /tmp/modelizeit
        state: directory
        mode: "0777"

    - name: Copy application package from S3
      aws_s3:
        bucket: saas-sandbox-staging
        object: ModelizeIT/modelizeIT-AnalysisServer.tgz
        dest: /tmp/modelizeit/modelizeIT-AnalysisServer.tgz
        mode: get

    - name: Extract the application package
      unarchive:
        src: /tmp/modelizeit/modelizeIT-AnalysisServer.tgz
        dest: /opt/
        remote_src: yes

    - name: Make startup scripts executable
      file:
        path: "{{ item }}"
        mode: "0755"
      loop:
        - /opt/modelizeIT/bin/RejuvenApptor-start.sh
        - /opt/modelizeIT/bin/modelizeIT-start.sh
        - /opt/modelizeIT/bin/Gatherer-UI.sh
        - /opt/modelizeIT/bin/Gatherer-JobRunner.sh

    - name: Execute startup scripts
      command: "sh {{ item }}"
      loop:
        - /opt/modelizeIT/bin/RejuvenApptor-start.sh
        - /opt/modelizeIT/bin/modelizeIT-start.sh
        - /opt/modelizeIT/bin/Gatherer-UI.sh
        - /opt/modelizeIT/bin/Gatherer-JobRunner.sh

    - name: Remove the application package
      file:
        path: /tmp/modelizeit/modelizeIT-AnalysisServer.tgz
        state: absent

      Run i

ansible-playbook -i inventory deploy_modelizeIT.yml




updated with the host there 

---
- name: Deploy ModelizeIT Application
  hosts: all
  become: yes
  vars:
    ansible_host: 192.168.1.10
    ansible_user: ec2-user
    ansible_ssh_private_key_file: ~/.ssh/id_rsa

  tasks:
    - name: Update system packages
      yum:
        name: "*"
        state: latest

    - name: Install unzip utility
      yum:
        name: unzip
        state: present

    - name: Download AWS CLI v2
      get_url:
        url: "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip"
        dest: "/tmp/awscliv2.zip"

    - name: Extract AWS CLI installer
      unarchive:
        src: /tmp/awscliv2.zip
        dest: /tmp/
        remote_src: yes

    - name: Install AWS CLI
      command: "/tmp/aws/install"

    - name: Clean up AWS CLI installation files
      file:
        path: "/tmp/awscliv2.zip"
        state: absent

    - name: Set permissions for the working directory
      file:
        path: /tmp/modelizeit
        state: directory
        mode: "0777"

    - name: Copy application package from S3
      aws_s3:
        bucket: saas-sandbox-staging
        object: ModelizeIT/modelizeIT-AnalysisServer.tgz
        dest: /tmp/modelizeit/modelizeIT-AnalysisServer.tgz
        mode: get

    - name: Extract the application package
      unarchive:
        src: /tmp/modelizeit/modelizeIT-AnalysisServer.tgz
        dest: /opt/
        remote_src: yes

    - name: Make startup scripts executable
      file:
        path: "{{ item }}"
        mode: "0755"
      loop:
        - /opt/modelizeIT/bin/RejuvenApptor-start.sh
        - /opt/modelizeIT/bin/modelizeIT-start.sh
        - /opt/modelizeIT/bin/Gatherer-UI.sh
        - /opt/modelizeIT/bin/Gatherer-JobRunner.sh

    - name: Execute startup scripts
      command: "sh {{ item }}"
      loop:
        - /opt/modelizeIT/bin/RejuvenApptor-start.sh
        - /opt/modelizeIT/bin/modelizeIT-start.sh
        - /opt/modelizeIT/bin/Gatherer-UI.sh
        - /opt/modelizeIT/bin/Gatherer-JobRunner.sh

    - name: Remove the application package
      file:
        path: /tmp/modelizeit/modelizeIT-AnalysisServer.tgz
        state: absent

##################################################################################################################################################################################################################################################################################
############Manual Powershell command to gather some information on windows servers##############
# Define output file
$outputFile = "C:\ServerInfo.txt"

# Collect Disk Usage Information
"Disk Usage Information:" | Out-File -FilePath $outputFile
Get-PSDrive -PSProvider FileSystem | Select-Object Name, @{Name="Used(GB)";Expression={[math]::round(($_.Used/1GB),2)}}, @{Name="Free(GB)";Expression={[math]::round(($_.Free/1GB),2)}}, @{Name="Total(GB)";Expression={[math]::round(($_.Used/1GB) + ($_.Free/1GB),2)}} | Format-Table | Out-File -FilePath $outputFile -Append

# Collect System Information
"`nSystem Information:" | Out-File -FilePath $outputFile -Append
Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object Caption, Version, BuildNumber, OSArchitecture | Format-List | Out-File -FilePath $outputFile -Append

# Collect Domain Information
"`nDomain Information:" | Out-File -FilePath $outputFile -Append
(Get-WmiObject Win32_ComputerSystem).Domain | Out-File -FilePath $outputFile -Append

# Collect Swap Space Information
"`nSwap Space Information:" | Out-File -FilePath $outputFile -Append
Get-CimInstance -ClassName Win32_PageFileUsage | Select-Object Name, AllocatedBaseSize, CurrentUsage, PeakUsage | Format-Table | Out-File -FilePath $outputFile -Append

# Collect Installed Packages Information
"`nInstalled Packages Information:" | Out-File -FilePath $outputFile -Append
Get-WmiObject -Class Win32_Product | Select-Object Name, Version, Vendor | Format-Table | Out-File -FilePath $outputFile -Append

# Collect Network Configuration
"`nNetwork Configuration:" | Out-File -FilePath $outputFile -Append
Get-NetIPConfiguration | Format-List | Out-File -FilePath $outputFile -Append

# Collect Hardware Configuration
"`nHardware Configuration:" | Out-File -FilePath $outputFile -Append
Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object Manufacturer, Model, TotalPhysicalMemory | Format-List | Out-File -FilePath $outputFile -Append

# Collect Processor Information
"`nProcessor Information:" | Out-File -FilePath $outputFile -Append
Get-CimInstance
::contentReference[oaicite:0]{index=0}


##########################################################################################################################################################################################################
# This is to gather information using CMD commands on windows severs 
Instructions:

Create the Script File:

Open Notepad or any text editor.
Copy and paste the above script into the editor.
Save the file with a .bat extension, for example, CollectServerInfo.bat.
Run the Script:

Right-click on the saved .bat file and select "Run as administrator" to ensure it has the necessary permissions.
The script will execute each command and append the output to C:\ServerInfo.txt.
Review the Output:

Navigate to C:\ServerInfo.txt to review the collected system information.
Note: Ensure you have the necessary administrative privileges to execute these commands and access the required system information.

By utilizing these commands and the batch script, you can effectively gather comprehensive system information on your Windows server, aiding in the migration process.

@echo off
set outputFile=C:\ServerInfo.txt

echo Disk Usage Information: > %outputFile%
wmic logicaldisk get name,filesystem,freespace,size >> %outputFile%

echo. >> %outputFile%
echo System Information: >> %outputFile%
systeminfo >> %outputFile%

echo. >> %outputFile%
echo Domain Information: >> %outputFile%
echo %USERDOMAIN% >> %outputFile%

echo. >> %outputFile%
echo Swap Space Information: >> %outputFile%
wmic pagefile list /format:list >> %outputFile%

echo. >> %outputFile%
echo Installed Packages Information: >> %outputFile%
wmic product get name,version,vendor >> %outputFile%

echo. >> %outputFile%
echo Network Configuration: >> %outputFile%
ipconfig /all >> %outputFile%

echo. >> %outputFile%
echo Hardware Configuration: >> %outputFile%
wmic computersystem get manufacturer,model,totalphysicalmemory >> %outputFile%

echo. >> %outputFile%
echo Processor Information: >> %outputFile%
wmic cpu get name,numberofcores,numberoflogicalprocessors >> %outputFile%

echo. >> %outputFile%
echo Information collection complete. Output saved to %outputFile%


 


 
