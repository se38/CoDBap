CoDBap
======

The ABAP CouchDB API

* Nugget: NUGG_CODBAP-xxx.nugg
 
## Required Packages
JSON Document Class (see https://cw.sdn.sap.com/cw/groups/zjson )

## Installation CoDBap
import Nugget with SAPlink (see https://www.assembla.com/spaces/saplink/wiki)
 
## Installation of CouchDB
Download and install (works fine on virtual servers)
Windows http://www.couchone.com/get#win
Ubunto http://www.couchone.com/get#ubuntu
 
Other Linux (e.g. RHEL/CentOS) (see http://wiki.apache.org/couchdb/Installing_on_RHEL5)
```
rpm -Uvh http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-4.noarch.rpm
yum install couchdb
```
 
Change the IP address in local.ini to the real IP (not 127.0.0.1)
Check your installation with http://your_ip_or_host::5984/_utils
 
## Usage of CoDBap
see Wiki ( https://github.com/se38/CoDBap/wiki )