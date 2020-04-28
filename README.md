# how to install Oracle Instant Client and SD on Ubuntu20.04

## Install Oracle Instant Client and SDK
### Step 1

Download the Oracle Instant Client and SDK from Oracle website.
(Need to login in Oracle page)

[http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)

Files: `instantclient-basic-linux.x64-19.6.0.0.0dbru.zip` and `instantclient-sdk-linux.x64-19.6.0.0.0dbru.zip`.

### Step 2

Create a new folder to store Oracle Instant Client zip files on your server.

Upload the Instant Clients files inside this folder.

```
mkdir /opt/oracle
```

### Step 3

Now we need move and extract the files.

```
sudo mv ~/Downloads/instantclient-*.zip /opt/oracle/

cd /opt/oracle

unzip instantclient-basic-linux.x64-19.6.0..0dbru.zip
unzip instantclient-sdk-linux.x64-19.6.0..0dbru.zip
```

### Step 4

Next, we need to create a symlink to Instant Client files(obs: maybe not necessite)

```
ln -s /opt/oracle/instantclient_19_2/libclntsh.so.19.1 /opt/oracle/instantclient_19_6/libclntsh.so
ln -s /opt/oracle/instantclient_12_2/libocci.so.19.1 /opt/oracle/instantclient_19_6/libocci.so
```

### Step 5

Add the folder to our `ldconfig`.

```
echo /opt/oracle/instantclient_19_6 > /etc/ld.so.conf.d/oracle-instantclient
```

### Step 6

Update the Dynamic Linker Run-Time Bindings

```
ldconfig
```

Done. Now we can proceed to the next part.


## Install Additional Packages

To install the OCI8 extension, we need to install some additional package on our server.

### Step 1

Run these command:

```
apt-get install php-dev php-pear build-essential libaio1
```

### Step 2

Once installed, we need to get the OCI8 file.
But, before that we need to update PECL channel.

```
pecl channel-update pecl.php.net

```
Then.

```
pecl install oci8
```


When you are prompted for the Instant Client location, enter the following:

```
instantclient,/opt/oracle/instantclient_19_6
```

### Step 3

We need to tell PHP to load the OCI8 extension.

```
sudo su sudo su
echo "extension =oci8.so" >> /etc/php/7.4/fpm/php.ini
echo "extension =oci8.so" >> /etc/php/7.4/cli/php.ini
echo "extension =oci8.so" >> /etc/php/7.4/apache2/php.ini

exit
```

We also need to add on apache those environment variables.

```
sudo su 

echo "export LD_LIBRARY_PATH=/opt/oracle/instantclient_19_6" >> /etc/apache2/envvars
echo "export ORACLE_HOME=/opt/oracle/instantclient_19_6" >> /etc/apache2/envvars

echo "LD_LIBRARY_PATH=/opt/oracle/instantclient_19_6:$LD_LIBRARY_PATH" >> /etc/environment

exit

```

### Step 4

Refresh the server. If you are accessing through SSH, then
```
exit
```
or
```
sudo shutdown -r now
```

Check if the extension is enabled.

```
php -m | grep 'oci8'
```

If returns `oci8`, its works!

### Step 5

Restart the PHP-FPM

```
service php7.4-fpm restart
```

Now you can connect to Oracle DBMS from your PHP applications.
