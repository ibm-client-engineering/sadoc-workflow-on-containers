The following documents the process for installing BAW standalone on a customer hybrid environment running Openshift.

### **Customer environment**

The customer was running Openshift in a shared environment that was provisioned via Openstack.

- OCP version: 4.8.x
- Number of worker nodes was set to 2 via node selector tag.
- Customer was using manila for file storage

### **Requirements**

BAW Standalone needed to run in a dedicated namespace. Customer also required the reuse of the existing certificate manager that was already running on the cluster. It was community edition that the customer upgraded to 1.7.1.

Customer requested an airgapped installation at first and configured Artifactory on an external vm to act as a docker proxy. This was then configured in OCP with an imagecontentpolicy setting.

### **BAW Standalone for Openshift requirements**

1. Must have required databases available and configured. This can be DB2, Oracle, etc. Customer in our case is using Oracle.
2. LDAP for authentication. Can also be OpenLDAP, IBM SDS, or Active Directory. Customer in our case is using Active Directory.

### **Steps followed to install BAW Standalone into client environment**

1. Customer configured an external VM running linux and installed the following tools:
    - kubectl
    - oc
    - podman
    - yq
2. Downloaded the CASE archive file for BAW standalone from https://github.com/IBM/cloud-pak/tree/master/repo/case/ibm-cs-bawautomation/2.2.5
3. On OCP, a new project was created dedicated to BAW standalone and customer user was set to be admin for the project.

### **Installation process**

### **Creating an LDAP container**

This does not reflect what was running in the customer environment. Rather this was a utility container running openldap that we spun up in order to test the installation of BAW Standalone in a separate env.

The exact instructions for creating an openldap pod are documented here:

https://github.ibm.com/kramerro/openldap-on-ocp

Clone the above repo somewhere.

Login to your OCP cluster with the `oc` client and create the following service account and policy settings:
```
oc create serviceaccount openldap -n <CUSTOMPROJECTNAME>
oc adm policy add-scc-to-user privileged system:serviceaccount:<CUSTOMPROJECTNAME>:openldap
oc adm policy add-scc-to-user anyuid system:serviceaccount:<CUSTOMPROJECTNAME>:openldap
```

Go into the repo you cloned above and set the `namespace:` in the following files to match your project name:

- `createimagestream.yaml`
- `buildconfig.yaml`
- `ldap-service.yaml`

In `ldap-deployment.yaml` you'll need to modify the `image:` entry to point to your correct namespace:

```
image: image-registry.openshift-image-registry.svc:5000/<CUSTOMPROJECTNAME>/openldap

```

Now create the image stream

```
oc create -f createimagestream.yaml
```
Kick off the custom build of the ldap image

```
oc start-build openldap -n <CUSTOMPROJECTNAME>
```
Now deploy an ldap instance
```
oc create -f ldap-deployment.yaml -n <CUSTOMPROJECTNAME>
```
Deploy an LDAP service

```
oc create -f ldap-service.yaml -n <CUSTOMPROJECTNAME>
```

Our ldap service should be available on openldap-service.ldap at ports 389 and 636. It should also have the necessary users for the baw installation.

### **Install DB2**

To be filled out

### **BAW Standalone Installation**

Extract the file `ibm-cs-bawautomation-2.2.5.tgz` somewhere

```
cd ibm-cs-bawautomation/inventory/cp4aOperatorSdk/files/deploy/crs/

tar xvf 21.0.3-IF009.tar

cd cert-kubernetes/scripts

./cp4a-clusteradmin-setup.sh baw
```

Answer the relevant install questions and set the namespace to be the dedicated customer namespace as previously defined on OCP.

This installs the ibm-cp4a-operator in that namespace.

### **Updating the CR**

The customer had a previously defined CR to use for deployment as they had previously deployed CP4BA on ICP.