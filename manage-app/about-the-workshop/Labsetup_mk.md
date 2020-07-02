#  Multitenant Workshop setup

## Connecting to workshop instance using Putty
There is a separate **[workshop](https://github.com/oracle/learning-library/blob/master/common/labs/generate-ssh-key/generate-ssh-keys.md#connecting-to-an-instance-using-putty)** on Oracle Cont

1.  Download and **[Install Putty](https://www.putty.org/)** and then open the PuTTY utility from the Windows start menu.   In the dialog box, enter the IP Address of your OCI Compute Instance.  This will be obtained from your instructor.

    ![](images/keylab-023.png " ")

2.  Under **Category** select **Connection** and then choose the **Data** field.  Enter the assigned username.  OCI instances will default to the username ```opc```.  Enter ```opc```.

    ![](images/keylab-024.png " ")

3.  Under **Category**, navigate to **Connection** - **SSH** and choose the **Auth** category.   Click on the **Browse** button and locate the ```Private Key file``` you created in the earlier step.   Click the Open button to initiate the SSH connection to your cloud instance.

    ![](images/keylab-025.png " ")

4. Click Session in the left navigation pane, then click Save in the Load, save or delete a stored session Step.

5.  Click **Yes** to bypass the Security Alert about the uncached key.

    ![](images/keylab-026.png " ")

6.  Connection successful.   You are now securely connected to an OCI Cloud instance.

    ![](images/keylab-027.png " ")

    You are now able to connect securely using the Putty terminal utility.   You can save the connection information for future use and configure PuTTY with your own custom settings.

## Connect from Mac
1. Run the following command to change the file permissions to 600 to secure the key. You can also set them to 400.

````
 chmod 600 labkey.ppk
 ````
2. Use the key to log in to the SSH client as shown in the following example, which loads the key in file labkey.ppk, and logs in as user to IP Address. Eg.
````
ssh -i lapkey.ppk oracle@192.237.248.66

````
## Run the Setup Scripts as oracle
if you connected with username  opc, then sudo to oracle user.
````
sudo su - oracle
````

````
<copy>
cd /home/oracle
wget https://objectstorage.us-phoenix-1.oraclecloud.com/n/oraclepartnersas/b/Multitenant/o/labs.zip
chown oracle:oinstall /home/oracle/labs.zip
unzip -o labs.zip
chmod -R +x /home/oracle/labs
/home/oracle/labs/multitenant/resetCDB.sh
</copy>
````
