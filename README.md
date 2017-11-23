# karamel-chef
This chef cookbook installs Karamel. Used by Vagrant to provision multi-node clusters.



1. Run the init script to download chef-dk and karamel to the downloads directory.

1a. Create your own cluster by copying an existing Karamel cluster definition. If your name is John, call it 'hopsworks.1.john'. Then customize it. 

2. To start a Hopsworks VM, use the run.sh script. The parameters are: <operating sys>(ubuntu or centos), number of VMs in the vagrant configuration (1 or 3),  <cluster-postfix-name> (john, hopsworks, jim, virtualbox, etc), [no-random-ports]  - this will forward the ports in the Vagrantfile.

For example,
./run.sh ubuntu 1 jim no-random-ports

3. To shutdown your cluster, run the kill.sh script:

./kill.sh 

# Hopsworsk with Hopssite/Dela installation details
# Installing the vm with hopssite:
Set the installation parameters in
```
hs_env.sh
```
Run (no parameters requires)
```
run_hopssite.sh
```
When deployment is finished, ssh in the vm and run:
```
(vagrant ssh)
/srv/hops/hopssite/hs_install.sh
```
Note: if you installed hopsworks with multi-user support change the user params in:
```
/srv/hops/hopssite/hs_env.sh
(MYSQL_USER)
(GLASSFISH_USER)
```
If you want to use the hopsworks/dela on this machine, reload hopsworks-ear from glassfish admin console
# Installing a vm with dela enabled(slave):
This vm is supposed to connect to a machine installed as per previous step(hopssite)

Set the installation parameters in
```
dela_env.sh
```
Run (no parameters required)
```
run_dela.sh
```

# Simple demo dela install
1. Register your demo installation with hopssite mirror: (http://bbc5.sics.se:8080/hopsworks-cluster/)
2. Change the cluster-defns/1.demodela.yml
  * hopsworks/email and hopssite/password have to match with the registered ones
  * hopsworks/cert/o has to be unique (currently just choose one specific to you). Next fix will include the organization name in the registration and will check there for uniqueness
3. Run de demodela recipies
```
./run.sh demodela 1 demodela
```
Note: Dela requires certain ports to be fixed, which means you can only run 1 instance of demodela on a machine. The default ports used by dela are 42011, 42012, 42013
