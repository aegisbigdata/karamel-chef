# karamel-chef
This chef cookbook installs Karamel. Used by Vagrant to provision multi-node clusters.

# Baremetal installation with karamel-ui
Used a ubuntu 16.04 64bit from osboxes(http://www.osboxes.org/):
https://drive.google.com/file/d/0B_HAFnYs6Ur-ZGdZYTNjMkxvM28/view?usp=sharing
password: osboxes.org
Note: The VM requires a bit of time to load everything - otherwise the apt-get install complains about locks being tacken when you try to install openssh, java or git.

1. The installing user must not have a sudo password. If your user has a password, do:
```
sudo visudo
```
and change the sudo line from:
```
%sudo   ALL=(ALL) ALL
```
to 
```
%sudo   ALL=(ALL) NOPASSWD:ALL
```
2. The installation script has to be able to ssh back into the installed vm - since this is a baremetal installation, we require the machine to be able to ssh into itself
2.1. generate ssh keys
```
cd
mkdir .ssh
chmod 700 .ssh
ssh-keygen -t rsa
```
2.2. authorise your key to ssh into this machine
```
cd .ssh
touch authorized_keys
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
```
2.3. install openssh-server
```
sudo apt-get install openssh-server
```
Note: Make sure you can ssh into the machine with the install user (osboxes) into the 10.0.2.15.

3. Modify in the /etc/hosts file this line:
```
127.0.1.1   osboxes
```
to:
```
10.0.2.15 osboxes
```
4. Install java
```
sudo apt-get install jre-default
```
5. Install git
```
sudo apt-get install git
```
6. Get karamel-chef and change the install user in the cluster definition
```
git clone https://github.com/aegisbigdata/karamel-chef 
```
Change the install/baremetal users to your user (osboxes) in the cluster definition: `karamel-chef/cluster-defns/1.hopsworks.yml`

https://github.com/aegisbigdata/karamel-chef/blame/master/cluster-defns/1.hopsworks.yml#L3

https://github.com/aegisbigdata/karamel-chef/blame/master/cluster-defns/1.hopsworks.yml#L13

7. Get karamel (http://karamel.io) version 0.4:

http://www.karamel.io/sites/default/files/downloads/karamel-0.4.tgz

8. Run karamel:
```
cd karamel-0.4
./bin/karamel
```
9. After running karamel it will automatically load in your browser under `localhost:9090/index.html`

10. Within karamel go to `Menu` - `Load Cluster Defn` and load the `1.hopsworks.yml` cluster definition, after which `Launch` the cluster.

Note: remember to change the install/baremetal user from vagrant to osboxes in this case.

11. Launch and check the status until all recipies are installed (status - done).

12. Access hops at the `localhost:8080/hopsworks`

FAQ
1. Recipe hopsworks::default fails and the logs point to some python permission issues. 
Fix: Change the owner of `~/.local/bin` and `~/.local/lib` (recursively) to the install user (osboxes in this case)
```
sudo chown -R osboxes:osboxes ~/.local/bin
sudo chown -R osboxes:osboxes ~/.local/lib
```

# Dela instructions
Follow the dela/README.md instructions

# Visualization libraries
1. Highcharts
```
pip install --user python-highcharts
```

2. Hide Code Extension
```
pip install --user hide_code
```
```
sudo jupyter nbextension install --py hide_code --sys-prefix
```
```
sudo jupyter nbextension enable --py hide_code --sys-prefix
```
```
sudo jupyter serverextension enable --py hide_code --sys-prefix
```
3. Folium
```
pip install --user folium
```

Folium might fail to display the map in the notebook due to CORS restrictions with iframes. To address this issue, edit the file `jupyter_notebook_config_template.py` to contain the following:

```
c.NotebookApp.tornado_settings = {
    'headers': {
        'Content-Security-Policy': "frame-ancestors 'self' "
    }
}
```
Fixed in: https://github.com/aegisbigdata/hopsworks/commit/9ffd3098498fec5b8a89d7428467cfa43ef3522a

4.
```
pip install --user pyarrow
pip install --user fastparquet
```
# Spark common lang lib fix:
Replace the hops-util lib within hdfs, with the snurran versison 0.4.1(as version 0.4.0)
```
/user/spark/hops-util-0.4.0.jar
```
    
