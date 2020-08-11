![image](https://user-images.githubusercontent.com/61896468/89897000-0035f880-dbfc-11ea-9dc3-ace04cf3b00a.png)

# LAUNCH WEB SERVER ON THE TOP OF DOCKER USING ANSIBLE PLAYBOOK 

Task in hand: Creation of an Ansible Playbook to perform the following Operations on the managed nodes.

Configure the Yum for docker.

Starting and enabling of the docker services.

Pulling of the httpd server image from the Docker hub.

Running the httpd container and exposing it to the public.

copying of the html code in /var/www/html directory and starting of the webserver.

So, Before Completing our task lets learn something about Ansible.

Ansible :

Ansible is an open-source automation tool, or platform, used for IT tasks such as configuration management, application deployment, intraservice orchestration, and provisioning. Automation is crucial these days, with IT environments that are too complex and often need to scale too quickly for system administrators and developers to keep up if they had to do everything manually. Automation simplifies complex tasks, not just making developers’ jobs more manageable but allowing them to focus attention on other tasks that add value to an organization. In other words, it frees up time and increases efficiency. And Ansible, as noted above, is rapidly rising to the top in the world of automation tools.

Playbooks

Ansible playbooks are like instruction manuals for tasks. They are simple files written in YAML, which stands for YAML Ain’t Markup Language, a human-readable data serialization language. Playbooks are really at the heart of what makes Ansible so popular is because they describe the tasks to be done quickly and without the need for the user to know or remember any particular syntax. Not only can they declare configurations, but they can orchestrate the steps of any manually ordered task, and can execute tasks at the same time or at different times.

Ansible Architecture :

No alt text provided for this image

Now lets begin with our task:

* So in the first step I will be launching 2 OS in which one of the node I have configured as the controller node which will be configured with the Ansible tool and this node will be used for writing the anisble-playbook.

* The other node will be my manager node on which all the operations are to be performed.

* You use the yum command to install software.

$ yum install ansible -y

* if that doesn't work you can use pip3 command to install the same. Since, Ansible is a tool that uses python3 script in its backend.

$ pip3 install ansible

and check the version with this.

$ ansible --version

No alt text provided for this image

Note : if you have installed Ansible with yum command it will give you a config file and a host file already configured but if you have installed the same with the pip3 command you have to create both the files and give the path of the host file in the config file. I have used Yum to install, so it if giving the config file pre-configured.

Now if you will open the ansible.cfg file you can see the all the setting already done in form of comments.

No alt text provided for this image

Inventory is all your hosts in ansible, as you can see it has already created hosts file and set its path in here all you have to do is remove the # in front of it to remove the comment so that ansible can read it.

No alt text provided for this image

you can also remove # from host_key_checking and set it to false, because when ansible tries to connect to a remote system it connects through the ssh protocol which will fail.

Now Lets check our hosts file.

No alt text provided for this image

Note : you can mention the IP Addresses of your remote systems here or you can add the user and password with it to avoid any failure because of ssh protocol.

Now to check the list of all my available hosts I have configured use the command

$ ansible all --list-hosts

Now to check weather my nodes are working properly or not I will ping once to check their communication

$ ansible <IP_Address_of_the_remote_system> -m ping

When you receive the response as "pong" means its working absolutely fine.

* we will be using the ansible-playbook using one single YAML file because ansible only understands YAML language to perform Task.

* So here the first thing we need to do is to configure the yum for the docker.

- hosts: <IP_Address_of_the_remote_system>

  tasks:

    - name: Configuring yum repository for docker

      yum_repository:

        name: dockerdvd

        baseurl: https://download.docker.com/linux/centos/7/x86_64/stable

        description: docker repository

        enabled: true

        gpgcheck: no

* Up next we will install the docker package, start the docker services and enable it.

    - name: Install Docker package and start the Docker Service

      package:

        name: "docker-ce-18.06.3.ce-3.el7.x86_64"

        state: present

               

    - name: Docker service start and making permanent

      service:

        name: "docker"

        state: started

        enabled: yes

* Now we need docker SDK for python3 for running the docker module

    - name: python3 Docker SKD

      command: pip3 install docker

      register: x

      ignore_errors: yes

         

    - debug:

        var: x

* Now we will create a directory in our remote system and copy our code to the directory in our remote system.

and thenwhere our apache-server runs that is into the /var/www/html. And for performing the same I will be using the file and the copy command.

    - name: Creating a folder in the remote system

      file:

        path: "/arun_ansible"

        state: directory

               

    - name: copying the files to the remote system

      copy:         

        src: "home.html"

        dest: "/arun_ansible/"

* Now up next we will create our docker container and for that we will first pull the httpd image of docker and then we will move the location of our file to the /arun_ansible/: /usr/local/apache2/htdocs/ and then we will expose it to the outside world.

    - name: Creating a Docker Container

      docker_container:

        name: "client-server"

        image: "httpd"

        state: started

        exposed_ports:

          - "80"

        ports:

          - "1280:80"

        volumes:

          - /arun_ansible/:/usr/local/apache2/htdocs/

now that all our requirements are set, run the file using the command...

$ ansible-playbook filename.yml

you will see...

No alt text provided for this image

to check the ip of your system with the port number 1280 with is connected to the port 80 of your docker container.

$ Docker ps

to list all the running container in your system.

No alt text provided for this image

Now you can check the website using the IP address of the Docker Container or you can do the same using the command prompt using the curl command.

No alt text provided for this image

with this we complete our task.

Thank you.
