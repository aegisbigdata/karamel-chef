# karamel-chef
This chef cookbook installs Karamel. Used by Vagrant to provision multi-node clusters.

# Baremetal installation with karamel-ui (version 0.6)
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
Change the `baremetal/username` user to your user (osboxes) in the cluster definition: `karamel-chef/cluster-defns/1.aegis.yml`

https://github.com/aegisbigdata/karamel-chef/blob/aegis/cluster-defns/1.aegis.yml#L3

7. Get karamel (http://karamel.io) version 0.5:

http://www.karamel.io/sites/default/files/downloads/karamel-0.5.tgz

8. Run karamel:
```
cd karamel-0.5
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
# Custom installation - mainly for testing cluster
## Dela instructions
Follow the dela/README.md instructions

## Visualization libraries
1. Hide Code Extension
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

Folium might fail to display the map in the notebook due to CORS restrictions with iframes. To address this issue, edit the file `jupyter_notebook_config_template.py` to contain the following:

```
c.NotebookApp.tornado_settings = {
    'headers': {
        'Content-Security-Policy': "frame-ancestors 'self' "
    }
}
```
Fixed in: https://github.com/aegisbigdata/hopsworks/commit/9ffd3098498fec5b8a89d7428467cfa43ef3522a


2. QGrid (https://github.com/quantopian/qgrid)
```
sudo pip install qgrid
sudo jupyter nbextension install --py qgrid --sys-prefix
sudo jupyter nbextension enable --py qgrid --sys-prefix
```
3. 
Install these libraries in both python 2.7 and 3.6 on all machines and add them to the variables table
```
python-highcharts
folium
termcolor
requests
```

## Spark common lang lib fix:
Replace the hops-util lib within hdfs, with the snurran versison 0.4.1(as version 0.4.0)
```
/user/spark/hops-util-0.4.0.jar
```
    
## Out of extent error fix (Fixed in 2.8.2.4)

Increase the preallocated disk space in the script /srv/hops/mysql-cluster/ndb/scripts/create-disk-tables.sh
by changing the dc parameter at the end of the command, the number is in GB, and then run the script.

## NDB Out of memory

Increase the DataMemory to 20GB, the IndexMemory to 10GB, and the MaxNoOfExecutionThreads to 4 in the config.ini file inside the mysql-cluster installation (/srv/hops/mysql-cluster/).

## Temporary fixes:
### python-highcharts
Error:
```
highcharts.src.js:28370 Uncaught TypeError: (e.series || []).forEach is not a function
    at a.Chart.firstRender (VM1982 highcharts.js:283)
    at a.Chart.<anonymous> (VM1982 highcharts.js:257)
    at a.fireEvent (VM1982 highcharts.js:31)
    at a.Chart.init (VM1982 highcharts.js:256)
    at a.Chart.getArgs (VM1982 highcharts.js:256)
    at new a.Chart (VM1982 highcharts.js:255)
    at HTMLDocument.<anonymous> (VM1986 about:srcdoc:1)
    at c (VM1981 jquery.min.js:3)
    at Object.fireWith [as resolveWith] (VM1981 jquery.min.js:3)
    at Function.ready (VM1981 jquery.min.js:3)
```
Problem: Version mismatch

Temporary Fix:

file /srv/hops/jupyter/.local/lib/python2.7/site-packages/highcharts/highcharts/highcharts.py
lines 71-74:
```
'https://code.highcharts.com/6.2.0/highcharts.js',
'https://code.highcharts.com/6.2.0/highcharts-more.js',
'https://code.highcharts.com/6.2.0/modules/heatmap.js',
'https://code.highcharts.com/6.2.0/modules/exporting.js'
```
line 210:
```
self.add_JSsource('http://code.highcharts.com/6.2.0/modules/treemap.js')
```
line 295:
```
self.add_JSsource("http://code.highcharts.com/6.2.0/highcharts-3d.js")
```
line 343:
```
self.add_JSsource('http://code.highcharts.com/6.2.0/modules/drilldown.js')
```
file /srv/hops/jupyter/.local/lib/python2.7/site-packages/highcharts/highstock/highstock.py
lines 65-67:
```
'https://code.highcharts.com/stock/6.2.0/highstock.js',
'https://code.highcharts.com/stock/6.2.0/modules/exporting.js',
'https://code.highcharts.com/6.2.0/highcharts-more.js',
```
