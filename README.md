
# Spark Delta Lake with Hive Metastore

A complete Docker-based development environment for Apache Spark with Delta Lake support, Hive Metastore, Hadoop HDFS, and Unity Catalog integration. This setup provides a fully functional big data stack for local development and testing.

## 🏗️ Architecture

This project sets up a complete data lakehouse environment with the following components:

- **Apache Spark 3.5.2** with Delta Lake 3.2.0 support
- **Apache Hive** with HiveServer2 and Hive Metastore
- **Hadoop HDFS** (NameNode and DataNode)
- **PostgreSQL 13** as the Hive Metastore backend
- **Hue** web interface for database and query management
- **Unity Catalog 0.2.0** for unified data governance
- **PGAdmin** for PostgreSQL administration

## 📋 Prerequisites

- Docker (20.10 or higher)
- Docker Compose (2.0 or higher)
- At least 8GB of available RAM
- 10GB of free disk space

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/johan9049/spark_delta_hive_metastore.git
cd spark_delta_hive_metastore
```

### 2. Download Required Binaries

Before building, ensure you have the following files in the `downloads/` directory:

- `spark-3.5.2-bin-hadoop3-scala2.13.tgz`
- `delta-spark_2.13-3.2.0.jar`
- `delta-storage-3.2.0.jar`
- `postgresql-42.7.4.jar`
- `unitycatalog-spark-0.2.0-SNAPSHOT.jar`
- `hive-site.xml`

### 3. Start the Environment

```bash
docker-compose up --build
```

Wait for all services to initialize. This may take a few minutes on first run.

## 🌐 Service Endpoints

Once the environment is running, access the following web interfaces:

| Service | URL | Credentials |
|---------|-----|-------------|
| **Hue Web UI** | http://localhost:8888 | admin / admin |
| **NameNode UI** | http://localhost:9870 | - |
| **DataNode UI** | http://localhost:9864 | - |
| **PGAdmin** | http://localhost:8081 | admin@admin.com / admin |

### Command-Line Access

**Connect to PostgreSQL Metastore:**
```bash
psql -h localhost -p 5432 -U hiveuser -d metastore
```
Password: `hivepassword`

**Connect to HiveServer2 via Beeline:**
```bash
beeline -u jdbc:hive2://localhost:10000
```

## 📖 Usage Examples

### Creating a Database and Table in Hive

Connect to HiveServer2 using Beeline or Hue, then execute:

```sql
-- Create a new database
CREATE DATABASE sample_db;

-- Show all databases
SHOW DATABASES;

-- Use the database
USE sample_db;

-- Create a sample table
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

-- Query the data
SELECT * FROM employees;
```

### Querying Hive Metastore Tables

Connect to PostgreSQL to explore the Hive Metastore schema:

```sql
-- View all databases
SELECT * FROM public."DBS";

-- View tables in a specific database
SELECT * FROM TBLS 
WHERE db_id IN (SELECT db_id FROM DBS WHERE name = 'sample_db');

-- View table schema/columns
SELECT * FROM COLUMNS_V2 
WHERE cd_id IN (
    SELECT sd_id FROM SDS 
    WHERE tbl_id IN (
        SELECT tbl_id FROM TBLS 
        WHERE tbl_name = 'employees'
    )
);

-- Get complete table information
SELECT * FROM TBLS 
WHERE db_id = (SELECT db_id FROM DBS WHERE name = 'sample_db') 
AND tbl_name = 'employees';
```

### Key Hive Metastore Tables

| Table | Description |
|-------|-------------|
| `DBS` | Contains information about Hive databases |
| `TBLS` | Contains information about Hive tables |
| `SDS` | Stores table storage descriptors |
| `COLUMNS_V2` | Contains table column definitions and schema |
| `PARTITIONS` | Stores partition information for partitioned tables |
| `BUCKETING_COLS` | Information about bucketed columns |

## 🔧 Configuration

### Hue Configuration

Hue automatically detects HiveServer2 through the Docker network. If you need to modify the configuration:

1. Edit `hue/desktop/conf/hue.ini` on your host machine
2. Update the HiveServer2 settings:

```ini
[beeswax]
hive_server_host=namenode
hive_server_port=10000
```

3. Restart the Hue container

### Spark with Delta Lake

The Spark service includes Delta Lake jars and is configured to work with the Hive Metastore. Configuration files are located in the `config/` directory:

- `core-site.xml` - Hadoop core configuration
- `hdfs-site.xml` - HDFS configuration
- `hive-site.xml` - Hive configuration
- `hadoop-env.sh` - Hadoop environment variables

## 🐛 Troubleshooting

### Check if HiveServer2 is Running

```bash
docker exec -it hive-server ps aux | grep hiveserver2
```

### View Container Logs

```bash
# View all logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f hive
docker-compose logs -f postgres
docker-compose logs -f namenode
```

### Reset the Environment

If you encounter issues, you can reset the entire environment:

```bash
# Stop and remove all containers
docker-compose down

# Remove volumes (WARNING: This deletes all data)
docker-compose down -v

# Rebuild and start fresh
docker-compose up --build
```

### Common Issues

**Issue:** Cannot connect to HiveServer2
- **Solution:** Wait a few minutes for HiveServer2 to fully initialize, then try again

**Issue:** PostgreSQL connection refused
- **Solution:** Ensure the postgres container is running: `docker ps | grep postgres`

**Issue:** Out of memory errors
- **Solution:** Increase Docker's memory allocation to at least 8GB

## 📁 Project Structure

```
.
├── config/                 # Hadoop and Hive configuration files
│   ├── core-site.xml
│   ├── hdfs-site.xml
│   ├── hive-site.xml
│   └── hadoop-env.sh
├── postgres/              # PostgreSQL initialization scripts
│   └── init-hive-metastore.sql
├── hue/                   # Hue configuration files
├── scripts/               # Utility scripts
│   ├── entrypoint.sh
│   └── hive_entrypoint.sh
├── downloads/             # Required binaries and JARs
├── hadoop.Dockerfile      # Hadoop/HDFS Dockerfile
├── hive.Dockerfile        # Hive Dockerfile
├── spark.Dockerfile       # Spark Dockerfile
├── docker-compose.yml     # Docker Compose configuration
└── README.md
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is provided as-is for educational and development purposes.

## 🔗 References

- [Apache Spark Documentation](https://spark.apache.org/docs/latest/)
- [Delta Lake Documentation](https://docs.delta.io/)
- [Apache Hive Documentation](https://hive.apache.org/)
- [Hadoop Documentation](https://hadoop.apache.org/docs/)
- [Unity Catalog](https://github.com/unitycatalog/unitycatalog)

