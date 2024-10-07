
## Technologies Used

- **Apache Flink**: For stream processing.
- **Apache Kafka**: For messaging between microservices.
- **Avro**: For data serialization.
- **SQL Database**: MongoDB to store invoice data.

## Architecture Overview


**Microservice 1: Invoice Reader**


**Functionality Overview**

-   Monitors a specified directory for new invoice files.
-   Processes files with the naming convention invoice_<date>.csv.
-   Reads and parses the file line by line, serializes the data to Avro format, and publishes it to a Kafka topic.
-   Upon completion, moves the processed file to a designated "processed" directory, renaming it accordingly.
-   Maintains a unique correlation ID for each invoice line, publishing successful acknowledgments to an "audit" topic.
-   Handles exceptions by publishing error messages to a Dead Letter Queue (DLQ) topic.


**Implementation Steps**


**Directory Monitoring**: Use Apache NIO's WatchService to monitor a folder for new CSV files.

**Reading and Processing Files**: Use Apache Flink to stream the CSV data line by line.

**Parsing Logic**: Implement a CSV parser that extracts fields into an Invoice object.

**Avro Serialization**: Serialize the parsed Invoice objects to Avro format using the defined Avro schema.

**Kafka Publishing**: Publish the serialized data to a Kafka topic named "invoice".

**File Management**: After processing, move the file to a "processed" directory and rename it to invoice_<date>_processed.csv.

**Correlation ID Management**: Generate and attach a unique correlation ID to each invoice line. Upon receiving acknowledgment from Kafka, publish the ID and other relevant data to the "audit" topic.

**Error Handling**: If any exceptions occur during processing, publish the error details to the "DLQ" topic.
 

**Microservice 2: Database Writer**

**Functionality Overview**

-   Consumes messages from the "invoice" Kafka topic.
-   Deserializes the incoming data into Invoice.java POJO.
-   Stores the invoice data in a relational database.

**Implementation Steps**

**Kafka Consumer Setup**: Use Apache Kafka’s consumer API to subscribe to the "invoice" topic.

**Data Deserialization**: Deserialize the incoming Avro messages into Invoice objects.

**Database Operations**: Use Spring Data JPA to persist the Invoice data into the database.

**Microservice 3: Audit Service**

**Functionality Overview**

-   Consumes messages from the "audit" Kafka topic.
-   Deserializes audit data into an Audit.java POJO.
-   Updates or inserts the audit data into the audit table in the database.

**Implementation Steps**

**Kafka Consumer Setup**: Use Apache Kafka’s consumer API to subscribe to the "audit" topic.

**Data Deserialization**: Deserialize the incoming messages into Audit objects.

**Database Operations**: Use JDBC or an ORM to update or insert the audit records into the database.


## Sample CSV Invoice File
CSV file -> invoice_07102024.csv

```
invoiceNumber,date,amount,customerName,itemDescription,quantity,totalAmount
INV-001,2024-10-07,1500.75,John Doe,Office Supplies,10,1500.75
INV-002,2024-10-08,320.50,Jane Smith,Printer Ink,5,320.50
INV-003,2024-10-09,2750.00,Acme Corporation,Office Furniture,15,2750.00
```

## Why
**Apache Flink** 

Famous for its powerful stream processing capabilities, enabling real-time data ingestion and analysis of invoice files as they are added. Its ability to handle high-throughput workloads makes it ideal for our use case, where invoice files can range from 200MB to 2TB. 

**True Stream Processing**: Unlike many other frameworks that operate in a batch-oriented manner, Apache Flink is designed for true stream processing. It processes data as it arrives, enabling real-time analytics and low-latency responses, which is crucial for applications requiring immediate insights, such as invoice processing.

**Optimized Execution Engine**: Flink’s execution engine is designed for high throughput, allowing it to handle millions of events per second with low latency. This is particularly beneficial when processing large invoice files or high volumes of transactions, ensuring that the system can scale effectively.

**Distributed Architecture**: Flink's distributed architecture allows it to scale horizontally by adding more nodes to the cluster. This elasticity enables it to handle increasing data loads seamlessly, making it suitable for processing large datasets like invoices ranging from 200MB to 2TB.

**Checkpointing Mechanism**: Flink’s checkpointing mechanism ensures that the system can recover from failures with minimal data loss. This feature is crucial for production systems processing financial data, such as invoices, where accuracy is paramount.

**Exactly-Once Semantics**: Flink provides strong guarantees for processing, including exactly-once semantics, ensuring that each record is processed once and only once, which is critical for maintaining data integrity.

## Why
**Apache Kafka**

Serves as a reliable messaging system, facilitating decoupled communication between microservices, ensuring that invoice data can be asynchronously transmitted from the reader to the writer and audit services without bottlenecks. 

## Why
**Avro**

