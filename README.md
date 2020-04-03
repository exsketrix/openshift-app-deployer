# openshift-app-deployer

This repository contains an Ansible role that provides a means for deploying applications to OpenShift. The rationale behind the creation of this repository stemmed from issues with using the in-built k8s Ansible module. Whilst this module has now been made the official module to be used for deploying to both Kubernetes and OpenShift, there are still a few issues that prevents it from working in all situations. This appears to include:

* Attempting to use the lookup function for retrieving the status and content of objects in Kubernetes/OpenShift where the instance uses a self signed cert e.g. 

```yaml
- name: Fetch all deployments in a namespace
  set_fact:
    deployments: "{{ lookup('k8s', kind='Deployment', namespace='myapp-dev', host='https://console.openshift.bizchat.io:8443', api_key='{{openshift_app_deployer_token}}', verify_ssl=no) }}"
```

* There appears to be a compatibility issue with Ansible v2.9, openshift python library v0.11.0, kubernetes python library v11.0.0 with OKD v3.11 which prevents the k8s module from being able to successfully access the server API (error message received is: 'the server could not find the requested resource').

For these reasons, this Ansible role works around these version compatibility issues through deploying resources using the OpenShift Client (oc) utility.

## Requirements

Requirements to run this Ansible role:

* Docker installed
* Ansible v2.6+
* An account on OpenShift with at least deployer access to a project/namespace

## Variables

This role performs validation to check that all mandatory variables are populated. The following table lists each variable that can be set for this role 

| Variable Name                                   |  Required | Example Value                                              |
| ----------------------------------------------- | --------- | ---------------------------------------------------------- |
| openshift_app_deployer_files_resource_path      | Y         | "{playbook_dir}/files"                                     |
| openshift_app_deployer_templates_resource_path  | Y         | "{playbook_dir}"/templates"                                |
| openshift_app_deployer_url                      | Y         | https://console.openshift.bizchat.io:8443                  |
| openshift_app_deployer_user_name                | N*        | openshift-user                                             |
| openshift_app_deployer_user_passwd              | N*        | <openshift_user_password>                                  |
| openshift_app_deployer_token                    | N*        | <openshift_access_token>                                   |
| openshift_app_deployer_project                  | Y         | myapp-dev                                                  |
| openshift_app_deployer_resources                | Y         | <p>- myapp_imagestream.yml<br>- myapp_service.yml</p>      |

`*` Either openshift_app_deployer_user_name and openshift_app_deployer_user_passwd are required or openshift_app_deployer_token if you have an access token