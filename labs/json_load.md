
# Oracle JSON 



## Steps:#


 **Create and Load JSON data from External Table**

1. Login to PDB: Below are the details-
   
    ````
    <copy>
    Username: appjson
    Password: Oracle_4U
    PDB name: pdbjxl
    PORT: 1530
    </copy>
    ````
    
2. Create a directory
    
    We will create a directory which will point to the location where json dump file is stored-
     
    ````
    <copy>
    create or replace directory ORDER_ENTRY as '/home/oracle/Spatial/script/db-sample-schemas-19.2/order_entry/';
    
             </copy>
    ````
   
3. Create a simple table to store JSON documents
   
   In Oracle there is no dedicated JSON data type. JSON documents are stored in the database using standard Oracle data types such as VARCHAR2, CLOB and BLOB. 

   In order to ensure that the content of the column is valid JSON data, a new constraint **IS JSON**, is provided that can be applied to a column. This constraint returns TRUE if the content of the column is well-formed, valid JSON and FALSE otherwise.

   This first statement in this module creates a table which will be used to contain JSON documents.



    ````
    <copy>
    create table PURCHASE_ORDER (
    ID            RAW(16) NOT NULL,
    DATE_LOADED   TIMESTAMP(6) WITH TIME ZONE,
    PO_DOCUMENT CLOB CHECK (PO_DOCUMENT IS JSON)
    )
    /
    </copy>
    ````
    This statement creates a very simple table, PURCHASE-ORDER. The table has a column PO-DOCUMENT of type CLOB. The IS JSON constraint is applied to the column PO-DOCUMENT, ensuring that the column can store only well formed JSON documents.

4. Loading JSON Documents into the database  

    This statement creates a simple external table that can read JSON documents from a dump file generated by a typical No-SQL style database. In this case, the documents are contained in the file PurchaseOrders.dmp. The SQL directory object ORDER_ENTRY points to the folder containing the dump file, and also points to the database’s trace folder which will contain any ‘log’ or ‘bad’ files generated when the table is processed.

    ````
    <copy>
    CREATE TABLE PURCHASE_EXT(
    JSON_DOCUMENT CLOB
    )
    ORGANIZATION EXTERNAL(
    TYPE ORACLE_LOADER
    DEFAULT DIRECTORY ORDER_ENTRY
    ACCESS PARAMETERS (
    RECORDS DELIMITED BY 0x'0A'
    DISABLE_DIRECTORY_LINK_CHECK  
    BADFILE ORDER_ENTRY: 'PURCHASE_EXT.bad'
    LOGFILE ORDER_ENTRY: 'PURCHASE_EXT.log'
    FIELDS(
    JSON_DOCUMENT CHAR(5000)
    ) 
    )
    LOCATION (
     ORDER_ENTRY:'PurchaseOrders.dmp'
    )
    )
    PARALLEL
    REJECT LIMIT UNLIMITED
    /
 
       </copy>
    ````

5. Loading  data from the external table into JSON table
 
    The following statement copies the JSON documents from the dump file into the PURCHASE-ORDER table. 
    ````
    <copy>
    insert into PURCHASE_ORDER
    select SYS_GUID(), SYSTIMESTAMP, JSON_DOCUMENT 
    from PURCHASE_EXT
    where JSON_DOCUMENT IS JSON
    /
    commit
    /
    
    </copy>
    ````

  ![](./images/purchase_order_count.PNG " ")
  ![](./images/insert_ot.PNG " ")


See an issue?  Please open up a request [here](https://github.com/oracle/learning-library/issues).