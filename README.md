The followind documents the process for installing BAW standalone on a customer hybrid environment running Openshift.

### Customer environment

The customer was running Openshift in a shared environment that was provisioned via Openstack.

<ENV BACKGROUND HERE>

### Requirements

BAW Standalone needed to run in a dedicated namespace. Customer also required the reuse of the existing certificate manager that was already running on the cluster. It was community edition that the customer upgraded to 1.7.1.