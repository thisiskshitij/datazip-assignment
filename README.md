
 PostgreSQL to Iceberg Data Pipeline via OLake & Spark

 Assignment Overview

This assignment demonstrates setting up a data pipeline using PostgreSQL, OLake's Hive integration, Apache Iceberg, and Apache Spark. It includes:

 Extracting data from a PostgreSQL database
 Syncing it into Iceberg format using OLake and Hive Metastore
 Querying the Iceberg tables using Apache Spark SQL

All components are containerized using Docker.

Tools Used

| Tool             | Purpose                               |
|------------------|----------------------------------------|
| Docker Compose   | Container orchestration                |
| PostgreSQL       | Source database                        |
| OLake CLI        | Schema discovery and data sync         |
| Hive Metastore   | Required for Iceberg table catalog     |
| Apache Iceberg   | Data lake table format                 |
| Apache Spark     | Querying the data via Spark SQL        |

 Step-by-Step Setup

 Clone OLake Repository

```bash
git clone https://github.com/datazip-inc/olake
cd olake
```

 Docker Compose Setup

Create a file named `docker-compose.yml` in the root directory:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14
    container_name: olake_postgres
    environment:
      POSTGRES_USER: olake
      POSTGRES_PASSWORD: olake123
      POSTGRES_DB: orders_db
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  hive-metastore:
    image: datazip/hive-metastore:latest
    container_name: olake_hive
    environment:
      METASTORE_DB_HOST: postgres
    ports:
      - "9083:9083"
    depends_on:
      - postgres

  olake:
    image: datazip/olake-cli:latest
    container_name: olake_cli
    volumes:
      - ./data:/data
    depends_on:
      - postgres
      - hive-metastore

  spark:
    image: bitnami/spark:latest
    container_name: olake_spark
    ports:
      - "4040:4040"
      - "8080:8080"
    depends_on:
      - hive-metastore

volumes:
  pgdata:
```

Start the containers:

```bash
docker-compose up -d
```

3. Create Sample Table in PostgreSQL

Connect to Postgres:

```bash
docker exec -it olake_postgres psql -U olake -d orders_db
```

Run SQL commands:

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_name VARCHAR(50),
    product VARCHAR(50),
    quantity INT,
    price DECIMAL(10,2),
    order_date DATE
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    electronics_item VARCHAR(100),
    amount DECIMAL
);

INSERT INTO orders (customer_name, electronics_item, amount) VALUES
('Aarav', 'Smartphone', 250.00),
('Vivaan', 'Laptop', 120.50),
('Aditya', 'Headphones', 350.75),
('Ishaan', 'Smartwatch', 150.00),
('Reyansh', 'Tablet', 200.25),
('Arjun', 'Bluetooth Speaker', 180.00),
('Sai', 'Laptop', 330.50),
('Rohan', 'Smartphone', 120.00),
('Mohit', 'Washing Machine', 450.99),
('Aakash', 'Refrigerator', 550.75),
('Saanvi', 'Microwave', 130.50),
('Priya', 'Smart TV', 700.00),
('Neha', 'Air Conditioner', 400.00),
('Ananya', 'Food Processor', 200.99),
('Shruti', 'Electric Kettle', 85.25),
('Krishna', 'Dishwasher', 300.50),
('Ravi', 'Vacuum Cleaner', 175.00),
('Pooja', 'Water Heater', 220.75),
('Kavya', 'Refrigerator', 480.50),
('Deepika', 'Printer', 145.00);


4. OLake Schema Discovery

Open a shell inside the OLake container:

```bash
docker exec -it olake_cli sh
```

Run the schema discovery command:

```bash
olake discover postgres   --host=olake_postgres   --port=5432   --user=olake   --password=olake123   --database=orders_db   --schema=public   --table=orders   --output=/data/orders_schema.yaml
```

 5. Sync Data to Iceberg via Hive

Run the sync command:

```bash
olake sync postgres   --host=olake_postgres   --port=5432   --user=olake   --password=olake123   --database=orders_db   --schema=public   --table=orders   --warehouse=hdfs:///user/hive/warehouse   --catalog=hive   --metastore-uri=thrift://olake_hive:9083
```

 6. Query Iceberg Table using Spark

Start a Spark SQL shell:

```bash
docker exec -it olake_spark spark-sql   --conf spark.sql.catalog.hive=org.apache.iceberg.spark.SparkCatalog   --conf spark.sql.catalog.hive.type=hive   --conf spark.sql.catalog.hive.uri=thrift://olake_hive:9083
```

Run a query:

```sql
SELECT * FROM hive.orders;
```

Screenshots to Include

- `docker ps` showing all containers running.
- Output of `SELECT * FROM hive.orders` in Spark SQL.
- Spark UI (http://localhost:4040 or :8080) showing query DAG.

 Challenges Faced

-Hive Metastore not reachable by OLake
-Port conflicts on 5432 or 9083
-Incorrect YAML format during schema discovery
-Network configuration between Docker containers
-OLake Docker container was not starting.
-Confusion over using Windows paths with Docker volume mounts.
-Uncertainty about whether --writer-path was required.
-Missing required files: --catalog, --destination, and --state.
-Incorrect Docker command syntax for Windows CMD.
-OLake Docker container was not connecting to the PostgreSQL database.

Suggested Improvements

- Automate discovery + sync in one script
- Use volume mounts to persist warehouse directory
- Add Airflow or Prefect for orchestration (bonus)

 Resources Used

- [OLake GitHub](https://github.com/datazip-inc/olake)
- [OLake Documentation](https://docs.olake.io/)
- [Apache Iceberg Docs](https://iceberg.apache.org/)
- [Apache Spark SQL Docs](https://spark.apache.org/docs/latest/sql-getting-started.html)
