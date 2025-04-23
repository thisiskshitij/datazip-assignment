
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

- Hive Metastore not reachable by OLake
- Port conflicts on 5432 or 9083
- Incorrect YAML format during schema discovery
- Network configuration between Docker containers
- OLake Docker container was not starting.
- Confusion over using Windows paths with Docker volume mounts.
- Uncertainty about whether --writer-path was required.
- Missing required files: --catalog, --destination, and --state.
- Incorrect Docker command syntax for Windows CMD.
- OLake Docker container was not connecting to the PostgreSQL database.

Suggested Improvements

- Automate discovery + sync in one script
- Use volume mounts to persist warehouse directory
- Add Airflow or Prefect for orchestration (bonus)





 Resources Used

- [OLake GitHub](https://github.com/datazip-inc/olake)
- [OLake Documentation](https://docs.olake.io/)
- [Apache Iceberg Docs](https://iceberg.apache.org/)
- [Apache Spark SQL Docs](https://spark.apache.org/docs/latest/sql-getting-started.html)

![Screenshot (126)](https://github.com/user-attachments/assets/1fd1962c-64f8-4f08-99c2-af49414a0c16)
![Screenshot (138)](https://github.com/user-attachments/assets/46c3b344-6cc9-45da-9461-7faed1047986)
![Screenshot (137)](https://github.com/user-attachments/assets/9db3f486-00a8-489e-b3cd-ae9e436fdb5a)
![Screenshot (136)](https://github.com/user-attachments/assets/98254ff5-10db-42e2-ac58-376ca9149a86)
![Screenshot (135)](https://github.com/user-attachments/assets/84a01af0-eee1-4d16-99c0-2a34fa9b36fa)
![Screenshot (134)](https://github.com/user-attachments/assets/77263986-7841-4c56-9e99-fe81dc2ec372)
![Screenshot (133)](https://github.com/user-attachments/assets/a9902695-0b7d-4e22-b7a8-d358beae60db)
![Screenshot (132)](https://github.com/user-attachments/assets/1ed0455d-1f35-4625-bde5-e701bfaa2b5e)
![Screenshot (131)](https://github.com/user-attachments/assets/49891180-6666-49f1-9be2-e9672100b759)
![Screenshot (130)](https://github.com/user-attachments/assets/e538aeea-4d68-413c-bfda-9caf5f963c1d)
![Screenshot (129)](https://github.com/user-attachments/assets/602b1525-817e-4947-9280-4dadb21070b5)
![Screenshot (128)](https://github.com/user-attachments/assets/c9fe8a45-608d-4f31-b5c6-6e71711e0458)
![Screenshot (127)](https://github.com/user-attachments/assets/947934d4-3ed2-4c63-b1c7-44be630371a5)
![Screenshot (149)](https://github.com/user-attachments/assets/7791743a-da6f-4909-ac50-81b81ddc3baf)
![Screenshot (148)](https://github.com/user-attachments/assets/0f4b9b0c-eab4-49b9-92b4-4347d431f9da)
![Screenshot (147)](https://github.com/user-attachments/assets/d2bb6460-3079-4c2b-b4b0-66c6ceafe450)
![Screenshot (146)](https://github.com/user-attachments/assets/a19d3797-182a-443d-94c8-9162d67a710a)
![Screenshot (145)](https://github.com/user-attachments/assets/fd604cfb-2f9e-4cdb-87f7-03a020d860e8)
![Screenshot (144)](https://github.com/user-attachments/assets/b3fcf75b-f008-423d-a541-985f7c5c8577)
![Screenshot (143)](https://github.com/user-attachments/assets/c42e4e8d-1e8e-44df-8498-eb2434434d2d)
![Screenshot (139)](https://github.com/user-attachments/assets/2eaa0073-7d16-4380-95bd-ebd5cce4246d)
![Screenshot (140)](https://github.com/user-attachments/assets/6239aaa2-21d3-4e47-bb89-bb9d864d62a7)
![Screenshot (141)](https://github.com/user-attachments/assets/064eda24-74f3-47e6-b46c-72419d1204aa)
![Screenshot (142)](https://github.com/user-attachments/assets/c41bcb43-2b78-44c3-afa3-1e3c50cbf07b)
