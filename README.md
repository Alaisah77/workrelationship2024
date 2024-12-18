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


 
