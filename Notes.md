Jenkins is the leading open source automation server.
Jenkins provides hundreds of plugins to support building, deploying and automating any project.

We will use docker to use Jenkins
Install docker and then install jenkins' using jenkins official image.
After installation,
  sudo systemctl start docker
  - to start docker

Now, we also want to ensure that docker is running when we turn on the 
machine.
  sudo systemctl enable docker
  - enable to turn on docker when the machine starts

Despite all this if we type:
  docker ps
  - lists all the running containers
  we get error because we are 'jenkins' user. So we type:
  sudo usermod -aG docker jenkins
  - adding 'jenkins' to the docker group so that we can use docker being 
    the jenkins user.

  We will still have error and to fix that, we have to logout and login 
  again.

Docker Compose: A utility of docker. Used to run multiple containers.
Docker-compose file: Get the command to install it for your particular OS 

Now, let us give executable permission to docker compose file:
  sudo chmod +x /usr/local/bin/docker-compose
  - give the executable permission
    if you notice while installing docker compose, the binaries have been 
    installed in /usr/local/bin/docker-compose
    
To test that docker-compose is installed, enter following command:
  docker-compose
  - lists all the options and details of the command.
  
We will install Jenkins using docker.
So, to install using the docker image:
  docker pull jenkins/jenkins
  - downloads the official jenkins image

  docker info | grep -i root
  Output: Docker Root Dir: /var/lib/docker
  - tells where where does docker saves the images

  docker du -sh /var/lib/docker
  - displays the space docker is taking

#Create a new directory to work on Jenkins:
  Create a sample docker compose file
  'vi docker-compose.yml'

  version:'3'
  services:
    jenkins:
      container_name: jenkins
      images: jenkins/jenkins
      port:
        - "8080:8080"
      volumes:
        - "$PWD/jenkins_home:/var/jenkins_home"
      networks:
        -  net:
  networks:
    net

    In the above docker-compose file, we have mapped a volume. To ensure 
    that the user who will write in this directory (jenkins) has proper permissions
    we will give permission. Jenkins is is 1000:
    sudo chown 1000:1000 jenkins_home -R

  To spin the service:
  docker-compose up -d
  - starts creating the container of the docker compose
  - note that docker compose doesn't create the image, it directly creates the container.

  To get logs of any container, we the command:
  docker logs -f
  - shows logs of all the containers
  docker logs -f jenkins
  - shows logs of jenkins containers by mentioning the name of the container.
  We will get to see a hash over here. This is the password to login in jenkins later.

  Jenkins is installed now. Lets open it in a browser. Use IP Address:
  IP_Address:8080

  Install suggested Plugin
  Enter Password from earlier

  Create user etc and Login

1st Job in Jenkins

  a. Create New - Give a name and create a 'Freestyle Project'
  b. Under 'Configure' we enter job body
  c. Under 'Build' write 'Hello World'
  d. Go to 'Console Output' to see the output
      You will see the job is finished successfully with 'Hello World'.

Playing with the Job

  a. Go to the job and click on 'Configure' section
  b. Date command in Linux:
      date
      - outputs the current date
    Now code:
    echo "Current date is $(date)"
    - outputs the above string with the actual date
    echo "Current user is $(whoami)"
    - self explainatory

Redirect your 1st job's output

  a. In bash - Born Against Shell
      To define Environmental Variables
      NAME = Zaman
      echo $Name
      Output: Zaman
      echo "Hello, $Name. Current date and time is $(date)."

      Lets put it in 'Build'
      Name = Zaman
      echo "Hello, $Name. Current date and time is $(date)"
  b. Save and Build Now
  d. Console Output
      Successfully executed

  e. Let input the above shell code in a file the CLI.
      This is known as Redirection
      echo "Hello, $Name. Current date and time is $(date)" > /tmp/info
      cat /tmp/info
      - outputs the above content:
      Hello, Zaman. Current date and time is Fri Dec 00:39 UTC ..
      Lets remove this 
      rm -rf /tmp/info


  f. We ca do the above thing from the jenkins server as well
    Paste the above redirect command in the Jenkins 'Build.sh' and you will
    get the file in /tmp/info
    
    

