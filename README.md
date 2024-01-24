# ansible
Provides example Ansible playbooks to automate the installation and configuration of ThinManager.

Additionally, there are 2 videos that show the playbooks in action - (1) [using ThinManager Activation (TMA)](https://youtu.be/oaAV2rtJhBU) and (2) [using FactoryTalk Activation (FTA)](https://youtu.be/udUWmjdlNUg).

There are 2 example playbooks provided - one to install ThinManager (thinmanager-install.yaml), and one to configure ThinManager (thinmanager-config.yaml).

The test bed used to create these playbooks included 2 Windows Server 2019 Standard builds - (1) a domain controller (DC.tmlab.loc), and (2) the host server where ThinManager would be installed (TMAnsible.tmlab.loc).  The Ansible control node was a Ubuntu 22.04.3 LTS image with Ansible version 2.15.8 installed.

In order for Ansible to control Windows hosts, WinRM must be configured on the host.  The [ConfigureRemotingForAnsible.ps1](https://github.com/AlbanAndrieu/ansible-windows/blob/master/files/ConfigureRemotingForAnsible.ps1) script was used to accomplish this.  In order for Ansible to control the Windows host remotely, you will need a Windows account with local Administrator permissions.  These credentials are set in the thinmanager file located in the group_vars folder.  In this file, also note the ansible_winrm_transport variable was set to "ntlm".

Install Playbook:  thinmanager-install.yaml
-------------------------------------------
Terminal command to launch (from location of the thinmanager-install.yaml file): 
ansible-playbook thinmanager-install.yaml -e "NODES=thinmanager" -vvv

This playbook installs ThinManager v13.2.0.  You can control the activation method used by ThinManager by setting install_ftam variable in the hosts file.  Setting it to a 1 will install FactoryTalk Activation Manager (FTAM), while setting it to a 0 will use ThinManager Activation (and not install FTAM).  If using FTAM, you can also automatically activate the FTA license by specifying the fta_serial_num, fta_product_key, and fta_qty variables also in the hosts file.  This will require the target host to have access to the Internet to work.  This playbook utilizes the separate install of FactoryTalk Activation Manager (v5.00.13) because this produced more consistent results with Ansible than the FTAM version included in the v13.2.0 installation folder of ThinManager.

We install the ThinManager MSI separate from the Common Installer in order to set the ENABLE_API argument to a 1 so we can automate the initial configuration of ThinManager in a separate playbook.  Once the ThinManager MSI is installed, the playbook will then install FactoryTalk Activation Manager if the install_ftam host variable is set to 1.

Once installation is completed, the playbook the configures the ThinServer service to run as a domain account that has local Administrator permissions on the host.  The domain account credentials are specified in the tm_service_username and tm_service_password variables.

Lastly, the playbook opens the commonly used ports on the Windows firewall.  The last 2 are commented out but can be included if your deployment will utilize a Docker Container host and/or Active Directory.

Configuration Playbook:  thinmanager-config.yaml
------------------------------------------------
Terminal command to launch (from location of the thinmanager-install.yaml file): 
ansible-playbook thinmanager-config.yaml -e "NODES=thinmanager" -vvv

This playbook performs some initial configuration of ThinManager v13.2.0, leveraging the REST API.  It first logs into the ThinManager REST API using the credentials supplied in the tm_service_username_upn and tm_service_password variables.  This initial REST API call returns an API key to be used in the other REST API plays.

The first action performed is to set a ThinManager configuration database password (tm_db_password).

The playbook then attempts to activate ThinManager based on the install_ftam host variable (if set to a 1, ThinManager is configured for FTA - if set to a 0, ThinManager is configured for TMA).

With activation handled, the playbook then demonstrates how to create an RDS Display Server, RDS Display Client, and a Terminal.  The best way to explore ThinManager's REST API is by using the built-in Swagger User Interface, which can be accessed by:

https://x.x.x.x:8443/api/documentation

Here you will see all of the REST API endpoints that ThinManager supports, and can try each one out.  By trying out an endpoint in the Swagger UI, you can see what Body each endpoint expects, which can then be used to body_text variable in each play below.

The last play in this playbook sets some initial ThinManager Server settings - like enabling the PXE Server and disabling Device Authentication.  This was placed at the end of the playbook because enabling the PXE Server takes more time for ThinManager to process, and subsequent REST API calls made immediately to ThinManager after enabling PXE would sometimes fail.

NOTE:  in version 13.2.0 of ThinManager, the /system/licensing/tma/license POST method, and the /system/licensingmode PUT method are not working properly.  As such these plays have been commented out.  This just means you will need to install your license manually if using TMA, or if using FTA, you will need to set the Licensing Mode to FTA manually in ThinManager.

Ansible Book Reference:
-----------------------
Also - a great reference for getting started with Ansible - [Ansible for Real-Life Automation](https://a.co/d/d0SOn0B).



