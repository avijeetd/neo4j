# neo4j

Created the Nodes and relationships in Aura DB as below


![Capture2](https://github.com/avijeetd/neo4j/assets/3703774/136221f9-17c7-4430-92c8-1a78520e0a85)

For the relationships created the CSV Files using SQL Workbench by creating Views based on foreign Keys in Customer, Purchases and Transfer Tables.

## View creation SQLs

Create view customer_purchases as 
select a.cif, a.cardnumber, b.transactionid from ad_customers a, ad_purchases b
where a.cardnumber = b.cardnumber;

create view customer_transfers_sender as 
select a.cif, a.accountnumber, c.transactionid from ad_customers a, ad_transfers c
where a.accountnumber = c.SENDERACCOUNTNUMBER;

create view customer_transfers_receiver as 
select a.cif, a.accountnumber, c.transactionid from ad_customers a, ad_transfers c
where a.accountnumber = c.RECEIVERACCOUNTNUMBER;

The CSV relationship files are in the folder.

## CYPHER Code generated by Import Utility of Aura DB - ID field selected for each table as CIF, TransactionID, 

Key statement
CREATE CONSTRAINT `imp_uniq_Customer_ID` IF NOT EXISTS
FOR (n: `Customer`)
REQUIRE (n.`ID`) IS UNIQUE;

UNWIND $nodeRecords AS nodeRecord
WITH *
WHERE NOT nodeRecord.`CIF` IN $idsToSkip AND NOT toInteger(trim(nodeRecord.`CIF`)) IS NULL
MERGE (n: `Customer` { `ID`: toInteger(trim(nodeRecord.`CIF`)) })
SET n.`Age` = toInteger(trim(nodeRecord.`Age`))
SET n.`EmailAddress` = nodeRecord.`EmailAddress`
SET n.`FirstName` = nodeRecord.`FirstName`
SET n.`LastName` = nodeRecord.`LastName`
SET n.`PhoneNumber` = nodeRecord.`PhoneNumber`
SET n.`Gender` = nodeRecord.`Gender`
SET n.`Address` = nodeRecord.`Address`
SET n.`Country` = nodeRecord.`Country`
SET n.`JobTitle` = nodeRecord.`JobTitle`
SET n.`CardNumber` = nodeRecord.`CardNumber`
SET n.`AccountNumber` = nodeRecord.`AccountNumber`;

CREATE CONSTRAINT `imp_uniq_Purchases_ID` IF NOT EXISTS
FOR (n: `Purchases`)
REQUIRE (n.`ID`) IS UNIQUE;

UNWIND $nodeRecords AS nodeRecord
WITH *
WHERE NOT nodeRecord.`TransactionID` IN $idsToSkip AND NOT toInteger(trim(nodeRecord.`TransactionID`)) IS NULL
MERGE (n: `Purchases` { `ID`: toInteger(trim(nodeRecord.`TransactionID`)) })
SET n.`CardNumber` = nodeRecord.`CardNumber`
SET n.`Merchant` = nodeRecord.`Merchant`
SET n.`Amount` = toFloat(trim(nodeRecord.`Amount`))
SET n.`PurchaseDatetime` = nodeRecord.`PurchaseDatetime`
SET n.`CardIssuer` = nodeRecord.`CardIssuer`;

CREATE CONSTRAINT `imp_uniq_Transfers_ID` IF NOT EXISTS
FOR (n: `Transfers`)
REQUIRE (n.`ID`) IS UNIQUE;

UNWIND $nodeRecords AS nodeRecord
WITH *
WHERE NOT nodeRecord.`TransactionID` IN $idsToSkip AND NOT toInteger(trim(nodeRecord.`TransactionID`)) IS NULL
MERGE (n: `Transfers` { `ID`: toInteger(trim(nodeRecord.`TransactionID`)) })
SET n.`SenderAccountNumber` = nodeRecord.`SenderAccountNumber`
SET n.`ReceiverAccountNumber` = nodeRecord.`ReceiverAccountNumber`
SET n.`Amount` = toFloat(trim(nodeRecord.`Amount`))
SET n.`TransferDatetime` = nodeRecord.`TransferDatetime`;

UNWIND $relRecords AS relRecord
MATCH (source: `Customer` { `ID`: toInteger(trim(relRecord.`CIF`)) })
MATCH (target: `Transfers` { `ID`: toInteger(trim(relRecord.`TRANSACTIONID`)) })
MERGE (source)-[r: `Transfer_out`]->(target);

UNWIND $relRecords AS relRecord
MATCH (source: `Customer` { `ID`: toInteger(trim(relRecord.`CIF`)) })
MATCH (target: `Purchases` { `ID`: toInteger(trim(relRecord.`TRANSACTIONID`)) })
MERGE (source)-[r: `Card Transactions`]->(target);

UNWIND $relRecords AS relRecord
MATCH (source: `Transfers` { `ID`: toInteger(trim(relRecord.`TRANSACTIONID`)) })
MATCH (target: `Customer` { `ID`: toInteger(trim(relRecord.`CIF`)) })
MERGE (source)-[r: `Tranfser_Ins`]->(target);

The Nodes (customer- 100, Purchases - 10000, Transfers - 1000) and Relationships ( Card Transactios - 10000, Transfers - 1000s) are available for Explore and Query

Beautiful Explore options
![Capture](https://github.com/avijeetd/neo4j/assets/3703774/3c188a08-2ef1-4192-9631-d70869b42c7e)

![Capture3](https://github.com/avijeetd/neo4j/assets/3703774/a30f77b5-47e0-49c4-b72c-ecdf8faeb10d)

![Capture1](https://github.com/avijeetd/neo4j/assets/3703774/1b12e289-4888-40a5-aec8-8112b29d4779)

Queries - I ran a few Queries below

## Max Merchant used
MATCH (p:Purchases)
RETURN count(p.Merchant), p.Merchant

367
"It Smart Group"
352
"Demaco"

## Customers who did Max purchases with It Smart Group

match (c:Customer)-[:`Card Transactions`]->(p:Purchases{Merchant:'It Smart Group'}) return count(c), c.FirstName,c.LastName

9
"Matt"
"Carter"
8
"Morgan"
"James"
8
"Candace"
"Shea"

## Transfers withing customers
match (c:Customer)-[:Transfer_out]->(t:Transfers)<-[:Transfer_out]-(c1:Customer)
return c.FirstName,max(t.Amount),c1.FirstName

"Sofie"
21815.7999
"Nathan"
"Nathan"
21815.7999
"Sofie"