Starting from Video 21
Section 3: Getting Started with Jenkins

21. Executing a Bash Script from Jenkins

1. Lets make a ssh script here first:

    #!/bin/bash
    echo "Hello, $NAME $LASTNAME"

    - the above ssh script will have an error if you haven't given the permissions. Here is the command for permission:
    chmod +x ./script.ssh

    The above script will have the following output:
    Hello, 
    - it is so beacuse you haven't provided the env variables
    - let's take the env variable from the command itself. Hence, we modify the script to:
    #!/bin/bash
    NAME=$1
    LASTNAME=$2
    echo "Hello, $NAME $LASTNAME"

    Now in the command, we enter:
    ./script.sh Md Zaman
    Output:
    Hello, Md Zaman

2. To copy the above file to a container:
    docker cp script.sh jenkins:/tmp/script.sh

    Now, we can execute the same from our Jenkins by mentioning the path
    in the shell box. Open the job and under 'Build' under 'Execute shell':

    /tmp/script.sh Md Zaman

    Click on Build now and see the result in the Output:
    Successfull

    Alternatively, you can also use the below command:
    NAME=Md
    LASTNAME=Zaman
    /tmp/script.sh $NAME $LASTNAME

    - gives you the same output

    Takeaway is that you can execute a shell script from Jenkins

