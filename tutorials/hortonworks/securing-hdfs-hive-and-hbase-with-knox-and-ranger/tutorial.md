---
layout: tutorial
title: Manage Scurity Policy for Hive and HBase with Knox and Ranger
tutorial-id: 410
tutorial-series: Security
tutorial-version: hdp-2.4.0
intro-page: true
components: [ knox, ranger ]
---

##  Introduction

Apache Ranger delivers a comprehensive approach to security for a Hadoop cluster. It provides a central security policy administration across the core enterprise security requirements of authorization, accounting and data protection.

Apache Ranger already extends baseline features for coordinated enforcement across Hadoop workloads from batch, interactive SQL and real–time in Hadoop.

In this tutorial, we cover using Apache Ranger for HDP 2.4 to secure your Hadoop environment. We will walkthrough the following topics:

1.  Support for Knox authorization and auditing
2.  Command line policies in Hive
3.  Command line policies in HBase
4.  REST APIs for policy manager

## Prerequisites

The only prerequisite for this tutorial is that you have a running instance of the [Hortonworks Sandbox](http://hortonworks.com/sandbox) via [Virtualbox](http://hortonworks.com/sandbox) or [Azure](http://hortonworks.com/hadoop-tutorial/deploying-hortonworks-sandbox-on-microsoft-azure/).

## Background

Ranger is a great tool to secure your data in Hadoop and HDP. It offers a simple centralized way to manage users' access to data in your cluster. Cluster administrators can easily manage policies for access to files folder, databases, tables, and more! In this tutorial we'll explore how we can use Ranger to manage access to these types of resources in the [Hortonworks Sandbox](http://hortonworks.com/sandbox)

In this tutorial we will also utilize Apache Knox which is a REST API gateway for Hadoop services like HDFS and Hive! It allows a user to plug-and-play with already existing LDAP and Microsoft AD infrastructure to manage access to Hadoop services. Not to mention it supports Kerberized clusters as well as utilizes HTTPS protocol for its communications.

## Procedure

First you're going to need to log in to your Sandbox via SSH.

If you're using Virtualbox you can log in with the command

	ssh root@127.0.0.1 -p 2222

If you're on Azure, change `127.0.0.1` to the IP address of your Azure machine.

**The password to log in is**: `hadoop`


![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-02 14.59.03.png)

### Starting Knox Service and Demo LDAP Service

Open up the Ambari user interface by using the URL `http://_host_:8080`

Using Virtualbox it might look like [http://127.0.0.1:8080](http://127.0.0.1:8080). If you're using Azure make sure to replace `127.0.0.1` with you host machine's IP address.

Login to Ambari using the following:

| Username | Password |
|:--------:|:--------:|
|  admin   | 4o12t0n  |


After logging in to Ambari, select **Knox** from the list of `Services` on the left-hand side of the page.  

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.17.05.png)

Then click on `Service Actions` from the top right hand side of the page click on `Start`. If you want to see 

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.27.06.png)

From the following you can track the start of the Knox service to completion:

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.20.01.png)

Then go back to the `Service Actions` button on the `Knox` service and click on `Start  Demo LDAP`. This LDAP server is when authenticating users against Knox in the Sandbox because there is no other LDAP server running on the Sandbox.

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.37.09.png)

You can track the start of the `Demo LDAP Service` from the following screen:

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.39.01.png)

### Knox Access Scenarios

Check if Ranger Admin console is running, at [http://localhost:6080/](http://localhost:6080/)from your host machine. The username is `admin` and the password is `admin`

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.11.01.png)

<!--
If it is not running you can start from the command line using the command

~~~
sudo service ranger-admin start
~~~

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-02 15.06.00.png)
-->

Click on sandbox_knox link under Knox section in the main screen of Ranger Administration Portal

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.30.58.png)

You can review policy details by a clicking on the policy name.

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.31.31.png)

To start testing Knox policies, we would need to turn off the "global knox allow" policy.  Edit the policy using the button on the right hand side and use the first slider to **disable** the policy.

Then, click **Save**

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.33.53.png)

Locate `Sandbox for Guest` policy on the Ranger Admin console and edit the policy

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.45.17.png)

and use the slider to enable the policy named "Sandbox for Guest". Click **Save**.

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.47.28.png)

