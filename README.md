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

Some information comes from https://github.ibm.com/matthias-benda-ibm/tech-log/tree/master/cp4a/baw21.0.3-IKS

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

#### Download DB2 product sources

Download the following packages for DB2 (internally from IBM Internal DSW Downloads or externally from Passport Advantage):

* IBM DB2 Advanced Workgroup Server Edition Server Restricted Use V11.1 for Linux on AMD64 and Intel EM64T systems (x64) Multilingual (CNB8FML)
* IBM DB2 Advanced Workgroup Server Edition Restricted Use Activation V11.1 for Linux, UNIX and Windows Multilingual (CNB21ML)
* The latest [DB2 FixPack](https://www.ibm.com/support/pages/download-db2-fix-packs-version-db2-linux-unix-and-windows)

Conntect to the server (as root) and create a directory to store the sources:

```
mkdir ~/install_sources
```

Transfer (via SCP) the packages downloaded to the new directory and extract them there. If `unzip` is not installed, install it using `yum install unzip`.

```
cd  ~/install_sources
tar xzf DB2_AWSE_REST_Svr_11.1_Lnx_86-64.tar.gz
unzip DB2_AWSE_Restricted_Activation_11.1.zip
mkdir db2fp
tar xzf v11.1.4fp5_linuxx64_server_t.tar.gz -C db2fp
```

#### Install DB2

```shell
#install prereqs
yum install pam.i686 libaio gtk2.i686 libXtst.i686 libstdc++.so.6 ksh binutils

#install base version
cd ~/install_sources/server_awse_o/
./db2_install -b /opt/ibm/db2/V11.1 -p server -n

#add licence
/opt/ibm/db2/V11.1/adm/db2licm -a ~/install_sources/awse_o/db2/license/db2awse_o.lic

#install fixpack
cd ~/install_sources/db2fp/server_t
./installFixPack -b /opt/ibm/db2/V11.1 -p /opt/ibm/db2/V11.1
```

#### Create Instance for BAW

```shell
groupadd db2baw
useradd -g db2baw -d /home/bawinst bawinst
passwd bawinst
useradd -g db2baw -d /home/bawfenc bawfenc
passwd bawfenc

/opt/ibm/db2/V11.1/instance/db2icrt -a SERVER -s ese -p 50000 -u bawfenc bawinst

su - bawinst

db2 update dbm cfg using SVCENAME db2c_bawinst

db2iauto -on bawinst
db2start
exit
```

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