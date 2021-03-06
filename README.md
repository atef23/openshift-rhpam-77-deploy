
# Deploying a RHPAM 7.7 in Openshift

**Requirements:**
- Openshift CLI
- Openshift Cluster with namespace created for deployment

Run the following commands to deploy an RHPAM 7.7 Authoring Environement based on the default Red Hat templates:

## Create a New Namespace

Run the following commands to deploy an RHPAM 7.7 Authoring Environement based on the default Red Hat templates:

    oc new-project rhpam-77
## Enable Red Hat Registry Access to Pull Images
If your cluster already has access to pull images from the Red Hat Registry skip this section, otherwise complete the following steps to enable Registry access (see [https://access.redhat.com/RegistryAuthentication] for more options on how to enable access):

1. Go to [https://access.redhat.com/terms-based-registry/](https://access.redhat.com/terms-based-registry/) and create a new service account or use an existing one
2. Click on the "Openshift Secret" tab under your service account
3. download the pull secret and run the following command to create it in your namespace: `oc create -f youruser-secret.yaml`

Now that the rhpam-77 namespace has your registry service account pull secret, you will be able to pull Red Hat images into your namespace. Alternatively, if the cluster is configured to pull images into the "openshift" or "default" namespace, you can import image streams into those namespaces and then pull them from there into "rhpam-77" for use in deployments.

## Create the Application
Once registry access is enabled run the following commands to deploy the Authoring template:

    oc apply -f rhpam77-image-streams.yaml
    
    oc apply -f templates/kieserver-app-secret-template.yml
    
    oc apply -f templates/businesscentral-app-secret-template.yml
    
    oc create secret generic rhpam-credentials --from-literal=KIE_ADMIN_USER=adminUser --from-literal=KIE_ADMIN_PWD=adminPassword
    
For Business Central with Git Hooks enabled, modify and add the following config maps to the namespace:

	oc create -f templates/git-post-commit-script-template.yaml
	oc create -f templates/git-post-commit-default-config-template.yaml


Business Central Authoring Environment:

    oc new-app -f templates/rhpam77-authoring.yaml -p BUSINESS_CENTRAL_HTTPS_SECRET=businesscentral-app-secret \
	 -p KIE_SERVER_HTTPS_SECRET=kieserver-app-secret \
	 -p APPLICATION_NAME=rhpam-77-authoring-env \
	 -p CREDENTIALS_SECRET=rhpam-credentials \
	 -p DB_VOLUME_CAPACITY=2Gi \
	 -p IMAGE_STREAM_NAMESPACE=rhpam-77 \
	 -p KIE_SERVER_IMAGE_STREAM_NAME=rhpam-kieserver-rhel8 \
	 -p IMAGE_STREAM_TAG=7.7.0 \
	 -p BUSINESS_CENTRAL_VOLUME_CAPACITY=2Gi \
	 -p GIT_HOOKS_DIR=/home/jboss/git-hooks

Immutable Kie-Server:

	 oc new-app -f templates/rhpam77-prod-immutable-kieserver.yaml \
	  -p KIE_SERVER_HTTPS_SECRET=kieserver-app-secret \
	  -p APPLICATION_NAME=rhpam-77-immutable-env \
	  -p CREDENTIALS_SECRET=rhpam-credentials \
	  -p DB_VOLUME_CAPACITY=2Gi \
	  -p IMAGE_STREAM_NAMESPACE=rhpam-kie-77 \
	  -p KIE_SERVER_IMAGE_STREAM_NAME=rhpam-kieserver-rhel8 \
	  -p IMAGE_STREAM_TAG=7.7.0 \
	  -p POSTGRESQL_MAX_PREPARED_TRANSACTIONS=100 \
	  -p KIE_SERVER_POSTGRESQL_DIALECT=org.hibernate.dialect.PostgreSQLDialect \
	  -p KIE_SERVER_CONTAINER_DEPLOYMENT=rhpam-kieserver-test=com.myspace:kjar-test-runner:1.0.0-SNAPSHOT \
	  -p SOURCE_REPOSITORY_URL=https://github.com/atef23/test-empty-bc-project \
	  -p KIE_SERVER_MGMT_DISABLED=true

Under "routes" open the secure Business Central route and login with user/password adminUser/adminPassword