From your local SSHd terminal (not directly on the Sandbox), run this CURL command to access WebHDFS

`curl -k -u admin:admin-password 'https://127.0.0.1:8443/gateway/knox_sample/webhdfs/v1?op=LISTSTATUS'`

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.50.43.png)

Go to Ranger Policy Manager tool →  Audit screen and check the knox access (denied) being audited.

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.53.47.png)

Now let us try the same CURL command using “guest” user credentials from the terminal

`curl -k -u guest:guest-password 'https://127.0.0.1:8443/gateway/knox_sample/webhdfs/v1?op=LISTSTATUS'`

The result which should return is a set of JSON data from WebHDFS which lists the status of the files in the root of HDFS

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.55.29.png)

    {"FileStatuses":{"FileStatus":[{"accessTime":0,"blockSize":0,"childrenNum":0,"fileId":16393,"group":"hadoop","length":0,"modificationTime":1439987528048,"owner":"yarn","pathSuffix":"app-logs","permission":"777","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":4,"fileId":16389,"group":"hdfs","length":0,"modificationTime":1439987809562,"owner":"hdfs","pathSuffix":"apps","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":17000,"group":"hdfs","length":0,"modificationTime":1439989173392,"owner":"hdfs","pathSuffix":"demo","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":16398,"group":"hdfs","length":0,"modificationTime":1439987529660,"owner":"hdfs","pathSuffix":"hdp","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":16394,"group":"hdfs","length":0,"modificationTime":1439987528532,"owner":"mapred","pathSuffix":"mapred","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":2,"fileId":16396,"group":"hadoop","length":0,"modificationTime":1439987538099,"owner":"mapred","pathSuffix":"mr-history","permission":"777","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":16954,"group":"hdfs","length":0,"modificationTime":1439988741413,"owner":"hdfs","pathSuffix":"ranger","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":3,"fileId":16386,"group":"hdfs","length":0,"modificationTime":1440165443820,"owner":"hdfs","pathSuffix":"tmp","permission":"777","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":8,"fileId":16387,"group":"hdfs","length":0,"modificationTime":1439988397561,"owner":"hdfs","pathSuffix":"user","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"}]}}

We can check the auditing in the Ranger Policy Manager → Audit screen

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.56.45.png)

Ranger plugin for Knox intercepts any request made to Knox and enforces policies which are retrieved from the Ranger Administration Portal

You can configure the Knox policies in Ranger to restrict to a specific service (WebHDFS, WebHCAT etc) and to a specific user or a group and you can even bind user/group to an ip address

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 10.59.43.png)

### Hive grant/revoke permission scenarios

Ranger supports the import of grant/revoke policies set through command line for Hive. 

Ranger can store these policies centrally along with policies created in the administration portal and enforce it in Hive using its plugin.

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 11.01.23.png)

As a first step, disable the global access policy for Hive in Ranger Administration Portal

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 11.03.30.png)

