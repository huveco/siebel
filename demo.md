# Playbook: siebel-migration-plan-repository-export.yml

## Summary
- This guide shows the steps on how to use the playbook

## Prerequisites
- Siebel Enterprise up and running
- Siebel ADM Deployment Enabled
- Siebel Migration Application Enabled
  - Ref Playbook: siebel-migration-application-enable.yml

## Implementation Steps

### Create Ansible inventory file
Defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.

1.  Navigate to the Ansible inventories directory
    ```sh
    $ cd << inventory_directory >>
    ```

1.  Create a new directory with the inventory name
    ```sh
    $ mkdir << inventory_name >>
    $ cd << inventory_name >>
    ```
    
1.  Create a file with the hosts
    ```sh
    $ vi hosts
    ```

1.  Add the following lines replacing the ip adresses with yours. Do not change the Group Names
    ```sh
    [source-siebel-server]
    129.213.113.88 ansible_user=<< ansible_user >>

    [source-siebel-smc]
    129.213.113.88 ansible_user=<< ansible_user >>
    ```

1.  Save the file


### Assign variables to all hosts

1.  Navigate to the new Ansible inventory directory
    ```sh
    $ cd << inventory_directory >>/<< inventory_name >>
    ```

1.  Create a new directory for the hosts variables
    ```sh
    $ cd << inventory_directory >>/<< inventory_name >>
    $ mkdir group_vars
    $ cd group_vars
    ```
    
1.  Create a file with all the variables. The file name must be all.yml
    ```sh
    $ vi all.yml
    ```

1.  Add the following variables
    ```sh
    ---
    migration_plan_export:            DDIT_DROP_19_2_N_2                    ## Source Env: Migration Plan Name to Execute and Export
    migration_plan_export_file:       DDIT_DROP_19_2_N_2_export.zip         ## Source Env: Export Package File Name that will be generated
    migration_sfs_path_source:        /u01/app/siebel/filesystem/migration  ## Source Env: Siebel File System directory
    migration_tableowner_source:      SIEBEL
    migration_tableownerpwd_source:   SIEBEL2017
    migration_dbusername_source:      SADMIN
    migration_dbusernamepwd_source:   SADMIN2017
    migration_ai_redirectport_source: 8081
    ses_shutdownport:                 8102
    ```
1.  Save the file

### Tasks from Ansible Tower
The following tasks are created from Ansible Tower

#### Create a new Inventory

1.  Login as a Superuser. Open a web browser and log in to Tower by browsing to the Tower server URL
    ```sh
    http://129.213.21.160
    https://<Tower server name>/
    Username: admin
    Password: password
    ```
    ![alt text][png001]    


1. Create a new inventory by navigating to they inventories tab. Click on ” + ”, and Details Tab
    ```sh
    Name: << inventory_name >> repository_export_inventory
    Organization: Ministry of Community and Social Services
    
    Save the Inventory
    ```
    ![alt text][png002]    


1.  Click on the Sources tab, and create a new source
    ```sh
    Name: my_inventory_source
    Source:  Sourced from a Project
    Project: "fcms github - dev"
    Inventory File: inventories/siebel_ent1/hosts
    ```
    ![alt text][png003]    

1.  Update Options
    ```sh
    Overwrite: checked
    Overwrite Variables: checked
    Update on Launch: unchecked
    Update on Project Update: checked

    Save the Inventory
    ```

1.  Start Sync Process
    ```sh
    Click on Sources Tab
    Click on Synch All
    
    Wait for the process to complete, and verify the sync completed
    ```


1.  Verify the Sync Process was successfull
    ```sh
    Click on Hosts Tab
    Verify the hosts were imported, and have a description of "Imported"
    ```
    ![alt text][png004]    

1.  Verify the Sync Process was successfull
    ```sh
    Click on Details Tab
    Verify the Variables were populated
    ```
    ![alt text][png005]    


#### Create a new Job Template

1.  Login as a Superuser. Open a web browser and log in to Tower by browsing to the Tower server URL
    ```sh
    http://129.213.21.160
    https://<Tower server name>/
    Username: admin
    Password: password
    ```
    ![alt text][png001]    

