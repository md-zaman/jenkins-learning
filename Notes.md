

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
        




 