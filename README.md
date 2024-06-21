# Trilio Ansible Playbook for Pipeline Operator Backups
Set of Ansible Roles and Playbooks for backing up a large number of Pipeline Operator namespaces

# Use Case
The intended use case for using this playbook is when there is an OpenShift cluster using the Pipelines Operator to deploy applications via CI/CD.   Trilio can backup an Operator and it's extended Custom Resources using an application backup.  The extra CRs are added to the backup plan.

The demo example is the following:

OpenShift cluster with Pipelines Operator installed.
Pipelines deployed in the following namespaces:
- pipelines-1
- pipelines-2
- pipelines-3
- pipelines-4
- pipelines-5

In order to capture the pipeline, you need to create an application backup for the pipeline operator plus the CRs - Pipeline and Task.  Once you run the pipeline, other CRs get crated like PipelineRun and TaskRun, we do not need to back these up.

An application backup can currently only contain extra CRs from one specified namespace.  Therefore, the backup all of the reources in this demo, you need to create 5 application backupplans and 5 backups. This playbook will then create 5 applicaton backupplans and backups for the 5 namespaces.  This playbook can also automate the restore process by iterating through the restores.

You need to adjust the config file for which actions the ansible scripts will perform.

To create the backupplans and backups:
```yaml
// Choose actions to take  
    trilio_kubernetes_create_backupplan: true
    trilio_kubernetes_create_backup: true
    trilio_kubernetes_create_restore: false
```
To create additional backups:
```yaml
// Choose actions to take  
    trilio_kubernetes_create_backupplan: false
    trilio_kubernetes_create_backup: true
    trilio_kubernetes_create_restore: false
```
To create the restores:
```yaml
// Choose actions to take  
    trilio_kubernetes_create_backupplan: false
    trilio_kubernetes_create_backup: false
    trilio_kubernetes_create_restore: true
```

The script is designed to run the backups in batches for scalability purposes.   The creation of Trilio backups consumes cluster resoucres because each backup creates one metamover pod and one datamover pod per PVC to be backed up.  These pods are temporary resources that only exist at the time of backup and then clean themselves up when the the backup is complete.  The amount of simultaneous backups you can run at once will be cluster specific based on available free CPU/Mem resoucres.   The default batch size is 2 but this can be modified in the config file.


# Steps to setup:
- untar the zip first
- install ansible on your local setup
- install kubernetes.core package using command `ansible-galaxy collection install kubernetes.core`
- after switch to playbook dir inside ansible-citi using `cd ansible-citi/playbooks`
- update trilio-utility.yaml role file location to your playbooks directory
   for example:
   ```yaml
   roles:
     - /Users/jeffligon/Documents/Demo-Platforms/demo-system-yamls/ansible/ansible-citi/roles/trilio_kubernetes
   ```
- update trilio_kubernetes-config.yaml according to your configuration.
    for example:
    ```yaml
    // Username/Pass
    trilio_kubernetes_username: kubeadmin
    trilio_kubernetes_password: g9Q23-w47H7-ztBTF-qGNY8
    trilio_kubernetes_auth_api: https://api.dev.staging.presales.trilio.io:6443
    trilio_kubernetes_host: https://api.dev.staging.presales.trilio.io:6443 
     
    // Backup Target
    trilio_kubernetes_target_name: nfs-sa
    trilio_kubernetes_target_namespace: trilio-system

    // Backup Plan Details
    trilio_kubernetes_backup_namespaces:
      - pipelines-1
      - pipelines-2
      - pipelines-3
      - pipelines-4
      - pipelines-5
    
    // Choose actions to take  
    trilio_kubernetes_create_backupplan: true
    trilio_kubernetes_create_backup: true
    trilio_kubernetes_create_restore: false
    ```
- then finally run:
```bash
ansible-playbook trilio-utility.yaml 
```
to execute ansible playbook.


