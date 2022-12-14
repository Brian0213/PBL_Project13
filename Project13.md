# Documentation for Project 13: ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

## Prerequisites:

- INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE

- Introducing Dynamic Assignment Into Our structure
In your https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it dynamic-assignments.

Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml. We will instruct site.yml to include this playbook later. For now, let us keep building up the structure.

Your GitHub shall have following structure by now.

Note: Depending on what method you used in the previous project you may have or not have roles folder in your GitHub repository – if you used ansible-galaxy, then roles directory was only created on your Jenkins-Ansible server locally. It is recommended to have all the codes managed and tracked in GitHub, so you might want to recreate this structure manually in this case – it is up to you.

- Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

Your layout should now look like this.


- Now paste the instruction below into the env-vars.yml file.

- ![Env Vars Yml File Update](./images/env-varsyml.PNG)

- Notice 3 things to notice here:

1. We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:
include_role
include_tasks
include_vars

2. We made use of a special variables { playbook_dir } and { inventory_file }. { playbook_dir } will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. { inventory_file } on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

3. We are including the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

### UPDATE SITE.YML WITH DYNAMIC ASSIGNMENTS

- Update site.yml with dynamic assignments

- Update site.yml file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come. So hang on to your hats)

- site.yml should now look like this.

- Community Roles
Now it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

- Download Mysql Ansible Role
You can browse available community roles here

We will be using a MySQL role developed by geerlingguy.

- Inside roles directory create your new MySQL role with ansible-galaxy install geerlingguy.mysql and rename the folder to mysql

`ansible-galaxy install geerlingguy.mysql`

![Ansible Mysql Download](./images/ansible-mysql.PNG)

`mv geerlingguy.mysql/ mysql`

![Mysql folder Name Change](./images/change-to-mysql.PNG)

#### LOAD BALANCER ROLES

- Load Balancer roles
We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

Nginx
Apache
With your experience on Ansible so far you can:

Download Nginx:

`ansible-galaxy install geerlingguy.nginx`

![Nginx Download](./images/nginx-dwnld.PNG)

Download Apache:

`ansible-galaxy install geerlingguy.apache`

![Apache Download](./images/apache-dwnld.PNG)

Change the names of the downloaded apache and nginx folder:

`mv geerlingguy.nginx/ nginx`

`mv geerlingguy.apache/ apache`

![Apache Nginx Folder Name Change](./images/apache-nginx-foldername-change.PNG)

- Go to the main.yml file inside the Nginx folder and copy the uat-webserver ip address inside the inventory to it.

![Copy Inventory/Uat to Nginx/Defaults/Main](./images/cp-inven-uat-nginx-defaults-main.PNG)

- In the Nginx Defaults Main.yml file, uncomment nginx_extra_http_options:

![Task above ](./images/nginx-http-uncomment.PNG)

![Task Main 1](./images/task-main-1.PNG)

![Task Main 2](./images/task-main-2.PNG)

- Make changes to the setup-RedHat.yml file in nginx/tasks:

![Set Redhat Changes](./images/nginx-redhat-fileupdate.PNG)

In the apache defaults main.yml file, add the uat web servers

![Adding Webservers to main file](./images/uat-webservers.PNG)

- Make changes to the setup-RedHat.yml file in apache/tasks

![Set Redhat Changes](./images/apache-task-set-redhat.PNG)

- In apache defaults main.yml add the load balancer name under apache_vhosts:

![Apache LoadBalancer Update](./images/apache-vhosts-loadbal.PNG)

- Update the Nginx task main.yml with the webserver /etc/hosts info:

![Nginx EtcHosts Update](./images/nginx-etc-hosts.PNG)

- Important Hints:

Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of variables.

Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.

Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.

Declare another variable in both roles load_balancer_is_required and set its value to false as well

![Apacge Enable](./images/apache-enable-load-bal.PNG)

![Nginx Enable](./images/nginx-enable-load-bal.PNG)

Update both assignment and site.yml files respectively:

Create a loadbalancers.yml file in the static-assignments folder:

![Loadbalancer file](./images/lb-file.PNG)

Create a db.yml file in the static-assignments folder:

![Database file](./images/db-file.PNG)

![Site file](./images/site-yml-.PNG)


- Now you can make use of env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

You will activate load balancer, and enable nginx by setting these in the respective environment’s env-vars file:

![Env Uat file](./images/update-envs-uat.PNG)

- Run the playbook command:

`ansible-playbook -i inventory/uat.yml playbooks/site.yml`

![Playbook Run](./images/ansible-inventory-uat-siteyml.PNG)

- Apache Playbook run output:

![Apache Output](./images/ansible-inventory-uat-siteyml.PNG)

![Apache Output](./images/apache-output-2.PNG)

![Apache Output](./images/apache-output-3.PNG)

![Apache Output](./images/apache-output-4.PNG)

![Apache Output](./images/apache-output-5.PNG)

- Loadbalancer Playbook run output:

![LoadBalancer Output](./images/lb-output-1.PNG)

![LoadBalancer Output](./images/lb-output-2.PNG)

![LoadBalancer Output](./images/lb-output-3.PNG)

![LoadBalancer Output](./images/lb-output-4.PNG)

![LoadBalancer Output](./images/lb-output-5.PNG)

![LoadBalancer Output](./images/lb-output-5.PNG)

![LoadBalancer Success Output](./images/lb-success-output.PNG)

- Folder and Files configuration:

![Apache Enable](./images/apache-enable-load-bal.PNG)

![Apache Task Folder](./images/apache-task-set-redhat.PNG)

![Apache vhosts](./images/apache-vhosts-loadbal.PNG)

![Nginx Folder](./images/nginx-dwnld.PNG)

![Nginx Enable](./images/nginx-enable-load-bal.PNG)

![Nginx Etc Hosts](./images/nginx-etc-hosts.PNG)

![Nginx Http](./images/nginx-http-uncomment.PNG)

![Nginx Redhat](./images/nginx-redhat-fileupdate.PNG)