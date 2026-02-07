
# Spark Delta Lake + Hive Metastore Environment

A containerized big data environment featuring Apache Spark with Delta Lake support, Apache Hive with a PostgreSQL-backed metastore, Hadoop HDFS, and Hue web interface. This setup provides a complete data lakehouse architecture for local development and testing.

## 📋 Table of Contents

- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Services & Endpoints](#services--endpoints)
- [Usage Examples](#usage-examples)
- [Hive Metastore Tables](#hive-metastore-tables)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)

## 🏗️ Architecture

This environment consists of the following services:

- **Apache Hadoop**: Distributed file system (HDFS) with NameNode and DataNode
- **Apache Hive 4.0.0**: Data warehouse infrastructure with HiveServer2
- **Apache Spark 3.5.2**: Distributed processing engine with Delta Lake 3.2.0 support
- **PostgreSQL 13**: Backend database for Hive Metastore
- **PGAdmin**: Web-based PostgreSQL administration interface
- **Hue**: Web-based interface for interacting with Hadoop and Hive

## 🔧 Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- At least 8GB of available RAM
- 20GB of free disk space

## 🚀 Quick Start

1. **Clone the repository**
   ```bash
   git clone https://github.com/johan9049/spark_delta_hive_metastore.git
   cd spark_delta_hive_metastore
   ```

2. **Start the environment**
   ```bash
   docker-compose up --build
   ```

   This will:
   - Build all Docker images
   - Initialize the Hive Metastore in PostgreSQL
   - Start all services (Hadoop, Hive, Spark, PostgreSQL, PGAdmin, Hue)

3. **Access the web interfaces** (see [Services & Endpoints](#services--endpoints) below)

## 🌐 Services & Endpoints

Once the environment is running, you can access the following services:

| Service | URL | Credentials |
|---------|-----|-------------|
| **Hue Web Interface** | http://localhost:8888 | Username: `admin`<br>Password: `admin` |
| **NameNode Web UI** | http://localhost:9870 | - |
| **DataNode Web UI** | http://localhost:9864 | - |
| **PGAdmin** | http://localhost:8081 | Email: `admin@admin.com`<br>Password: `admin` |
| **HiveServer2 (Beeline)** | `jdbc:hive2://localhost:10000` | - |
| **PostgreSQL** | `localhost:5432` | Username: `hiveuser`<br>Password: `hivepassword`<br>Database: `metastore` |

### Connecting via Command Line

**Beeline (Hive CLI):**
```bash
beeline -u jdbc:hive2://localhost:10000
```

**PostgreSQL (psql):**
```bash
psql -h localhost -p 5432 -U hiveuser -d metastore
```

## 📝 Usage Examples

### Creating and Managing Hive Tables

Connect to Hive using Beeline or Hue, then execute the following:

```sql
-- Create a database
CREATE DATABASE sample_db;

-- Show all databases
SHOW DATABASES;

-- Use the database
USE sample_db;

-- Create a table
CREATE TABLE employees (
    emp_id INT,
    name STRING,
    position STRING,
    salary FLOAT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;

-- Insert sample data
INSERT INTO employees VALUES
    (1, 'John Doe', 'Software Engineer', 75000.00),
    (2, 'Jane Smith', 'Data Scientist', 95000.00),
    (3, 'Mike Johnson', 'DevOps Engineer', 85000.00);

-- Query the table
SELECT * FROM employees;
```

## 🗄️ Hive Metastore Tables

The Hive Metastore stores metadata in PostgreSQL. Key tables include:

| Table | Description |
|-------|-------------|
| `DBS` | Contains information about Hive databases |
| `TBLS` | Contains information about Hive tables |
| `SDS` | Stores table storage descriptors |
| `COLUMNS_V2` | Contains table column details (schema) |
| `PARTITIONS` | Stores partition details for partitioned tables |
| `BUCKETING_COLS` | Information about bucketed columns |

### Querying Metastore Metadata

Connect to PostgreSQL and execute these queries:

```sql
-- View all databases
SELECT * FROM public."DBS";

-- View tables in a specific database
SELECT * FROM TBLS 
WHERE db_id IN (SELECT db_id FROM DBS WHERE name = 'sample_db');

-- View specific table details
SELECT * FROM TBLS 
WHERE db_id = (SELECT db_id FROM DBS WHERE name = 'sample_db') 
  AND tbl_name = 'employees';

-- View columns for a specific table
SELECT * FROM COLUMNS_V2 
WHERE cd_id = (
    SELECT sd_id FROM SDS 
    WHERE tbl_id = (
        SELECT tbl_id FROM TBLS WHERE tbl_name = 'employees'
    )
);
```

## ⚙️ Configuration

### Hue Configuration

Hue automatically detects HiveServer2 through the internal Docker network. If you need to modify the configuration, edit `hue/desktop/conf/hue.ini`:

```ini
[beeswax]
hive_server_host=namenode
hive_server_port=10000
```

### Spark with Delta Lake

The Spark container includes:
- Delta Lake 3.2.0 for ACID transactions
- PostgreSQL JDBC driver for Hive Metastore connectivity
- Unity Catalog 0.2.0-SNAPSHOT (experimental)
- Pre-configured `hive-site.xml` for metastore integration

## 🔍 Troubleshooting

### Check if HiveServer2 is running

```bash
docker exec -it hive-server ps aux | grep hiveserver2
```

### View container logs

```bash
# Hive server logs
docker logs hive-server

# NameNode logs
docker logs namenode

# DataNode logs
docker logs datanode

# PostgreSQL logs
docker logs hive-metastore-postgres
```

### Restart a specific service

```bash
docker-compose restart <service-name>
# Example:
docker-compose restart hive
```

### Clean restart (removes all data)

```bash
docker-compose down -v
docker-compose up --build
```

### Cannot connect to Hive

1. Ensure all services are running: `docker-compose ps`
2. Check NameNode is accessible: Visit http://localhost:9870
3. Verify PostgreSQL metastore: `docker logs hive-metastore-postgres`
4. Check Hive server logs: `docker logs hive-server`

## 📦 What's Included

### Docker Images

- **Hadoop**: Debian-based image with Hadoop 3.4.0
- **Hive**: Debian-based image with Hive 4.0.0 and Hadoop 3.4.0
- **Spark**: Python 3.11-based image with Spark 3.5.2, Delta Lake 3.2.0, and Jupyter support
- **PostgreSQL**: Official PostgreSQL 13 image
- **PGAdmin**: Official dpage/pgadmin4 image
- **Hue**: Official gethue/hue image

### JAR Dependencies (Spark)

- `delta-spark_2.13-3.2.0.jar`
- `delta-storage-3.2.0.jar`
- `postgresql-42.7.4.jar`
- `unitycatalog-spark-0.2.0-SNAPSHOT.jar`

## 📄 License

This project is provided as-is for educational and development purposes.

## 🤝 Contributing

Feel free to open issues or submit pull requests for improvements!