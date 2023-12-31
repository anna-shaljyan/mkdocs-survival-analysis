# Database

### ERD Structure

![Database ERD Structure](ERD.jpg)

### Overview of Tables

#### DimCustomer Table

![DimCustomer Table](DimCustomer_table_overview.png)

#### FactPredictions Table

![FactPredictions Table](FactPredictions_table_overview.png)

#### FactPushNotification Table

![FactPushNotification Table](FactPushNotification_table_overview.png)

#### FactEmail Table

![FactEmail Table](FactEmail_table_overview.png)

### Code parts

```py title="schema.py"
"""
Module: schema.py

This module contains Python code for defining and creating a database schema using SQLAlchemy. 
It defines four tables: 'DimCustomer', 'FactPredictions' 'FactPushNotification' and 'FactEmail'.

It also configures a custom logger for informational messages regarding the schema creation.

"""

import logging
import os
from ..logger import CustomFormatter

# Initialize and configure the logger
logger = logging.getLogger(os.path.basename(__file__))
logger.setLevel(logging.DEBUG)
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
ch.setFormatter(CustomFormatter())
logger.addHandler(ch)

from sqlalchemy import create_engine, Column, Integer, String, Float, DATE, DateTime, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship
from datetime import datetime

# Define and configure the database engine (Change the connection URL as needed)
engine = create_engine('sqlite:///sa_db.db')

# Create a base class for declarative class definitions
Base = declarative_base()

class Customer(Base):
    """
    Class: Customer

    This class defines the 'DimCustomer' table, which represents customer information.

    Attributes:
    - Customer_ID (int): Primary key for the customer.
    - Age (int): Customer's age.
    - Tenure (int): Customer's tenure with the company.
    - Gender (str): Customer's gender.
    - Income (int): Customer's income.
    - Marital_Status (str): Customer's marital status.
    - Address_ID (int): Customer's address identifier.
    - Education (str): Customer's education level.
    - Retirement (str): Customer's retirement status.
    - Churn (str): Customer's churn status.
    - Region (str): Customer's region.
    - Service_Category (str): Service category.
    - Voice_Included (str): Voice service inclusion status.
    - Internet_Included (str): Internet service inclusion status.
    - Forward_Included (str): Forwarding service inclusion status.
    """
    __tablename__ = "DimCustomer"

    Customer_ID = Column(Integer, primary_key=True)
    Age = Column(Integer)
    Tenure = Column(Integer)
    Gender = Column(String(10))
    Income = Column(Integer)
    Marital_Status = Column(String(15))
    Address_ID = Column(Integer)
    Education = Column(String(45))
    Retirement = Column(String(3))
    Churn = Column(String(3))
    Region = Column(String(15))
    Service_Category = Column(String(15))
    Voice_Included = Column(String(3))
    Internet_Included = Column(String(3))
    Forward_Included = Column(String(3))

class FactPredictions(Base):
    """
    Class: FactPredictions

    This class defines the 'FactPredictions' table, which stores predictive information related to customers.

    Attributes:
    - pred_period (int): Primary key representing the prediction period.
    - customer_ID (int): Foreign key referencing the 'DimCustomer' table.
    - CLV (float): Customer Lifetime Value.
    - Churn_Rate (float): Churn rate.
    - customer (relationship): Establishes a relationship with the 'DimCustomer' table.

    """
    __tablename__ = "FactPredictions"

    pred_period = Column(Integer, primary_key=True)
    customer_ID = Column(Integer, ForeignKey('DimCustomer.Customer_ID'), primary_key=True)
    CLV = Column(Float)
    Churn_Rate = Column(Float)
    customer = relationship("DimCustomer")

class FactPushNotification(Base):
    """
    Class: FactPushNotification

    This class defines the 'FactPushNotification' table, which stores information about the push notifications sent to customers.

    Attributes:
    - sent_date (DateTime): Primary key representing the date the push notification was sent.
    - customer_ID (int): Foreign key referencing the 'DimCustomer' table.
    - success (int): A binary column representing wether the push notification was sent successfully or not.
    - customer (relationship): Establishes a relationship with the 'DimCustomer' table.

    """
    __tablename__ = "FactPushNotification"

    sent_date = Column(DateTime, primary_key=True)
    customer_ID = Column(Integer, ForeignKey('DimCustomer.Customer_ID'), primary_key=True)
    success = Column(Integer)
    customer = relationship("DimCustomer")

class FactEmail(Base):
    """
    Class: FactEmail

    This class defines the 'FactEmail' table, which stores information about the emails sent to customers.

    Attributes:
    - sent_date (DateTime): Primary key representing the date the push notification was sent.
    - customer_ID (int): Foreign key referencing the 'DimCustomer' table.
    - success (int): A binary column representing wether the push notification was sent successfully or not.
    - customer (relationship): Establishes a relationship with the 'DimCustomer' table.

    """
    __tablename__ = "FactEmail"

    sent_date = Column(DateTime, primary_key=True)
    customer_ID = Column(Integer, ForeignKey('DimCustomer.Customer_ID'), primary_key=True)
    success = Column(Integer)
    customer = relationship("DimCustomer")


# Create the tables defined in the schema
Base.metadata.create_all(engine)

# Log a message indicating that the schema has been created
logger.info("Schema Has Been Created")
```

