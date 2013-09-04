CoDBap
======

The ABAP CouchDB API


Nugget: NUGG_CODBAP-xxx.nugg
 
## Required Packages
JSON Document Class (see https://cw.sdn.sap.com/cw/groups/zjson )
 
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
see embedded demo report
 
## Creating the instance
```
codbap = zcl_codbap=>get_api( 'your_couchdb_host_or_ip' ).
```

## Tests
```
*--- Test 1: Get CouchDB Version ------------------------------------*
DATA: version TYPE string.

TRY.
 	version = codbap->get_version( ).

   CATCH zcx_codbap_error INTO codbap_error.
 	error_text = codbap_error->get_text( ).
 	WRITE:/ error_text.
 	RETURN.

ENDTRY.

WRITE:/ version.
 
 
*--- Test 2: Create new Database -----------------------------------*
TRY.
 	codbap->create_db( 'spfli' ).

   CATCH zcx_codbap_error INTO codbap_error.
 	error_text = codbap_error->get_text( ).
 	WRITE:/ error_text.
 	RETURN.

ENDTRY.

WRITE:/ 'Ok'.
 
 
*--- Test 3: copy SAP table entries to CouchDB ----------------------*
DATA: id   	TYPE string
	, rev  	TYPE string
	, spfli_t  TYPE TABLE OF spfli
	, json 	TYPE string
	.
FIELD-SYMBOLS: <spfli> TYPE spfli.
SELECT * FROM spfli
  INTO TABLE spfli_t.
LOOP AT spfli_t
  ASSIGNING <spfli>.
  codbap->json_doc->set_data( <spfli> ).
  json = codbap->json_doc->get_json( ).
  TRY.
  	codbap->put_doc(
     	EXPORTING dbname  = 'spfli'
               	json	= json
     	IMPORTING id_out  = id
               	rev_out = rev
               	).
	CATCH zcx_codbap_error INTO codbap_error.
  	error_text = codbap_error->get_text( ).
  	WRITE:/ error_text.
  	RETURN.
  ENDTRY.
  WRITE:/ 'ID :', id, 'Rev :', rev.
ENDLOOP.
 
 
*--- Test 4: Change Document ----------------------------------------*
DATA: id TYPE string
	, rev TYPE string
	, spfli TYPE spfli
	, json TYPE string
	.
 
SELECT SINGLE * FROM spfli
  INTO spfli
  WHERE carrid = 'UA'
  AND   connid = '3504'.
 
CHECK sy-subrc = 0.
 
spfli-airpto = 'DUS'.
spfli-cityto = 'DUESSELDORF'.
 
codbap->json_doc->set_data( spfli ).
json = codbap->json_doc->get_json( ).
 
TRY.
	codbap->put_doc(
   	EXPORTING dbname  = 'spfli'
             	id_in   = 'A77ED94CC573100CE1000000C0A8263A'
             	rev_in  = '1-9b92806f09117d71b3497aa8a4707d8b'
             	json	= json
   	IMPORTING id_out  = id
             	rev_out = rev
             	).
 
  CATCH zcx_codbap_error INTO codbap_error.
	error_text = codbap_error->get_text( ).
	WRITE:/ error_text.
	RETURN.
 
ENDTRY.
 
WRITE:/ 'ID :', id, 'Rev :', rev.
 
 
*--- Test 5: Create View (map only) ---------------------------------*
DATA: id   	TYPE string
	, rev  	TYPE string
	.
 
TRY.
	codbap->create_design_doc(
   	EXPORTING dbname   	= 'spfli'
             	folder   	= 'test5'
             	viewname 	= 'map_only'
             	map_function = 'function(doc){ emit(doc.airpfrom, doc.cityfrom) }'
   	IMPORTING id_out  = id
             	rev_out = rev
             	).
 
  CATCH zcx_codbap_error INTO codbap_error.
	error_text = codbap_error->get_text( ).
	WRITE:/ error_text.
	RETURN.
 
ENDTRY.
 
WRITE:/ 'ID :', id, 'Rev :', rev.
 
 
*--- Test 6: Display View Data (map only) ----------------------------*
DATA: key TYPE string
	, value TYPE string
	, rows TYPE string
	, row TYPE string
	.
 
TRY.
	codbap->get_view_data(
  	dbname   = 'spfli'
  	folder   = 'test5'
  	viewname = 'map_only'
*  	key = '"FRA"'        	"<- if set, don't use start-/endkey
*  	startkey = '"FRA"'
*  	endkey   = '"SFO"'
  	).
 
  CATCH zcx_codbap_error INTO codbap_error.
	error_text = codbap_error->get_text( ).
	WRITE:/ error_text.
	RETURN.
 
ENDTRY.
 
rows = codbap->json_doc->get_value( 'rows' ).
codbap->json_doc->set_json( rows ).
 
WHILE codbap->json_doc->get_next( ) IS NOT INITIAL.
 
  key = codbap->json_doc->get_value( 'key' ).
  value = codbap->json_doc->get_value( 'value' ).
 
  WRITE:/ key, value.
 
ENDWHILE.
 
 
*--- Test 7: Create View (map and reduce, count entries) ------------*
DATA: id   	TYPE string
	, rev  	TYPE string
	.
 
TRY.
	codbap->create_design_doc(
   	EXPORTING dbname      	= 'spfli'
             	folder      	= 'test7'
             	viewname    	= 'map_reduce'
             	map_function	= 'function(doc){ emit([doc.airpfrom,doc.airpto], 1) }'
             	reduce_function = 'function(keys, values, rereduce){ return sum(values) }'
   	IMPORTING id_out  = id
             	rev_out = rev
             	).
 
  CATCH zcx_codbap_error INTO codbap_error.
	error_text = codbap_error->get_text( ).
	WRITE:/ error_text.
	RETURN.
 
ENDTRY.
 
WRITE:/ 'ID :', id, 'Rev :', rev.
 
 
*--- Test 8: Display View Data (map and reduce) ----------------------*
DATA: key TYPE string
	, value TYPE string
	, rows TYPE string
	, row TYPE string
	.
 
TRY.
	codbap->get_view_data(
  	dbname   = 'spfli'
  	folder   = 'test7'
  	viewname = 'map_reduce'
  	startkey = '["FRA"]'
  	endkey   = '["SFO","SIN"]'
  	group_level = '2'
  	).
 
  CATCH zcx_codbap_error INTO codbap_error.
	error_text = codbap_error->get_text( ).
	WRITE:/ error_text.
	RETURN.
 
ENDTRY.
 
rows = codbap->json_doc->get_value( 'rows' ).
codbap->json_doc->set_json( rows ).
 
WHILE codbap->json_doc->get_next( ) IS NOT INITIAL.
 
  key = codbap->json_doc->get_value( 'key' ).
  value = codbap->json_doc->get_value( 'value' ).
 
  WRITE:/ key, value.
 
ENDWHILE.
``` 
 
This is a beta release, please be careful and don't run this API against an important CouchDB instance!
Feedback welcome...