Let us try running a Grant operation using user Hive from the command line. Login into beeline tool using the following command

	beeline -u "jdbc:hive2://sandbox.hortonworks.com:10000/default"  -n it1 -p it1-d org.apache.hive.jdbc.HiveDriver`

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 11.05.44.png)

Then issue the `GRANT` command

	grant select, update on table xademo.customer_details to user network1;

If you correctly disabled the Hive policies which allowed this command you should see the following error:

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 11.07.14.png)

If you don't see the error, make sure you've disabled the correct Hive policies.

Let's check the audit log in the Ranger Administration Portal → Audit

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 11.08.52.png)

You can see that access was denied for an admin operation for user it1.

We can create a policy in Ranger for user ‘it1’ to be an admin. Create a new policy from the Ranger Admin console and ensure the configuration matches the illustration below

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/hive-grant-revoke-1.png)

We can try the beeline command again, once the policy has been saved.

	GRANT select, update on table xademo.customer_details to user network1;

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 13.00.35.png)

If the command goes through successfully, you will see the policy created/updated in Ranger Admin Portal → Policy Manager. It checks if there is an existing relevant policy to update, else it creates a new one.  

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/hive-policy-view.png)

What happened here?

Ranger plugin intercepts GRANT/REVOKE commands in Hive and creates corresponding policies in Admin portal. The plugin then uses these policies for enforcing Hive authorization (Hiveserver2)

Users can run further GRANT commands to update permissions and REVOKE commands to take away permissions.

### HBase grant/revoke permission scenarios

Ranger can support import of grant/revoke policies set through command line in Hbase. Similar to Hive, Ranger can store these policies as part of the Policy Manager and enforce it in Hbase using its plugin.

Before you go further, ensure HBase is running from Ambari – [http://127.0.0.1:8080](http://127.0.0.1:8080) (username is `admin` and password is `4o12t0n`).

If it is not go to `Service  Actions` button on top right and `Start` the service

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 13.10.32.png)

As a first step, let us try running a Grant operation using user Hbase.

Disable the public access policy “HBase Global Allow” in Ranger Administration Portal – policy manager

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 13.13.08.png)

Login into HBase shell as ‘it1’ user

    su it1

    [it1@sandbox ~]$ hbase shell

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 13.14.33.png)

Run a grant command to give “Read”, “Write”, “Create” access to user mktg1 in table ‘iemployee’

	grant 'mktg1',  'RWC',  'iemployee'

	hbase(main):001:0> grant 'mktg1',  'RWC',  'iemployee'

you should get a Acess Denied as below:

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 14.10.21.png)

Go to Ranger Administration Portal → Policy Manager and create a new policy to assign “admin” rights to user it1

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 14.13.46.png)

Save the policy and rerun the HBase command again

    hbase(main):006:0> grant 'mktg1', 'RWC', 'iemployee'

    0 row(s) in 0.8670 seconds

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 14.14.27.png)

Check HBase policies in the Ranger Policy Administration portal. The grant permissions were added to an existing policy for table ‘iemployee’ that we created in previous step

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 14.15.44.png)

You can revoke the same permissions and the permissions will be removed from Ranger admin. Try this in the same HBase shell

    hbase(main):007:0> revoke 'mktg1', 'iemployee'

    0 row(s) in 0.4330 seconds

You can check the existing policy and see if it has been changed

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 14.16.34.png)

What happened here?

Ranger plugin intercepts GRANT/REVOKE commands in Hbase and creates corresponding policies in the Admin portal. The plugin then uses these policies for enforcing authorization

Users can run further GRANT commands to update permissions and REVOKE commands to take away permissions.

### REST APIs for Policy Administration

Ranger policies administration can be managed through REST APIs. Users can use the APIs to create or update policies, instead of going into the Administration Portal.

### Running REST APIs from command line

From your local command line shell, run this CURL command. This API will create a policy with the name “hadoopdev-testing-policy2” within the HDFS repository “sandbox_hdfs”

    curl -i --header "Accept:application/json" -H "Content-Type: application/json" -u admin:4o12t0n -X POST http://127.0.0.1:6080/service/public/api/policy -d '{ "policyName":"hadoopdev-testing-policy2","resourceName":"/demo/data","description":"Testing policy for /demo/data","repositoryName":"sandbox_hadoop","repositoryType":"HDFS","permMapList":[{"userList":["mktg1"],"permList":["Read"]},{"groupList":["IT"],"permList":["Read"]}],"isEnabled":true,"isRecursive":true,"isAuditEnabled":true,"version":"0.1.0","replacePerm":false}'

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 14.18.43.png)

the policy manager and see the new policy named “hadoopdev-testing-policy2”

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/Screenshot 2015-09-08 14.21.07.png)

Click on the policy and check the permissions that has been created

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/policy-id-ranger-1.png)

The policy id is part of the URL of this policy detail page [http://127.0.0.1:6080/index.html#!/hdfs/1/policy/21](http://127.0.0.1:6080/index.html#!/hdfs/1/policy/21)

We can use the policy id to retrieve or change the policy.

Run the below CURL command to get policy details using API

	curl -i -u admin:4o12t0n -X GET http://127.0.0.1:6080/service/public/api/policy/21

![](/assets/securing-hdfs-hive-hbase-with-knox-ranger/policy-rest-api-get.png)

What happened here?

We created a policy and retrieved policy details using REST APIs. Users can now manage their policies using API tools or applications integrated with the Ranger REST APIs

Hopefully, through this whirlwind tour of Ranger, you were introduced to the simplicity and power of Ranger for security administration.
