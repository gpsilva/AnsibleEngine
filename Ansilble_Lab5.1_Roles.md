# 2. Create Roles

Create roles to deploy the web application. You create a role to set up the Apache web server. Then you create a role to install mariadb and use a database backup file to populate the database. Lastly, you create a role for setting up a HAProxy load balancer for high availability for your web application.

## 2.1 Create Role to setup up web services

1. Create a role called *app-tier* using the *ansible-galaxy* command.

2. Add a task to install and enable *firewalld*.

3. Add a task to install and start *httpd*.

4. Add a task to create a custom *vhost.conf* configuration file under the */etc/httpd/conf.d/* directory.

    * The *vhost.conf.j2* template is already created to help you with this step.

5. Add a task to create the */var/www/vhost/* document root directory.

6. Add a task to create an *index.php* file in the document root directory using *index.j2* as the template.

7. Add a task to open the firewall ports as per the requirements.

8. Enable SELinux so that the Apache back-end server can connect to the database:

   * *httpd_can_network_connect_db*

   * *httpd_can_network_connect*

9. Create the vars/main.yml file under the app-tier role directory that contains definitions for all of the variables defined in tasks.

10. Create the file *handlers/main.yml* under the *app-tier* role directory that contains a handler to *restart* services if needed.

