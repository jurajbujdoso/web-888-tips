# web-888-using-ssh-key
The standard disk image does not have option to ssh to device using ssh key enabled.

# 1. adding your own ssh keys script
```
web-888:~# ls /media/mmcblk0p1/web-888.apkovl.tar.gz
/media/mmcblk0p1/web-888.apkovl.tar.gz


#standar file system is read only
web-888:~# mount -o remount,rw /media/mmcblk0p1/

web-888:~# mount|grep mmcblk
/dev/mmcblk0p1 on /media/mmcblk0p1 type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
```

##  2. Create a custom script
```
cd /media/mmcblk0p1


#create script     vi /media/mmcblk0p1/INSTALLER.SH

cd /media/mmcblk0p1
echo "Add package for rsyslog"
#apk add rsyslog
#echo "Prepare config"
#cp ./rsyslog.conf /etc
#echo "Restart rsyslog"
#/etc/init.d/rsyslog restart
mkdir -p /root/.ssh
#replace bewllow with your ssh key
echo "ssh-rsa AAAAB3------------NzaC1ycYOURsshKey--------------------------= YOUR@yourDevice" > /root/.ssh/authorized_keys
echo "PubkeyAuthentication yes" >> /etc/ssh/sshd_config
chmod 700 /root/.ssh
chmod 600 /root/.ssh/*
/etc/init.d/sshd restart


chmod +rx /media/mmcblk0p1/INSTALLER.SH

```

##  3. Make script accessible

### 4. Backup original files

```
mkdir MODIFICATION
cp /media/mmcblk0p1/web-888.apkovl.tar.gz ./MODIFICATION/
cp /media/mmcblk0p1/web-888.apkovl.tar.gz /media/mmcblk0p1/web-888.apkovl.tar.gz-original
```


### 5. Modify for script processing
```
cd MODIFICATION
#switch uset to be ROOT !!!

tar -xzvf /media/mmcblk0p1/web-888.apkovl.tar.gz
rm /media/mmcblk0p1/web-888.apkovl.tar.gz
```


### 6. we have to make install script available by adding it in prereq section
```
vi ./etc/init.d/sdrd

#just add 4 last lines of code inside /etc/init.d/sdrd    DO NOT REMOVE ANY CODE WHICH IS THERE
start_pre() {
    cp -fu /media/mmcblk0p1/websdr.bin /root/websdr.bin
    
    cp -fu /media/mmcblk0p1/INSTALLER.sh /root/INSTALLER.sh
    chown -R root:root /root/
    chmod u+s /root/INSTALLER.sh    
    /root/INSTALLER.sh
  
```

### 7. Prepare working solution
```
cd  /media/mmcblk0p1/MODIFICATION/
tar -czvf mmcblk0p1/web-888.apkovl.tar.gz .
mv web-888.apkovl.tar.gz /media/mmcblk0p1/web-888.apkovl.tar.gz

#check if file is latest
ls -lt /media/mmcblk0p1/web-888.apkovl.tar.gz


sync;sync
mount -o remount,ro /media/mmcblk0p1/
#reboot the changes will take effect
reboot

```


# Modify of rssylog.conf
If you want to use rsyslog, then uncoment block bellow in step 2. and create file rsyslog.conf 


#### Uncomment
```
#apk add rsyslog
#echo "Prepare config"
#cp ./rsyslog.conf /etc
#echo "Restart rsyslog"
#/etc/init.d/rsyslog restart
```

#### Create rsyslog.conf
```
cd /media/mmcblk0p1

cp /etc/rsyslog.conf ./

vi /media/mmcblk0p1/rsyslog.conf
#add/modify your own rssylog conf inside
```

# Secure device root account with random 32chars password
If you are sure that your ssh key is working fine, you can add following line in the script INSTALLER.sh. 
It will set root password to 32 random characters, so no random guest in your network will be able to ssh in.




```
web-888:~# mount -o remount,rw /media/mmcblk0p1/; cd /media/mmcblk0p1/

#add this to the end of script INSTALLER.sh. 
vi INSTALLER.sh.
NEWPASS=$(< /dev/urandom tr -dc 'A-Za-z0-9' | head -c32); echo "root:$NEWPASS" | chpasswd;

sync;sync;  mount -o remount,ro /media/mmcblk0p1/
reboot

```
