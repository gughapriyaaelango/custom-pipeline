Imagining a user-friendly UI for users to build custom ETL pipelines using drag-and-drop method.<br>
In Graph representation, the pipeline can be represented as a graph, with nodes as vertices and connections as edges..<br>
Nodes can be categorized as Source/Target, Transform, Join/Split, Custom and Subgraph. Each node will have unique id and configuration details. The user defines the nodes and connects them to define the workflow. This is the example pipeline assumed for this task:
.<br>Oracle CDC >> Data ingestion, transformation, denormalization >> Delta Lake.<br>.<br>

The pipeline can be represented as a JSON object containing an array of nodes, where each node represents a step in the ETL process..<br>
To compile and execute:.<br>
The JSON representation is interpreted and the necessary Spark code is compiled to execute the ETL pipeline. The function of each node will be mapped from sql to Spark..<br>
Oracle CDC with JDBC connection can be used to ingest the data to staging layer or transform further to delta lake..<br>
Important config details -.<br>
Using source database authentication.<br>
Specifying source and target table, their primary keys and incremental column.<br>
Number of partitions/mappers, estimated record size.<br>
If needed for hive connection, hive authentication to query from presto.<br>
.<br>
Oracle CDC - Change data capture from last succcessful job time.<br>
From Oracle database, whatever insert,update,delete has changed from last successful job time, only the changes will be processed. This incremental processing of data requires us to select a specific column to capture changes like the insert_timestamp. The challenge can be a lag of 1 run when performing incremental joins.(can be more prone when using complex joins, window functions, etc.)Example:
.<br>1st run.<br>
Main. Reference.<br>
Address Zipcode Join - addr_zip.<br>
1 60601 60601 60601 - 60601.<br>
2 60602 No Match 60602 - null.<br>
2nd Run.<br>
Address Zipcode Join - addr_zip.<br>
3 60603 60602 60601 - 60601.<br>
4 60604 60603 60602 - 60602.<br>.<br>

Transformation logic assumed:.<br>
Task 1 - Explode table customer from array datatype to columns.<br>
Task 2 - Left join customer and orders table.<br>
Task 3 - Filter only columns where delete_ind = 0.<br>
Task 4 - Select only distinct records.<br>
Task 5 - Get counts of distinct orders of each customer.<br>
Task 6 - Aggregate by customer id and get max date for latest order.<br>
Task 7 - Window partition by customer id and get max bill sequence id.<br>
Task 8 - When order discount == "BlackFriday", True, False.<br>
Task 9 - Write True as 1 and False as 0.<br>
Task 10 - Take 10% off the order price.<br>
Task 11 - Create UDF to take customer id, order id, partition id and concatenate into UUID using |.<br>
Task 12 - Check if order price is null.<br>
Task 13 - Coalesce the order price.<br>
Task 14 - Create column order count to count number of orders for each unique customer.<br>
Task 15 - Broadcast a small table order category.<br>
Task 16 - Trim the order name column strings.<br>
Task 17 - Select purchase year and check if it is above 2018.<br>
Task 18 - Sort by the customer id.<br>
Task 19 - Rename column seq_id to sequence_id.<br>
Task 20 - Format and denormalise to write data to delta lake.<br>