22. Add Parametres to your job in Jenkins

    We can enable parametres in a jenkins job by:
    Configure>General>'The project is parameterized'
    Let us use the 'String Parameter' here
    This way you can get to change the parameters when we click on 'Build Now'- We will get an option:
    For String Parameter we get 3 options:
    a. Name: FIRST_NAME [This will the name of the parameter, not the 
                parameter itself]
    b. Default Value: Md [This will be the default value if later you 
                don't put any value in the text box, this value will be used]
    c. Description: [I'll fill it later]

    Do the same this for the Second name:
    Name: SECOND_NAME 
    Default Value: Zaman
    Description: [I'll fill it later]

    Now in the 'Execute shell'
    echo "Hello, $FIRST_NAME $SECOND_NAME"

    Once you click on Build Now [By now this button will be changed to "Build with Parameters"]
    Here, you will be asked to enter the parameters.

23. Creating a Jenkins 'list' parameters with your script.
    Here, we will get multiple option to choose from. A dropdown to select from.
    Simple choose 'Choice Parameter'here.

    Name: LAST_NAME
    Choices:    Smith
                Gonzales
                Doe
    Description:

    Change the script:
    echo "Hello, $FIRST_NAME $SECOND_NAME $LAST_NAME"

    Now you will get option to Enter the First Name and the Second Name and a choice to select from a dropdown where the above options are available- Smith, Gonzales, Doe

24. Basic Logic and Boolean Parameters

1. We can make a logic and add a boolean parameter, in that case the name will only be displayed if the condition is true- in our case of the check box is checked.
Lets modify our shell script and execute it directly from Jenkins

#!/bin/bash
    NAME=$1
    LASTNAME=$2
    SHOW=$3
    if ["$SHOW = "true"]; then
        echo "Hello, $NAME $LASTNAME"
    else
        echo "If you want to see the name, please mark the show option"
    fi

    Go to Jenkins and add a boolean parameter and remove the choice parameter here.
    Name: SHOW
    Value: check [there will be a checkbox, if you check it it will be true and vice versa]
    
    
    and enter the shell path under "Execute Shell"
    /tmp/script.sh $FISRT_NAME $SECOND_NAME $SHOW


Section 4: Jenkins & Docker
25. Docker + Jenkins + SSH -1

Let's learn how we can connect from one server to another server.
We will use containers here to connect. 1st container will be Jenkins and 2nd container will be a container where we will install ssh.

We are using containers here but we can also use 2 different ec2s to represent two servers.

So, let us create 2 containers with the help of a 'docker-compose' file.

  version:'3'
  services:
    jenkins:
      container_name: jenkins
      image: jenkins/jenkins
      ports:
        - "8080:8080"
      volumes:
        - "$PWD/jenkins_home:/var/jenkins_home"
      networks:
        - net
    remote_host:
      container_name: remote-host
      image: remote-host
      build:
        context: ubuntu
  networks:
    - net

The second container, we will create a Dockerfile and write the below code:

    FROM ubuntu
    RUN apt -y install openssh-server
    RUN useradd remote_user && \
        echo "1234" | passwd remote_user --stdin && \
        mkdir 700 /home/remote_user/.ssh

To login from jenkins to ssh, we need to create a keygen. So enter the 
following command for keygen:
  ssh-keygen -f remote-key
  - (Press 2 Enters after the above command)
  - creates a public key and a private key

Now, since we have two keys- public and private, we will put the public 
key to the remote machine so that we can easily login to our ssh container 
using the private key. So, for this we will change the Dockerfile of ssh 
container as below:

  FROM ubuntu
    RUN apt -y install openssh-server
    RUN useradd remote_user && \
        echo "1234" | passwd remote_user --stdin && \
        mkdir 700 /home/remote_user/.ssh
    COPY remote-key.pub /home/remote_user/.ssh
    RUN chown remote_user:remote_user -R /home/remote_user/.ssh && \
        chown 600 /home/remote_user/.ssh/authorized_keys
    RUN /usr/sbin/sshd-keygen
    CMD /usr/sbin/sshd -D

Now, simply run deploy the containers with the follwing command and then we will try to connect from jenkins container to the ssh container:
  docker-compose up -d
  - brings up the containers

  Now let us login to the jenkins container and ssh to the remote ssh container:
  docker exec -it jenkins bash
  ssh remote_user@remote_host
  - this will ask for password and then you will be able to get inside of 
    the container.
  
  Let us also use the key file to connect to the ssh container. But before this ensure that you have the file in your jenkins container:
  docker cp remote-key jenkins:/tmp/remote-key

  Now enter into the ssh container:
  docker exec -it jenkins bash
  cd /tmp
  ls
  - you will find the remote key here
  ssh -i remote-key remote_user@remote_host
  - this time you will not be asked the password because you are using the 
    remote key

29. Jenkins Plugin Installation

To install any container you must ensure that of all it has the internet connection. Try to ping and see.
To install plugin click on:
Manage Jenkins>Manage Plugins
Then it is easy.

30. Integrate Docker ssh with Jenkins

For integration, click on Manage Jenkins>Configure System>SSH remote host
Click on 'Add' in SSH sites. Now fill in the details:
Hostname- remote_host
Port- 22
Credentails- from the dropdown select. If you don't find one, click on add.
  You can either add the cedential from here or 
  Credentails>Jenkins>Global Credentails>Add Credentials 
  Kind- SSH Username and Private key
  Scope Global - Default
  Username - remote_user
  Private key - [enter it directly] go to the cli and cat and copy the 
                SSH keygen
  Click Ok

  Now, check connection. 
  If it is successfull it indicates that Jenkins Server is able to ping 
  the SSH server from here.

31. Run Jenkins job on your Docker remote host through SSH

We can create a job in our Jenkins server which will be executed in our 
SSH machine by:

Create a new job>Under 'Build' Click 'Execute shell script on remote host using ssh' from the dropdown> Select your SSH and write the command which you want to execute.
SSH site - the machine that you have added
Command - the command that you want to execute. Let us execute the below 
          script:
          NAME=Zaman
          echo "Hello, $NAME. Current date and time is $(date)">/tmp/remote-file


Build the job and you will get the remote-file in the remote_host.

Section 5: Jenkins & AWS
32. We will create a jenkins job which will create an SQL backup and 
upload it on S3.

34. Step 1:
    Add one more service to the docker compose:
version:'3'
  services:
    jenkins:
      container_name: jenkins
      image: jenkins/jenkins
      ports:
        - "8080:8080"
      volumes:
        - "$PWD/jenkins_home:/var/jenkins_home"
      networks:
        - net
    remote_host:
      container_name: remote-host
      image: remote-host
      build:
        context: ubuntu
    db_host:
      container_name: db
      image: mysql:5.7
      environment:
        - "MYSQL_ROOT_PASSWORD=1234" # This is prescribed in docker.
      volume:
        - "$PWD/db_data:/var/lib/mysql" # This is where mysql data lives.
      networks:
        - net
  networks:
    net:

In the above docker compose file, environment variable is mentioned in the 
docker hub as for password. 
For volume, we are using the path of the container where mysql data lives.

Lets us recreate the mysql service along with other containers:
  docker-compose up -d
  - recreates all the containers

Check if the mysql containers is ready:
  docker logs -f db
  - to check the logs
  - we specifically want to know if this container is ready. There will 
    be a line where it says: 
    'mysqld: ready for connections.'

Get inside the container and login into mysql:
  docker exec -ti db bash
  - enter into the container

  mysql -u root -p
  - asks for password. Enter '1234' to login because this password 
    was assigned earlier
  
  show databases;
  - shows all dbs

34. Install MySQL Client and AWS CLI

We will install the MySQL client and AWS CLI in our SSH container

    FROM ubuntu
    RUN apt -y install openssh-server
    RUN useradd remote_user && \
        echo "1234" | passwd remote_user --stdin && \
        mkdir 700 /home/remote_user/.ssh
    COPY remote-key.pub /home/remote_user/.ssh
    RUN chown remote_user:remote_user -R /home/remote_user/.ssh && \
        chown 600 /home/remote_user/.ssh/authorized_keys
    RUN /usr/sbin/sshd-keygen

    RUN apt -y install mysql

    RUN apt -y install epel-release && \
        apt -y install python3-pip && \
        pip3 install --upgrade pip && \
        pip3 install awscli

    CMD /usr/sbin/sshd -D

We have added SQL and aws cli in our ubuntu remote host. Now, simply 
recreate the docker-compose to get the remote host updated.

  docker compose build
  - to build the container
  docker-compose up -d
  - creates the containers

Go inside the SSH container and enter values in your sql data:
  docker exec -ti remote-host bash
  - enters into the container
  mysql -u root -h db_host -p
  - enters into the mysql container from remote_host container
  - here we are adding the '-h' which means the host where we want to 
    login as from our current container
  - enter the password to login

  Enter the following set of commands to create a database and enter details:
  show databases;
  create database testdb;
  use testdb;
  create table info (name varchar(20)), lastname varchar(20), age int(2);
  show tables;
  desc info;
  insert into info values ('md', 'zaman', 21);
  select * from info;

36. Create an S3 Bucket
Create an S3 bucket in AWS

34. Add a user and attach a policy giving it S3 full access: 
'AmazonS3FullAccess' and preferably down the access file for 
authentication.

35. Manually take a backup and upload it on S3

Steps:
a. Login into the remote container
b. Create a backup [there is a command for this]
c. Upload this backup to S3. Since we have AWS CLI already in this 
    container [from Dockerfile]

a. Login:
  docker exec -ti remote_host bash

b. mysqldump -u root -h db_host -p testdb > /tmp/db.sql
    - creates a backup of 'testdb' from the host (here, host=db_host and 
      testdb is the name of the db) 
  Make sure you are logged in to your AWS CLI. You can do that by:
  export AWS_ACCESS_KEY_ID=<your_ID>
  export AWS_SECRET_ACCESS_KEY=<your_SECRET_KEY>

c. aws s3 cp /tmp/db.sql s3://jenkins-mysql-backup/db.sql
    - copies the 'db.sql' back to our s3

39. & 40. Automating the backup and upload process with a shell script

We are going to use the following script:

#/bin/bash

DATE=$(date +%H-%M-%S)
BACKUP=db-$DATE.sql

DB_HOST=$1
DB_PASSWORD=$2
DB_NAME=$3
AWS_SECRET=$4
BUCKET_NAME=$5

mysqldump -u root -h $DB_HOST -p$DB_PASSWORD $DB_NAME > /tmp/$BACKUP && \
export AWS_ACCESS_KEY_ID=AKIAJRWZWY3CPV3F3JPQ && \
export AWS_SECRET_ACCESS_KEY=$AWS_SECRET && \
echo "Uploading your $BACKUP backup" && \
aws s3 cp /tmp/db-$DATE.sql s3://$BUCKET_NAME/$BACKUP

41. Manage Sensitive Information in Jenkins (Keys and Passsword)

We have 2 sesitive information in our script which we want to execute 
The sql password and our AWS secret key.
Go to jenkins and simply add them in the credentials by selecting secret 
key.

42. Create a Jenkins job to upload your DB to AWS

a. Select freestyle project and name it as you want. We will name it 
    'backup-to-aws'
b. Select on 'this project is parameterised' and select the 'String 
    Parameter'
c. Look at the script and first add those parameters which are not 
    secret. (exclude the sql pass and aws secret key for now)
    Add the parameter as directed earlier
d. For secrets scroll down and find 'Build Environment' and select 
    'Use secret texts or files'
    You will get a 'Binding' option
    Select 'Secret file' here
    You will get the dropdown here. This is coming from 'credentials' 
    you have defined earlier
    So, add sql password and the aws secret key
e. Scroll down further you will get a 'Build' section and select from 
    the dropdown 'Execute shell script on remote host using ssh'
    You will get the remote host the password of which you have added 
    earlier.
    Now, in the 'Command' section, add the command with the parameters 
    you have mentioned above:
    /tmp/script.sh $MYSQL_HOST $MYSQL_PASSWORD $DATABASE_NAME $AWS_SECRET_KEY $AWS_BUCKET_NAME 
    Save the job and execute it 
    

Skipping Section 6: Jenkins & Ansible

Section 7: Jenkins & Security

67: Intro- Learn how to Enable/Disable Login in Jenkins

Configure Global Security and enable it.
This is to ensure that whenever we enter the URL from incognito browser, we
need password to login. If we don't do this and enter the URL, we will be 
directly logged in in Jenkins.

68: Allow Users to Sign Up

Manage Jenkins > Configure Global Security > Security Realm >
Allow users to sign up (This is a checkbox) enable/disable this.
Now, when you enter the URL, you will have the option to sign up

* Keep this disabled because any user who signs up is by default an 
admin and has all the access of all the jobs etc..

69. Install powerful security plugin

There is a security plugin by the name : "Role-based Authorization Strategy"
Install it.
Now, let's play with it..
Manage Jenkins > Configure Global Security > Authorization > Role-Based 
Strategy > Save
When you do this we see a new section in "Manage Jenkins" by the name: 
"Manage and Assign Roles"

70. Create users manually in the Jenkins DB

This is how to do it without signing up from the Login Option shown 
previously.
Manage Jenkins > Manage Users > Create User
If you create a user using this method, and login you will see the msg:
"Access Denied - tom is missing the Overall/Read permission" 
This is because of the security plugin we had installed (Role-based Authorization Strategy) 
So, users cannot have any permissions unless you explicitly define that.
We have delegated the permissions to the above plugin.

71. Ever heard about roles? Let's create a Read Only role!

Manage Jenkins > Manage and Assign Roles >  There are options here you 
have to look
 

73. Create a role to execute jobs, and assign that role to your user

Same as above. Here, create a "Role to add" and then make such a role
Ensure to enable "Overall Read"
Save
Then click "Assign Roles"
and explore the options here.

74. Learn how to restrict Jobs to users using Project Roles.

There is an option called "Project roles"
Use that and go to Assign Roles and do it.
Explore it.

Section 8: Jenkins Tips & Tricks

75. Global environment variables in Jenkins

There are few Global Variables already available in Jenkins which we don't 
have to define. Google and you will find

To check we make a freestyle project and under "Execute shell" write in 
command: 
echo "Build Number is $BUILD_NUMBER"
echo "Build ID is $BUILD_ID"

All these variables are predefined variables in Jenkins
Build Now and check the Console Output

Application- when we need mail to notify that this build number with this 
job ID is failing etc.

76. Create your own custom global environment variables

Manage Jenkins > Configure System > Global Properties > Environment Variables (enable this) > Add 
Give and name and its value

You can check if it is working.

Stop

77. Modify the Jenkins URL

You can edit the jenkins URL (doesn't seem that important)

78. Meet the Jenkins' cron: Learn how to execute Jobs automatically

Used in Backup jobs 
Supppose you want to take a backup at 1 am at night

Jenkins has a built-in cron like a scheduler

Select any job or create a new one > Configure >
Build Triggers > Build periodically > Here we need to add a cron 
expression - Google to find out cron expression like "cron expressio for 
everyday at 1 am" and copy and past it here.

You can also make it every single minute ans save the job.

Now this job will run every single minute.

79. Learn how to trigger Jobs from external sources: Create a generic user

Create a user and give the user necessary permissions so that it can 
trigger the jobs etc.
Manage > Manage Users > Create User > Enter

Manage Jenkins > Manage Roles > Create a role by the name "trigger-jobs" 
and for 
Overall section : "Read" for "trigger-jobs"
Job section: "Build" and "Read"
Save

Now assign the user "Jenkins" and give the role of "trigger-jobs"
Save

81. Trigger your Jobs from Bash Scripts (No parameters)

a. Right click on "Build Now" button and copy the link (You know this link 
will do the same thing as the button)

b. Token - We need to pass something like a token
    Manage Jenkins > Configure Global Security > CSRF Protection 
      - ignore this - this was to tell you that it uses crumbs
    
    We need to retrieve a crumb by using the below link:
    crumb=$(curl -u "jenkins:1234" -s 'http:jenkins.local:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)')
    - here the link "jenkins.local" is is not recognised so let us edit the
      host
      sudo vo /etc/hosts
      Write the below:
        127.0.0.1  jenkins.local

      After adding the host, you will be able to resolve it by:
      curl jenkins.local:8080
      So, now jenkins will also return a crumb if we enter:
      curl -u "jenkins:1234" -s 'http:jenkins.local:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)'
      - this will return a crumb like:
        Jenkins-Crumb:6ef554... jenkins-data]$
      Let us execute the same command in a variable:
      crumb=$(curl -u "jenkins:1234" -s 'http:jenkins.local:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)')

      Now if we echo crumb we will get a value
      echo $crumb
      jenkins..

      Let us create a scrript to create the job:
      
      vi crumb.sh
      crumb=$(curl -u "jenkins:1234" -s 'http://jenkins.local:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)')
      curl -u "jenkins:1234" -H "$crumb" -X POST http://jenkins.local:8080/job/ENV/build?delay=0sec

      chmod +x crumb.sh
      ./crumb.sh

82. Trigger your Jobs from Bash Scripts (With Parameters)

Over here also we will do the same thing i.e., we will retrieve the crumb 
and add the add the link of the "Build Now" button but this time, we will 
add parametres with this link.

  vi crumb.sh
  crumb=$(curl -u "jenkins:1234" -s 'http://jenkins.local:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)')
      curl -u "jenkins:1234" -H "$crumb" -X POST http://jenkins.local:8080/job/ENV/build?delay=0sec
  chmod +x crumb.sh
  ./crumb.sh

Section 9: Jenkins & Email

84. Install a Mail Plugin 
    Install the plugin "Mailer Plugin"

85. Integrate Jenkins and AWS Simple Email Service

    On AWS:
    Go to SES (Simple Email Service)
    Verify a Domain or an Email Address

    On Jenkins:
    Manage Jenkins > Configure System > Email Notification > Click on 
    "Advanced" and provide all the information > get the smtp settings 
    from AWS (left-hand side)

    Fill all and come to AWS and "Create smtp credentails" 
    make an IAM user and create 
    after saving come back here and you will be able to find out all the 
    ports you can use
    Paste the port here in jenkins also
    Scroll up and enter the System Admin e-mail address = ...
    Test configuration


86. Integrate Jenkins and Gmail

  Same way but use your gmail Username and Password in Jenkins
  Google for rest of the details.

  Activate Gmail "less secure apps"
  Test Configuration - Success (Check your gmail)


87. Add notifications to your jobs

  a. Open any job and scroll down to the last part: "Post-build Actions"
  b. Click "Add post-build action" > "E-mail Notification"
  c. Add email notification Recipents - <enter your mail>

  Now you will receive an email whenever the jobs fails
  Once you fix the job, you will receive an email that the job is back to 
  normal.
  But it will not send you an email for every successful build henceforth

  

  