1.  Navigate to the template on AWX/Ansible Tower console and click on “+” to create a new Job Template
    ```sh
    Enter the required template details.  (*)
    
    Name: Siebel Migration Repository Export
    Job Type: Run
    Inventory: repository_export_inventory
    Project: "fcms github - dev"
    Playbook: siebel-migration-plan-repository-export.yml
    Credential: opc-oci-ansible-key-hvelez
    
    Save the Template
    ```
    ![alt text][png006]    


#### Launch a Job Template

1.  Login as a Superuser. Open a web browser and log in to Tower by browsing to the Tower server URL
    ```sh
    http://129.213.21.160
    https://<Tower server name>/
    Username: admin
    Password: password
    ```
    ![alt text][png001]    

1.  Launch a job template
    ```sh
    Access the job template list from the templates-icon navigational link to access the launch button from the list of templates.
    While in the Job Template List of the job template you want to launch, click Launch.

    A job may require additional information to run. The following data may be requested at launch:

    - Verbosity: Level of Output
    ```
    ![alt text][png007]    
    ![alt text][png009]    
    
1.  Verify Job Status
    ```sh
    Once the Job is complete, verify the status is succesful
    ```
    ![alt text][png008]    

### Verify the Job was Completed Successfuly

#### Verify from Siebel Migration Application

1.  Login into Siebel Migration Application. Open a web browser and log in to Siebel Migration
    ```sh
    https://129.213.113.88/migration
    User ID: SADMIN
    Password: SADMIN2017
    ```

1.  Siebel Migration Execution Plan
    ```sh
    Click on the History Tab
    Verify the Status is Success for the Job DDIT_DROP_19_2_N_2
    ```

1.  Siebel Migration Execution Plan Details
    ```sh
    Click on the History Tab
    Expand the Job DDIT_DROP_19_2_N_2
    Verify status is Success for the following:
    - Schema Service - Export
    - Runtime Repository Data Service - GetWatermark
    - Runtime Repository Data Service - Export
    - Application Workspace Data Service - GetFullSeedWatermark
    - Application Workspace Data Service - FullSeedExport
    ```

#### Verify Export File Was Created on Source Siebel File System

1.  Login into Siebel Server Source machine
    ```sh
    129.213.113.88
    user: opc
    ssh: private key
    ```

1.  Navigate to the siebel filesystem directory
    ```sh
    cd /var/lib/awx/projects/_35__fcms_dev/roles/siebel-mig-exec-export-post/files
    ls -alh DDIT_DROP_19_2_N_2_export*.zip

    162M Oct 30 17:52 DDIT_DROP_19_2_N_2_export.zip
    162M Oct 30 17:53 DDIT_DROP_19_2_N_2_export.zip_2019-10-30-17:53:06
    ```

#### Verify Export File Was Created on Ansible Main Controller

1.  Login into Ansible Controller Server
    ```sh
    129.213.21.160
    user: ansible
    ssh: private key
    ```

1.  Navigate to the directory where the files are stored, and verify the export file exists
    ```sh
    cd /var/lib/awx/projects/_35__fcms_dev/roles/siebel-mig-exec-export-post/files
    ls -alh DDIT_DROP_19_2_N_2_export.zip

    162M Oct 30 17:53 DDIT_DROP_19_2_N_2_export.zip
    ```




[//]: # (This syntax works like a comment, and won't appear in any out)
[//]: # (Here goes the reference for the images)

[png001]: images/ansible_tower_login.png "Ansible Tower Login Page"
[png002]: images/ansible_tower_inventories_create_inventory.png  "Create Inventory"
[png003]: images/ansible_tower_inventories_create_source.png "Create Inventory Source"
[png004]: images/ansible_tower_inventories_hosts.png "Inventory Hosts"
[png005]: images/ansible_tower_inventories_details.png "Inventory Details"
[png006]: images/ansible_tower_templates_new_template.png "Create Job Template"
[png007]: images/ansible_tower_template_repository_export_launch.png "Start a Job"
[png008]: images/ansible_tower_template_repository_export_status.png "Job Status Details"
[png009]: images/ansible_tower_template_repository_export_launch_launch.png "Launch a Job"