**Compact Format**: Avro uses a compact binary format for data serialization, which significantly reduces the size of the data being transmitted over the network or stored in a database. This compactness leads to lower I/O operations, which can greatly enhance performance.

**Schema-Based**: Avro requires a schema for data serialization, allowing it to serialize data efficiently by encoding only the differences in schema, rather than the entire dataset. This feature is particularly advantageous when dealing with large datasets, as it minimizes the data overhead.

**Compatibility**: Avro is widely supported across various big data tools and frameworks, such as Apache Spark, Apache Hadoop, and Apache Kafka. This makes it a natural fit for applications that require high-performance data processing in distributed environments.

**Stream Processing**: When used with stream processing frameworks like Apache Flink, Avro's efficiency in serialization/deserialization becomes even more pronounced, allowing for real-time data processing without significant latency

**Speed**: Avro is designed for fast serialization and deserialization. The binary encoding is quicker to process than text-based formats like JSON or XML, which is essential for high-throughput systems.

**Reduced Payload Size**: Since Avro files are smaller compared to their JSON or XML counterparts, they require less bandwidth for transmission. This is particularly beneficial in cloud environments or when communicating between microservices over a network.

## Why
**MongoDB**

**Document-Oriented Storage**: MongoDB is a NoSQL database that stores data in a flexible, JSON-like format (BSON). This flexibility allows for easy modifications to the data structure without requiring complex migrations, which is particularly beneficial for invoices that may have varying fields and structures

**Horizontal Scalability**: MongoDB supports sharding, allowing it to scale horizontally by distributing data across multiple servers. This makes it well-suited for applications with large datasets, such as invoices ranging from 200MB to 2TB, as it can handle growing volumes of data seamlessly.

**High Throughput and Low Latency**: MongoDB is designed for high performance with the ability to handle large volumes of read and write operations efficiently. Its in-memory processing capabilities contribute to low latency, which is crucial for applications requiring real-time data access and processing.

**Flexible Query Language**: MongoDB offers a powerful and expressive query language that supports ad-hoc queries, indexing, and aggregation. This flexibility allows for complex data retrieval operations that can be performed efficiently on invoice data.

**Natural Fit for JSON Data**: Since invoice data can often be represented in JSON format, MongoDB’s native handling of JSON-like documents makes it a natural choice for storing and querying such data efficiently.

## Performance Enhancements

**Auto-Scaling**: Implement auto-scaling for your microservices using Kubernetes to dynamically adjust resources based on the workload. This ensures optimal performance during peak times without over-provisioning during low demand.
Flink Task Slots Optimization: Fine-tune the task slot allocation in Flink to maximize resource utilization based on the workload characteristics.
Batch Processing for Ingestion:

**Buffered Writes**: For MongoDB, consider batching writes to reduce the overhead of individual document insertions, especially during high-throughput scenarios. Use bulk operations to improve performance when writing large numbers of invoices.
Data Caching:

**In-Memory Caching**: Implement an in-memory caching layer (like Redis or Memcached) to cache frequently accessed invoice data. This reduces the load on MongoDB and improves response times for read operations.
Aggregation Pipelines in MongoDB:

**Pre-Computed Aggregations**: Use MongoDB’s aggregation framework to pre-compute and store aggregated data for reporting, which can reduce query time for analytics.
Asynchronous Processing:

**Asynchronous Database Writes**: Use asynchronous database writes in your database writer microservice to avoid blocking operations and improve throughput.

## Security Enhancements

**Data Encryption**:At-Rest and In-Transit Encryption: Ensure that sensitive data, such as invoice details, is encrypted both at rest (in MongoDB) and in transit (using TLS for Kafka and Flink communications). This protects data from unauthorized access.
Role-Based Access Control (RBAC):

**Fine-Grained Access Control**: Implement RBAC in MongoDB to control access to data at a granular level. Define roles and permissions based on the principle of least privilege, ensuring that only authorized users and services can access sensitive data.

**Secure Kafka Configuration**:

**Authentication and Authorization**: Enable authentication and authorization for Kafka using SASL and ACLs to control access to topics, ensuring that only trusted applications can publish or consume messages.
Regular Security Audits:

**Vulnerability Scanning**: Conduct regular security audits and vulnerability scans on your microservices and underlying infrastructure to identify and address potential security gaps.

**API Security**:Implement OAuth 2.0: If your services expose APIs, consider implementing OAuth 2.0 for secure token-based authentication and authorization, providing a secure way for services to communicate with each other.
Logging and Monitoring:

**Centralized Logging**: Implement centralized logging for all microservices using tools like ELK Stack (Elasticsearch, Logstash, Kibana) or Grafana Loki. This helps in monitoring activities and identifying potential security breaches.
Intrusion Detection: Consider using an intrusion detection system (IDS) to monitor network traffic and detect suspicious activities.
