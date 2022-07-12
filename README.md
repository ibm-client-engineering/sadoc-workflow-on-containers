The following documents the process for installing BAW standalone on a customer hybrid environment running Openshift.

### Customer environment

The customer was running Openshift in a shared environment that was provisioned via Openstack.

<ENV BACKGROUND HERE>

### Requirements

BAW Standalone needed to run in a dedicated namespace. Customer also required the reuse of the existing certificate manager that was already running on the cluster. It was community edition that the customer upgraded to 1.7.1.

Customer requested an airgapped installation at first and configured Artifactory on an external vm to act as a docker proxy. 

### BAW Standalone for Openshift requirements

1. Must have required databases available and configured. This can be DB2, Oracle, etc. Customer in our case is using Oracle.
2. LDAP for authentication. Can also be OpenLDAP, IBM SDS, or Active Directory. Customer in our case is using Active Directory.