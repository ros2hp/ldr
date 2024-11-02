## What is Ldr?

Loader (Ldr) is the first in the sequence of five programs that constitute the GoGraph RDF load process.  GoGraph is a rudimentary graph database, developed originally in Go principally as a way to learn the language and now refactored in Rust for a similar reason. GoGraph employees the Tokio asynchronous runtime to implement a highly concurrent and asynchronous design. The database design enables scaling to internet size data volumes. GoGraph currently supports AWS's Dynamodb, although other hyper scalable databases, such as Google's Spanner, is also in development. 



## GoGraph RDF Load Programs

The table below lists the sequence of programs that load a RDF file into the target database, Dynamodb. There is no limit to the size of the RDF file. 

MySQL is used as an intermediary storage facility providing both query and sort capabilties to each of the load programs. Unlike the Go implementation, the Rust version does not support restartability for any of the load programs. This is left as a future enhancement. It is therefore recommended to take a backup of the database after running each of the load programs. 

| Load Program           |  Repo       |  Task                                                   |  Data Source           | Target Database |
|-----------------------:|-------------|---------------------------------------------------------|------------------------|-----------------|
| _RDF-Loader_           |   _ldr_     | _Load a RDF file into Dynamodb and MySQL_               |  _RDF file_            | _Dynamodb, MySQL_ |
|  Attach                | rust-attach | Link child nodes to parent nodes                        |  MySQL           | Dynamodb        |
|  Scalar Propagation    |    sp       | Propagate child scalar data into parent node and  generate reverse edge data     |  MySQL        | Dynamodb     |
|  Double Propagation    |   dp        | Propagate grandchild scalar data into grandparent node* |  MySQL           | Dynamodb        |
|  ElasticSearch         |   es        | Load data into ElasticSearch                            |  MySQL          | Dynamodb        |


* for 1:1 relationships between grandparent and grandchild nodes

## GoGraph Design Guide ##

[A detailed description of GoGraph's database design, type system and data model are described in this document](docs/GoGraph-Design-Guide.pdf)

## Ldr Highlights ##

* Loads a RDF file of unlimited size into Dynamodb.
* Configured number of parallel Tokio tasks or OS threads (depending on version) to load into Dynamodb
* Converts all identifiers (Subject & Object) to UUIDs on the fly.  
* Fully implements Tokio Asynchronous runtime across all operations.

## Ldr Schematic ##

A simplified view of SP is presented in the two schematics below. The first schematic describes the generation of reverse edge data  (child to parent as opposed to he more usual parent-child) which uses a dedicated cache to aggregate the reverse edges and a LRU algorithm to manage the persistance of data to Dynamodb.  The second schematic shows the simpler scalar propagation load.  Parent-child edges are held in MySQL while the scalar data is queried from Dynamodb for each child node and then saved back into Dynamodb where it is associated with the parent node. No cache is required to propagate the child data.

              -----------
             | RDF file  |
              -----------      
                   |
                   V

                  Main                                    ( Main reads batches of RDF records )
                                                          ( and allocates to Loaders          )
                   |
            ----------- . . ---
           |       |           |
           V       V           V
                                                 ---------
         Loader  Loader . .  Loader  --- < > ---|  MySQL  |   
                                                 ---------
           |       |           |
           V       V           V
      ==============================
     |          Dynamodb            |     
      ==============================
       
        Fig 1.  Ldr Schematic  




