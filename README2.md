Certainly! Below is the documentation formatted for a GitHub repository README.md file:

---

# Asterisk Server Setup and Integration with Jenkins and AWS RDS

This repository provides documentation on setting up an Asterisk server on an Amazon EC2 instance, integrating it with Jenkins, and configuring an AWS RDS database with AWS Secrets Manager.

## 1. Setting up Asterisk Server on EC2 Instance

### 1.1. Launch an EC2 instance

Launch an EC2 instance with Amazon Linux 2 as the operating system.

```bash
# Commands to install and configure Asterisk on EC2 instance
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-20-current.tar.gz
tar -xvf asterisk-20-current.tar.gz
sudo yum install --skip-broken --assumeyes gcc make gcc-c++ pkgconfig libedit-devel jansson-devel libuuid-devel sqlite-devel libxml2-devel speex-devel speexdsp-devel libogg-devel libvorbis-devel alsa-lib-devel portaudio-devel libcurl-devel xmlstarlet bison flex postsql-dev postgresql-dev postgresql-devel unixODBC-devel neon-devel gmime-devel lua-devel uriparser-devel libxslt-devel openssl-devel mysql-devel bluez-libs-devel radcli-devel freetds-devel jack-audio-connection-kit jack-audio-connection-kit-devel bash libcap-devel net-snmp-devel iksemel-devel corosynclib-devel newt-devel popt-devel libical-devel spandsp-devel libresample-devel uw-imap-devel binutils-devel libsrtp-devel gsm-devel doxygen graphviz zlib-devel openldap-devel codec2-devel hoard fftw-devel unbound-devel wget subversion bzip2 patch xmlstarlet libjansson jansson-devel
cd asterisk-20.4.0
sudo su
./configure --prefix=/apps/asterisk --with-pjproject-bundled --with-jansson-bundled
make menuselect
make
make install
make basic-pbx
make install-logrotate
tar -cvf /apps/asterisk.tar /apps/asterisk/
/apps/asterisk/sbin/asterisk -vvvvvvvgc
```

*Note: Adjust configurations as per your requirements.*

## 2. Jenkins Setup

### 2.1. Install Jenkins

Install Jenkins on a server.

### 2.2. Install AWS CLI

Install the AWS CLI on the Jenkins server.

## 3. RDS Database Configuration

### 3.1. Create an RDS MySQL database

Create an RDS MySQL database on AWS.

### 3.2. Configure RDS

Configure the RDS database for public access, enable automatic backups, and set a temporary password.

### 3.3. Security Group

Ensure the security group associated with the RDS instance allows inbound traffic on port 3306.

### 3.4. Verify Connectivity

Verify RDS connectivity using MySQL Workbench with the temporary password.

## 4. AWS Secrets Manager Integration

### 4.1. Navigate to Secrets Manager

Navigate to AWS Secrets Manager.

### 4.2. Store a New Password

Store a new password for the RDS database with a name, username, and the temporary password.

### 4.3. Configure Rotation

Configure automatic rotation in Secrets Manager.

### 4.4. Modify RDS Database

Modify the RDS database to manage master credentials in AWS Secrets Manager.

### 4.5. Retrieve Password

Retrieve the updated password from Secrets Manager and verify connectivity using MySQL Workbench.

## 5. IAM User Configuration

### 5.1. Create a New User

In AWS IAM, create a new user for Jenkins with full permissions to Secrets Manager.

### 5.2. Attach Policies

Attach the required policies directly to the user, ensuring it has all the necessary permissions.

### 5.3. Download Access Keys

Download the access keys for the newly created IAM user.

## 6. Jenkins AWS Credentials Configuration

### 6.1. Install AWS Credentials Plugin

Install the AWS Credentials Plugin in Jenkins.

### 6.2. Add Global Credential

In Jenkins Credentials, add a new global credential of type "AWS Credentials".

### 6.3. Enter Access Keys

Enter the AWS access key and secret access key obtained from the IAM user.

## 7. Jenkins Pipeline

### 7.1. Create Jenkins Pipeline

Create a Jenkins pipeline with the provided script in the `Jenkinsfile`.

```groovy
pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
    }

    stages {
        stage('Get RDS Password from Secrets Manager') {
            steps {
                script {
                    // Retrieve AWS credentials from Jenkins Credentials
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_Creds', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        
                        // Retrieve RDS DB password from Secrets Manager
                        def rdsDbPassword = sh(script: """
                            aws secretsmanager get-secret-value \
                                --secret-id your-rds-db-secret-name \
                                --region \${AWS_REGION} \
                                --query 'SecretString' \
                                --output text
                        """, returnStdout: true).trim()

                        echo "RDS DB Password: ${rdsDbPassword}"
                    }
                }
            }
        }
    }
}
```

### 7.2. Run Jenkins Pipeline

Replace 'your-rds-db-secret-name' with the actual name of your RDS DB secret. Run the Jenkins pipeline to retrieve the RDS DB password.

---

*Note: Ensure to customize configurations, replace placeholders, and adapt the steps as per your specific requirements and environment. This documentation assumes basic familiarity with AWS services, Jenkins, and MySQL.*
