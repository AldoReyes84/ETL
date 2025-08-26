# ETL

Create an Integreation package that updates a copy of the FactInternet Sales table  with Face Sales Data createed in Python

## Create a copy table for SQL Server AdventureWorksDW2022.FactInternetSales table.

<br>

<img width="634" height="91" alt="image" src="https://github.com/user-attachments/assets/31072e54-5a4a-47e7-a3ca-ff9cc68b1a77" />

<br>

## Create a fake daily Internet Sales report file

### Prepare a review of the field to be created 

I used the Excel Data Model connected to Adventureworks FactInternetSales table to prepare the parameter to use Python Faker to create some random InternetSales data. 

<br>

<img width="622" height="222" alt="image" src="https://github.com/user-attachments/assets/7e9e3e2c-a70b-4aaa-a17e-de7cf8a02555" />


<br>

We are no trying to recreate an exact structure the SQL table because we want to do some transforations in the SSIS Data Flow. The last five fields will be added in the transformation step. 


-----------------

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
        
### # Get the highest SalesOrderNumber with prefix 'SO'   
           
        cursor.execute("""
        SELECT MAX(CAST(SUBSTRING(SalesOrderNumber, 3, LEN(SalesOrderNumber)) AS INT))
        FROM FactInternetSales2
        WHERE SalesOrderNumber LIKE 'SO%'
        """)
        last_order_number = cursor.fetchone()[0]
        order_counter = (last_order_number or 0) + 1

### Get ListPrice and StandardCost from DimProduct

        cursor.execute("""
        SELECT ProductKey, ListPrice, StandardCost
        FROM DimProduct
        WHERE ProductKey IS NOT NULL
        """)

### Store price data in a dictionary for quick lookup
        product_prices = {}
        for row in cursor.fetchall():
        product_key = row.ProductKey
        list_price = row.ListPrice if row.ListPrice is not None else 0
        standard_cost = row.StandardCost if row.StandardCost is not None else 0
        product_prices[product_key] = {
        'UnitPrice': list_price,
        'ProductStandardCost': standard_cost
        }

        conn.close()

# Define date range for order generation

        current_date = start_date
        end_date = datetime(2014, 2, 3)

This script will read the last order date in FactInternetSales2 and set it up as start_date

end_date will be provided manually to determin how many daily files will this scritp generate

### Define lookup values

        product_options = [214, 217, 222, 225, 228, 231, 234, 237, 310, 311, 312, 313, 314, 320, 321, 322, 323, 324, 325,             326, 327, 328, 329, 330, 331, 332, 333, 334, 335, 336, 337, 338, 339, 340, 341, 342, 343, 344, 345, 346, 347, 348,            349, 350, 351, 352, 353, 354, 355, 356, 357, 358, 359, 360, 361, 362, 363, 368, 369, 370, 371, 372, 373, 374, 375,            376, 377, 378, 379, 380, 381, 382, 383, 384, 385, 386, 387, 388, 389, 390, 463, 465, 467, 471, 472, 473, 474, 475,            476, 477, 478, 479, 480, 481, 482, 483, 484, 485, 486, 487, 488, 489, 490, 491, 528, 529, 530, 535, 536, 537, 538,            539, 540, 541, 560, 561, 562, 563, 564, 565, 566, 567, 568, 569, 570, 571, 572, 573, 574, 575, 576, 577, 578, 579,            580, 581, 582, 583, 584, 585, 586, 587, 588, 589, 590, 591, 592, 593, 594, 595, 596, 597, 598, 599, 600, 604, 605,            606]
        promotion_options = [1, 2, 13, 14]
        currency_options = [6, 19, 29, 39, 98, 100]

### Initialize list to store order records

        orders = []

### Loop through each day in the date range

        while current_date <= end_date:

### Generate a random number of orders for the current day

        num_orders = random.randint(1, 8)

        for _ in range(num_orders):
        product_key = random.choice(product_options)
        price_info = product_prices.get(product_key, {'UnitPrice': 0, 'ProductStandardCost': 0})

        order_id = f"SO{order_counter}"

        tax_rate = Decimal('0.08')

        order_date = current_date.date()
        order_date_key = int(current_date.strftime('%Y%m%d'))  # Formato YYYYMMDD como entero

### Generate DueDate: OrderDate + 4 to 30 days
     
        due_date = order_date + timedelta(days=random.randint(4, 30))
        due_date_key = int(due_date.strftime('%Y%m%d'))

### Generate ShipDate: DueDate + 2 to 15 days
  
        ship_date = due_date + timedelta(days=random.randint(2, 15))
        ship_date_key = int(ship_date.strftime('%Y%m%d'))

