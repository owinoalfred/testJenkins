A Practical Ansible Project
===========================

To install Ansible on your machine, follow these steps:

1. Install the required software properties common package:

```shell
sudo apt-get install software-properties-common
```

2. Add the Ansible Personal Package Archive (PPA) repository:

```shell
sudo apt-add-repository ppa:ansible/ansible
```

3. Update the package lists:

```shell
sudo apt-get update
```

4. Install Ansible:

```shell
sudo apt-get install ansible
```

Once you have completed these steps, Ansible will be installed on your machine.

There are host files, also known as inventory files, where you can declare defaults and group hosts together. The default inventory file is located at '/etc/ansible/hosts'. You can specify a different inventory file using the -i flag or a variable in the playbook.

Inventory files are written in the INI format, which is used for writing settings files. Here's an example of a host file:

```ini
127.0.0.1 localhost
12.34.5.65 node1
32.3.45.6.4 node2
```

To verify that everything has been set up correctly, you can use the following command:

```shell
ansible -m ping all
```

We can also attempt to connect it through SSH directly using the code below:

```shell
ssh ubuntu@node1
```

This won't accept the connection since the private key hasn't been used to login. So, we shall download what we call an SSH key, which is given to you every time you create an instance on the cloud.

To add the key to the particular session, we will use a program called `ssh-agent`. This program holds private keys used for public key authentication. We usually start `ssh-agent` before a login session to verify the public key with the private key.

To run the `ssh-agent`, we first get its PID (Process ID) with the command below:

```shell
eval `ssh-agent -s`
```

After confirming that it is running, we then add the key with the command below:

```shell
ssh-add NameOfTheKey.pem
```

After completing the installation, you can connect to the virtual machine by running the following command:

```shell
ssh ubuntu@node1
```

To disconnect from the virtual machine, use the command:

```shell
exit
```

To verify the connection to the virtual machine, you can use the following command:

```shell
ansible -m ping all
```

It is a best practice to ping the machine before running any playbooks to ensure a successful connection.

Now, let's discuss how to write Ansible commands in Markdown.

To execute commands in the terminal using the ad-hoc method, you can use the following format:

```shell
ansible <target> -m <module-name> -a arguments
```

In this format, `<target>` refers to the host or group of hosts on which the command will be executed. `<module-name>` represents the specific Ansible module to be used, and `arguments` are the parameters or options passed to the module.

For example, to install a package on a target machine, you can use the `apt` module as follows:

```shell
ansible <target> -m apt -a "name=<package-name> state=installed"
```

To copy a file from the local machine to the target machine, you can use the `copy` module:

```shell
ansible <target> -m copy -a "src=<source-path> dest=<destination-path>"
```

These ad-hoc commands are useful for quick tasks or one-time operations.

On the other hand, if you want to write commands as part of a playbook, you can define tasks using the YAML syntax. Here's an example playbook:

```yaml
---
- hosts: <target>
    tasks:
        - name: Install package
            apt:
                name: <package-name>
                state: installed

        - name: Copy file
            copy:
                src: <source-path>
                dest: <destination-path>
```

In this playbook, `<target>` represents the host or group of hosts on which the tasks will be executed. Each task is defined with a name and the corresponding module, along with any required parameters.

By using playbooks, you can automate complex tasks and manage multiple hosts simultaneously.

Remember, Ansible provides a wide range of modules to control system resources, services, or files. You can explore the available modules by running the command:

```shell
ansible-doc -l
```

That's it! You now have a better understanding of how to write Ansible commands in Markdown.

For example, to use the ad-hoc method, we can write the following command in our terminal:

```shell
ansible demo_host -m copy -a "src=/tmp/test1 dest=/tmp/test1"
```

In this command, the `-m` flag specifies the module to be used, which in this case is `copy`. The `copy` module allows us to copy files from the local machine to the destination folder on the target machine.

To explore the available modules, you can run the following command:

```shell
ansible-doc -l
```

This command will provide a list of all the available modules in Ansible.

**Facts:**

This is information about the target machine fetched in the local machine. As we write the commands, these facts will keep popping up. To get all the facts, run the command below:

```shell
ansible hosts -m setup
```

**Variables:**

The facts spit out some variables. An Ansible variable is a temporary placeholder for a piece of information about a machine. We use variables to deal with differences in the system. Instead of using a hard, long name of a resource, we can use a variable to refer to it, making our work easier and more uniform. Variables can be defined in the inventory files or in the playbooks. We can also create separate files for the variables and then declare them in the playbook.

**Roles:**

We use roles to manage, organize, and load certain variables, files, etc. Roles help us structure our playbooks and make them more modular and reusable.

### Sections in a Playbook

```markdown
apt: pkg=nginx state=installed update_cache=true
```

This is a module command that accomplishes something on the target machine. Remember, you can also define your own modules and share them with the rest of the world. The `installed=true` means we want to install the package on the target machine, and the `update_cache` first updates the packages. The `become` allows you to become any user as you install.

If you want to uninstall, then you will use `state=absent`.

In the above playbook, we add another task named installing `wget`. Here, we introduce a variable name. Like we said, we want to install it, so the `state` is set to `installed`. (Check the synopsis for each module before you use it in your playbook.)

So, we need to configure our remote user so that it can execute the commands using the `sudo -i` privileges without asking for passwords. If we were using the Ubuntu cloud machines, we wouldn't need to set that up since it comes set with the `sudo -i` command by default. But since we will be using the local machine, we need to configure this manually by configuring the sudo file. Go into the sudo file and enable passwordless sudo access for the particular user that you are using. However, please note that this comes with risks.

To run the playbook, you can simply type:

```shell
ansible-playbook NameOfThePlaybook.yaml
```

We can add another task to our playbook, and this one should copy a file from the master machine to the other machine.

To find out whether our changes occurred, let's SSH into our Ubuntu node 1 using the command:

```shell
ssh ubuntu@node1
```

And to first find out whether our file was copied, you can use the command:

```shell
cat /tmp/test11
```

This will show the contents of the file.

We will also run `$dpkg -l | grep nginx` to check the status of the process. Using `service nginx status`, if it is installed, it will run.

Next, we will be installing Apache web server onto the nodes that we created. We start by ensuring the connection is established by running the `$ansible -m ping all`.

Now, let's create another playbook as below:

```yaml
---
- hosts: web_portal
    tasks:
        - name: apt get update
            apt:
                update_cache: yes
        - name: install apache
            apt:
                name: apache2
                update: no
        - name: copy data files
            copy:
                src: index.html
                dest: /var/www/html/
```

Create the host group and add node1 to it, then ping it to see the results. But we should run it with the argument `-b` to elevate our privileges.

To see whether the changes were made, SSH into the node using:

```shell
ssh ubuntu@node1
```

Then run:

```shell
dpkg -l | grep apache
```

It should show that Apache is installed.
