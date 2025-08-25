# ETL

Create a copy table for SQL Server AdventureWorksDW2022.FactInternetSales table.

<img width="634" height="91" alt="image" src="https://github.com/user-attachments/assets/31072e54-5a4a-47e7-a3ca-ff9cc68b1a77" />

Use SQL Server Integration Service (SSIS) enable in Visual Studion to create a Integreation Project

Create a New Data Flow Task

<img width="644" height="292" alt="image" src="https://github.com/user-attachments/assets/a5fabbf1-c108-4577-87a8-670b4da7a001" />

Add a OLE DB Source to integrate FactInternetSales2

<img width="343" height="224" alt="image" src="https://github.com/user-attachments/assets/bb1de104-795a-4d29-8582-9ecacb2dcb34" />

Connect to our SSMS Server and our DataBase AdventureWorksDW2022

<img width="873" height="731" alt="image" src="https://github.com/user-attachments/assets/29a20f14-913a-4892-9096-4089a1b3a81e" />


# Create a fake daily Internet Sales report file

Prepare a review of the field we will create 

I used the Excel Data Model connected to Adventureworks FactInternetSales table to prepare the parameter to use Python Faker to create some random InternetSales data. 

<img width="1292" height="641" alt="image" src="https://github.com/user-attachments/assets/8f23bd6f-329b-4c15-a560-368796ea4847" />

We are no trying to recreate an exact structure the SQL table because we want to do some transforations on the Data Flow. The last five fields will be added in the transformation step. 


