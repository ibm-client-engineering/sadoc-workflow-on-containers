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