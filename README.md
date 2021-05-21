# aitflow-setup-on-ubuntu

# Step 1: Stand up the EC2 Instance
  Launch EC2 instance:
    1. Select Ubuntu AMI
    2. t2.medium
    3. Open 8080 with in Security group
# Step 2: Install Postgres Server on the EC2 Instance
  By default airflow uses sqlite to store metadata but if you want a fairly robust installation you want to use postgres database to store all the metadata.
  
    sudo apt-get update
    sudo apt-get install python-psycopg2
    sudo apt-get install postgresql postgresql-contrib
   
  Also create an os user called "airflow" to do the rest of the installation
  
    sudo adduser airflow
    sudo usermod -aG sudo airflow
    su - airflow
  
  Provide sudo permissions for "airflow" user
    
    sudo visudo
    -----------------------------
    airflow ALL=(ALL:ALL) ALL
    
  Once the postgres database is installed it’s time to create database and users to access the database.
  
    sudo -u postgres psql
    
  Execute the following commands to create the airflow database and user
    
    CREATE USER airflow PASSWORD 'airflow';
    CREATE DATABASE airflow;
    GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO airflow;
    \q
  
  Next we need to set a few config settings to ensure the airflow server can connect to the postgres database. Find the pg_hba.conf file and edit it.
  
    sudo vi /etc/postgresql/12/main/pg_hba.conf
  Find the below line:
  
    # IPv4 local connections:
    host    all             all             127.0.0.1/32         md5
  and change it to:
    
    # IPv4 local connections:
    host    all             all             0.0.0.0/0            trust
  
  Next open the following config file:
    
    sudo vi /etc/postgresql/12/main/postgresql.conf
  Uncomment the following lines
    
    #listen_addresses = 'localhost' # what IP address(es) to listen on
  to
    
    listen_addresses = '*' # what IP address(es) to listen on
  Restart the postgresql server to get all the config changes
    
    sudo service postgresql restart
# Step 3: Install Airflow server
  Next we will install airflow server
  
    su - airflow
    sudo apt-get install python3-pip -y
    sudo python3 -m pip install apache-airflow[postgres,s3,aws,azure,gcp,slack]
  Initilise Database using below command:
  
    airflow db init
# Step 4: Connect Airflow to Postgresql
  Next we will work on connecting the airflow server to the Postgresql database as its metadata database.
  
    cd /home/airflow/airflow
    vi airflow.cfg
    -------------------------------------------------------------------------------
    sql_alchemy_conn = postgresql+psycopg2://airflow:a1rfl0w@localhost:5432/airflow
    executor = LocalExecutor
    load_examples = False
  
  Start Scheduler and Webservers:
  
    airflow scheduler
    airflow webser
# Step 5: Create dags
    cd /home/airflow/airflow
    mkdir dags
    cd dags
    vi helloworld.py
  