```py title="sql_interactions.py"
"""
Module: sql_interactions.py

This module defines a Python class called 'SqlHandler' for interacting with SQLite databases. The class allows for various operations on SQLite databases, such as connecting, inserting data, retrieving data, and more.

Attributes:
- dbname (str): The name of the SQLite database.
- table_name (str): The name of the table within the database.

Methods:

- __init__(self, dbname: str, table_name: str) -> None:
    Constructor for the 'SqlHandler' class. Initializes a connection to an SQLite database and specifies the target table.

- close_cnxn(self) -> None:
    Closes the SQLite database connection.

- insert_one(self, data) -> str:
    Inserts a single row to the database. 

- get_table_columns(self) -> list:
    Retrieves a list of column names for the specified table.

- truncate_table(self) -> None:
    Truncates the specified table, removing all its data.

- drop_table(self) -> None:
    Deletes the specified table from the database.

- insert_many(self, df: pd.DataFrame) -> str:
    Inserts data from a pandas DataFrame into the specified table, mapping columns to their respective database columns.

- from_sql_to_pandas(self, chunksize: int, id_value: str) -> pd.DataFrame:
    Retrieves data from the database and loads it into a pandas DataFrame in chunks of a specified size.

- update_table(self, set_dict, cond_dict) -> str:
    Update rows in a database table based on the set and where conditions provided in dictionaries.


"""

import sqlite3
import logging
import pandas as pd
import numpy as np
import os
from ..logger import CustomFormatter

# Initialize and configure the logger
logger = logging.getLogger(os.path.basename(__file__))
logger.setLevel(logging.DEBUG)
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
ch.setFormatter(CustomFormatter())
logger.addHandler(ch)

class SqlHandler:

    def __init__(self, dbname: str, table_name: str) -> None:
        """
        Constructor for the SqlHandler class.

        Parameters:
        - dbname (str): The name of the SQLite database.
        - table_name (str): The name of the table within the database.
        """
        self.cnxn = sqlite3.connect(f'{dbname}.db')
        self.cursor = self.cnxn.cursor()
        self.dbname = dbname
        self.table_name = table_name

    def close_cnxn(self) -> None:
        """
        Closes the SQLite database connection.
        
        """
        logger.info('Committing the changes')
        self.cnxn.close()
        logger.info('The connection has been closed')

    def insert_one(self, data: dict) -> str:
        """
        Inserts a single record from a dictionary into the specified table, mapping keys to their respective database columns.

        Parameters:
        - data (dict): A dictionary containing data to be inserted.

        Returns:
        - str: A message indicating that the data has been loaded.
        """

        #getting the DB columns

        columns = self.get_table_columns()
        sql_column_names = [col.lower() for col in columns]

        # Convert keys to lowercase
        data = {key.lower(): value for key, value in data.items()}

        # Filter the dictionary to keep only keys that match the database column names
        filtered_data = {key: value for key, value in data.items() if key in sql_column_names}
        
        # Prepare the values to be inserted
        row_values = list(filtered_data.values())
        columns = list(filtered_data.keys())
        
        ncolumns = ['?' for _ in columns]
        cols = ', '.join(columns)
        params = ', '.join(ncolumns)

        logger.info(f'Insert structure: colnames: {cols} params: {params}')

        query = f"""INSERT INTO {self.table_name} ({cols}) VALUES ({params});"""

        self.cursor.execute(query, row_values)
    
        try:
            logger.info(self.cursor.messages)
        except:
            pass

        self.cnxn.commit()
        logger.warning('The data is loaded')


    def get_table_columns(self) -> list:
        """
        Retrieves a list of column names for the specified table.

        Returns:
        - column_names (list): List of column names in the table.
        """
        self.cursor.execute(f"PRAGMA table_info({self.table_name});")
        columns = self.cursor.fetchall()
        column_names = [col[1] for col in columns]
        logger.info(f'The list of columns: {column_names}')
        return column_names
    
    def truncate_table(self) -> None:
        """
        Truncates the specified table, removing all its data.
        """
        query = f"DELETE FROM {self.table_name};"
        self.cursor.execute(query)
        self.cnxn.commit()
        logging.info(f'The {self.table_name} is truncated')
        self.cursor.close()

    def drop_table(self) -> None:
        """
        Deletes the specified table from the database.
        """
        query = f"DROP TABLE IF EXISTS {self.table_name};"
        logging.info(query)
        self.cursor.execute(query)
        self.cnxn.commit()
        logging.info(f"Table '{self.table_name}' deleted.")
        logger.debug('Using drop table function')

    def insert_many(self, df: pd.DataFrame) -> str:
        """
        Inserts data from a pandas DataFrame into the specified table, mapping columns to their respective database columns.

        Parameters:
        - df (pd.DataFrame): The DataFrame containing data to be inserted.

        Returns:
        - str: A message indicating that the data has been loaded.
        """
        df = df.replace(np.nan, None)  # For handling NULLS
        df.rename(columns=lambda x: x.lower(), inplace=True)
        columns = list(df.columns)
        logger.info(f'BEFORE the column intersection: {columns}')
        sql_column_names = [i.lower() for i in self.get_table_columns()]
        columns = list(set(columns) & set(sql_column_names))
        logger.info(f'AFTER the column intersection: {columns}')
        ncolumns = list(len(columns) * '?')
        data_to_insert = df.loc[:, columns]
        values = [tuple(i) for i in data_to_insert.values]
        logger.info(f'The shape of the table which is going to be imported {data_to_insert.shape}')
        if len(columns) > 1:
            cols, params = ', '.join(columns), ', '.join(ncolumns)
        else:
            cols, params = columns[0], ncolumns[0]
        logger.info(f'Insert structure: colnames: {cols} params: {params}')
        logger.info(values[0])
        query = f"""INSERT INTO {self.table_name} ({cols}) VALUES ({params});"""
        logger.info(f'QUERY: {query}')
        self.cursor.executemany(query, values)
        try:
            for i in self.cursor.messages:
                logger.info(i)
        except:
            pass
        self.cnxn.commit()
        logger.warning('The data is loaded')

    def from_sql_to_pandas(self, chunksize: int, id_value: str) -> pd.DataFrame:
        """
        Retrieves data from the database and loads it into a pandas DataFrame in chunks of a specified size.

        Parameters:
        - chunksize (int): The chunksize for data extraction.
        - id_value (str): The values by which the data should be sorted.

        Returns:
        - pd.DataFrame: A DataFrame containing the retrieved data.
        """
        offset = 0
        dfs = []
        while True:
            query = f"""
            SELECT * FROM {self.table_name}
                ORDER BY {id_value} 
                LIMIT {chunksize}
                OFFSET {offset};
            """
            data = pd.read_sql_query(query, self.cnxn)
            logger.info(f'The shape of the chunk: {data.shape}')
            dfs.append(data)
            offset += chunksize
            if len(dfs[-1]) < chunksize:
                logger.warning('Loading the data from SQL is finished')
                logger.debug('Connection is closed')
                break
        df = pd.concat(dfs)
        return df

   
    def update_table(self, set_dict: dict, cond_dict: dict) -> str:
        """
        Update rows in a database table based on the set and where conditions provided in dictionaries.

        Parameters:
        - set_dict (dict): A dictionary with column names as keys and new values as values for the SET clause.
        - cond_dict (dict): A dictionary with column names as keys and conditions as values for the WHERE clause.

        Returns:
        - str: A message indicating that the data has been updated.
        """
        
        #getting the db column names
        columns = self.get_table_columns()
        sql_column_names = [col.lower() for col in columns]

        # Convert keys to lowercase
        set_dict = {key.lower(): value for key, value in set_dict.items()}
        cond_dict = {key.lower(): value for key, value in cond_dict.items()}

        # Filter the dictionary to keep only keys that match the database column names
        set_dict = {key: value for key, value in set_dict.items() if key in sql_column_names}
        cond_dict = {key: value for key, value in cond_dict.items() if key in sql_column_names}    
        
        try:
            # Generate the SET clause
            set_clause = ', '.join([f"{col} = ?" for col in set_dict.keys()])
            set_values = list(set_dict.values())
            logger.info(f'Set Clause Structure: {set_clause}')

            # Generate the WHERE clause
            where_clause = ' AND '.join([f"{col} = ?" for col in cond_dict.keys()])
            where_values = list(cond_dict.values())  # Add values for the WHERE clause
            logger.info(f'Where Clause Structure: {where_clause}')

            # Build the SQL query
            query = f"""
                UPDATE {self.table_name}
                SET {set_clause}
                WHERE {where_clause};
                    """
            logger.info(f'Generated SQL query: {query}')  # Log the generated query
            self.cursor.execute(query, set_values + where_values)  # Combine SET and WHERE values

            self.cnxn.commit()
            logger.warning(f'The table {self.table_name} is updated.')
        except Exception as e:
            logger.warning(f"Error updating rows: {e}")
```



