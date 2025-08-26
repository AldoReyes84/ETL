# ETL

Create an Integreation package that updates a copy of the FactInternet Sales table  with Face Sales Data createed in Python

## Create a copy table for SQL Server AdventureWorksDW2022.FactInternetSales table.

<br>

<img width="634" height="91" alt="image" src="https://github.com/user-attachments/assets/31072e54-5a4a-47e7-a3ca-ff9cc68b1a77" />


## Create a fake daily Internet Sales report file

### Prepare a review of the field to be created 

I used the Excel Data Model connected to Adventureworks FactInternetSales table to prepare the parameter to use Python Faker to create some random InternetSales data. 

<img width="1292" height="641" alt="image" src="https://github.com/user-attachments/assets/8f23bd6f-329b-4c15-a560-368796ea4847" />

We are no trying to recreate an exact structure the SQL table because we want to do some transforations in the SSIS Data Flow. The last five fields will be added in the transformation step. 

## Python

### Import Liberies

        import pandas as pd                                 # to create the Data Frame
        import random                                       # to create the Random Data
        from datetime import datetime, timedelta            # to manade Date Data
        import pyodbc                                       # to connect and query SQL Server DB Data
        from decimal import Decimal                         # to define decimal numbers for tax rate
        import os                                           # to export the final resulting files to folder


### Connect to AdventureWorksDW2022 

        conn = pyodbc.connect('DRIVER={SQL Server};SERVER="Server_Name";DATABASE=AdventureWorksDW2022;Trusted_Connection=yes;')
        cursor = conn.cursor()

### Get the last OrderDate From FactInternetSales2

        cursor.execute("SELECT MAX(OrderDate) FROM FactInternetSales2")
last_order_date = cursor.fetchone()[0]
start_date = last_order_date + timedelta(days=1)

Create a copy table for SQL Server AdventureWorksDW2022.FactInternetSales table.

<img width="634" height="91" alt="image" src="https://github.com/user-attachments/assets/31072e54-5a4a-47e7-a3ca-ff9cc68b1a77" />

Use SQL Server Integration Service (SSIS) enable in Visual Studion to create a Integreation Project

Create a New Data Flow Task

<img width="644" height="292" alt="image" src="https://github.com/user-attachments/assets/a5fabbf1-c108-4577-87a8-670b4da7a001" />

Add a OLE DB Source to integrate FactInternetSales2

<img width="343" height="224" alt="image" src="https://github.com/user-attachments/assets/bb1de104-795a-4d29-8582-9ecacb2dcb34" />

Connect to our SSMS Server and our DataBase AdventureWorksDW2022

<img width="873" height="731" alt="image" src="https://github.com/user-attachments/assets/29a20f14-913a-4892-9096-4089a1b3a81e" />