### Create Fields Dictionary

        orders.append({
            'ProductKey': product_key,                                # Product identifier
            'OrderDateKey': order_date_key,                           # Order date as integer YYYYMMDD
            'DueDateKey': due_date_key,                               # Due date as integer YYYYMMDD
            'ShipDateKey': ship_date_key,                             # Ship date as integer YYYYMMDD
            'CustomerKey': random.randint(11000, 29483),              # Customer identifier
            'PromotionKey': random.choice(promotion_options),         # Promotion applied
            'CurrencyKey': random.choice(currency_options),           # Currency used
            'SalesTerritory': random.randint(1, 10),                  # Sales territory ID
            'SalesOrderNumber': order_id,                             # Unique order ID
            'SalesOrderLine': random.randint(1, 8),                   # Line number within the order
            'RevisoryNumber': None,                                   # Revisory number null
            'SalesOrderQuantity': 1,                                  # Fixed quantity per order
            'UnitPrice': price_info['UnitPrice'],                     # ListPrice from DimProduct
            'ExtendedAmount': price_info['UnitPrice'],                # Extended amount = UnitPrice
            'UnitPriceDiscount': 0,                                   # Unit price discount = 0
            'ProductStandardCost': price_info['ProductStandardCost'], # StandardCost from DimProduct
            'TotalProductCost': price_info['ProductStandardCost'],    # Total product cost = ProductStandardCost
            'SalesAmount': price_info['UnitPrice'],                   # Sales amount = UnitPrice
            'TaxAmount': price_info['UnitPrice'] * Decimal('0.08'),   # Tax amount = 8% of SalesAmount
            'Freight': None,                                          # Freight requires another DB for practical purposes won't implement it here
            'OrderDate': current_date.date(),                         # Date of the order
            'DueDate': due_date,                                     # Due date
            'ShipDate': ship_date,                                   # Ship date
        })
            
        order_counter += 1  # Increment global order ID

        current_date += timedelta(days=1)  # Move to the next day

### Convert list of orders to DataFrame

        df = pd.DataFrame(orders)

### Group by OrderDate

        grouped = df.groupby('OrderDate')

### Display the resulting DataFrame
        
        print(df)

### Define output folder and ensure it exists
        
        output_folder = r'C:\Users\aldoa\Documents\Proyectos\Python Faker\FakeInternetSalesOrders\\'
        os.makedirs(output_folder, exist_ok=True)

### Iterate for each group (day)

        for order_date, group in grouped:
        # Convert the date to YYYYMMDD format
        date_str = order_date.strftime('%Y%m%d')

        # Define the full path for the file
        filename = f"{output_folder}FakeOrders_{date_str}.csv"

### Save the group as CSV in the correct folder
    
        group.to_csv(filename, index=False, encoding='utf-8')

### Resulst

<img width="1432" height="368" alt="image" src="https://github.com/user-attachments/assets/d3ba20e2-567d-484d-9849-4d941a59535b" />

<br>
<br>

<img width="665" height="279" alt="image" src="https://github.com/user-attachments/assets/7f7ce703-800b-47a1-96fc-e319a4e4f373" />

<br>
<br>

-------
## SQL Server Integration Services (SSIS)

In Visual Studion 2022 create an Integreation Project

Create a New - Foreach Loop Container, set up for Foreach File Emulator, provide the Path to our Faker csv files folder, assign a variable and inside add a Data Flow Task 

<img width="1159" height="718" alt="image" src="https://github.com/user-attachments/assets/2270bb45-8a73-4cd2-b01f-92337eae2b79" />


<br>

On the Data Flow Task add a Flat File, select one of the FakeOrder files and accept to create a new connection 

<img width="886" height="746" alt="image" src="https://github.com/user-attachments/assets/6b42d48b-119c-4522-a0b2-c79fc1a061af" />

### Prepare the Data Transformation

Compare the FactInternetSales2 table and the FakeOrder files headers 

The First fields match ok 

<img width="1296" height="171" alt="image" src="https://github.com/user-attachments/assets/def60e63-f00c-44b7-9fe5-3086c9256f18" />


Add a OLE DB Source to integrate FactInternetSales2

<img width="343" height="224" alt="image" src="https://github.com/user-attachments/assets/bb1de104-795a-4d29-8582-9ecacb2dcb34" />

Connect to our SSMS Server and our DataBase AdventureWorksDW2022

<img width="873" height="731" alt="image" src="https://github.com/user-attachments/assets/29a20f14-913a-4892-9096-4089a1b3a81e" />




