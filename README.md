# Best Buy Case Study Relational-Database-Design


	USE master;

-- Drop database

	IF DB_ID(N'Best Buy') IS NOT NULL DROP DATABASE [Best Buy];

-- If database could not be created due to open connections, abort

	IF @@ERROR = 3702 
  		 RAISERROR(N'Database cannot be dropped because there are still open connections.', 127, 127) WITH NOWAIT, LOG;

   -- Create database
   
	CREATE DATABASE [Best Buy];
	GO
-- Use the database Created

	USE [Best Buy];
	GO

-- Create Schemas


	CREATE SCHEMA HR AUTHORIZATION dbo;
	GO
	CREATE SCHEMA Product AUTHORIZATION dbo;
	GO
	CREATE SCHEMA Geolocation AUTHORIZATION dbo;
	GO
	CREATE SCHEMA Sales AUTHORIZATION dbo;
	GO

-- Create Tables
---------------------------------------------------------------------


--Create HR.Designation

	CREATE TABLE HR.Designation
	(
	DesignationID	SMALLINT		NOT NULL IDENTITY,
	Designation		NVARCHAR(20)	NOT NULL,
	CONSTRAINT PK_Desination PRIMARY KEY (DesignationID)
	);
	CREATE NONCLUSTERED INDEX idx_nc_Designation ON HR.Designation (Designation);

-- Create HR.Department

	CREATE TABLE HR.Department
	(
	DepartmentID	SMALLINT				NOT NULL IDENTITY,
	Department		NVARCHAR(25)			NOT NULL,
	CONSTRAINT PK_Department PRIMARY KEY (DepartmentID)
	);
	CREATE NONCLUSTERED INDEX idx_nc_Department ON HR.Department (Department);

--Create Geolocation.Country

	CREATE TABLE Geolocation.Country
	(
	CountryID		INT				NOT NULL	IDENTITY,
	Country			NVARCHAR(25)	NOT NULL
		CONSTRAINT DFT_Country_Country DEFAULT ('CANADA'),
	CONSTRAINT PK_Country	PRIMARY KEY (CountryID)
	);
	CREATE NONCLUSTERED INDEX idx_nc_Country ON Geolocation.Country (Country);

--Create Geolocation.Province

	CREATE TABLE Geolocation.Province
	(
	ProvinceID		INT				NOT NULL IDENTITY,
	Province		NVARCHAR(25)	NOT NULL
		CONSTRAINT DFT_Province_Province DEFAULT ('ALBERTA'),
	CountryID		INT				NOT NULL,
	CONSTRAINT PK_Province PRIMARY KEY (ProvinceID),
	CONSTRAINT FK_Geolocation_Country FOREIGN KEY (CountryID)
		REFERENCES Geolocation.Country (CountryID)
	);
		CREATE NONCLUSTERED INDEX idx_nc_Province	ON Geolocation.Province (Province);
		CREATE NONCLUSTERED INDEX idx_nc_CountryID	ON Geolocation.Province (CountryID);

--Create Geolocation.City

	CREATE TABLE Geolocation.City
	(
	CityID		INT				NOT NULL IDENTITY,
	City		NVARCHAR(25)	NOT NULL
		CONSTRAINT DFT_City_City DEFAULT ('CALGARY'),
	ProvinceID	INT				NOT NULL,
	CONSTRAINT PK_City	PRIMARY KEY (CityID),
	CONSTRAINT FK_Geolocation_Province FOREIGN KEY (ProvinceID)
		REFERENCES Geolocation.Province (ProvinceID)
	);
	CREATE NONCLUSTERED INDEX idx_nc_City ON Geolocation.City (City);
	CREATE NONCLUSTERED INDEX idx_nc_ProvinceID ON Geolocation.City (ProvinceID);

--Create Sales.Customer

	CREATE TABLE Sales.Customer
	(
	Email			NVARCHAR(50)	NOT NULL,
	CustomerName	NVARCHAR(25)	NULL,
	Address			NVARCHAR(50)	NULL,
	PostalCode		NVARCHAR(50)	NULL,
	CityID			INT				NOT NULL,
	CONSTRAINT PK_Customer	PRIMARY KEY (Email),
	CONSTRAINT FK_Geolocation_City FOREIGN KEY (CityID)
		REFERENCES Geolocation.City (CityID),
	);
	CREATE UNIQUE NONCLUSTERED INDEX udx_nc_Email ON Sales.Customer (Email);
	CREATE NONCLUSTERED INDEX idx_nc_CityID ON Sales.Customer (CityID);
	CREATE NONCLUSTERED INDEX idx_nc_PostalCode ON Sales.Customer (PostalCode);

--Create Sales.Store

	CREATE TABLE Sales.Store
	(
	StoreID			INT				NOT NULL IDENTITY,
	StoreName		NVARCHAR(25)	NOT NULL,
	Address			NVARCHAR(25)	NOT NULL,
	CityID			INT				NOT NULL,
	CONSTRAINT PK_Store PRIMARY KEY (StoreID),
	CONSTRAINT FK_Geolocation_City2	FOREIGN KEY (CityID)
		REFERENCES Geolocation.City (CityID)
	);
	CREATE NONCLUSTERED INDEX idx_nc_StoreName ON Sales.Store (StoreName);
	CREATE NONCLUSTERED INDEX idx_nc_Address ON Sales.Store (Address);
	CREATE NONCLUSTERED INDEX idx_nc_CityID ON Sales.Store (CityID);


--Create Sales.POS

	CREATE TABLE Sales.POS
	(
	PointOfSaleID			INT			NOT NULL	IDENTITY,
	StoreID					INT			NOT NULL,
	SerialNumber			INT			NOT	NULL,
	CONSTRAINT PK_POS	PRIMARY KEY (PointOfsaleID),
	CONSTRAINT FK_Sales_Store FOREIGN KEY (StoreID)
		REFERENCES Sales.Store (StoreID)
	);
	CREATE UNIQUE NONCLUSTERED INDEX udx_nc_SerialNumber ON Sales.POS (SerialNumber);
	CREATE NONCLUSTERED INDEX idx_nc_StoreID ON Sales.Store(StoreID);

-- Create table HR.Employees

	CREATE TABLE HR.Employees
	(
	EmployeeID			INT					NOT NULL IDENTITY,
	LastName			NVARCHAR(20)		NOT NULL,
	FirstName			NVARCHAR(10)		NOT NULL,
	DesignationID		SMALLINT			NOT NULL,
	DepartmentID		SMALLINT			NOT NULL,
	BirthDate			DATE				NOT NULL,
	HireDate			DATE				NOT NULL,
	Email				NVARCHAR(50)		NOT NULL,
	Address				NVARCHAR(60)		NOT NULL,
	CityID				INT					NOT NULL,
	Postalcode			NVARCHAR(10)		NULL,
	Phone				NVARCHAR(24)		NOT NULL,
	SupervisorID		INT					 NULL,
	CONSTRAINT PK_Employees PRIMARY KEY(EmployeeID),
	CONSTRAINT FK_Employees_Employees FOREIGN KEY(SupervisorID)
		REFERENCES HR.Employees(EmployeeID),
	CONSTRAINT FK_Geolocation_City3	FOREIGN KEY (CityID)
		REFERENCES Geolocation.City (CityID),
	CONSTRAINT FK_HR_Designation	FOREIGN KEY (DesignationID)
		REFERENCES HR.Designation (DesignationID),
	CONSTRAINT FK_HR_Department	FOREIGN KEY (DepartmentID)
		REFERENCES HR.Department (DepartmentID),
	CONSTRAINT CHK_Birthdate CHECK(Birthdate <= CAST(SYSDATETIME() AS DATE)),
	CONSTRAINT CHK_Hiredate CHECK(HireDate <= CAST(SYSDATETIME() AS DATE))
 	 );
	  CREATE UNIQUE NONCLUSTERED INDEX udx_nc_Email ON HR.Employees (Email);
 	 CREATE NONCLUSTERED INDEX idx_nc_SupervisorID ON HR.Employees (SupervisorID);
 	 CREATE NONCLUSTERED INDEX idx_nc_CityID ON HR.Employees (CityID);
  	CREATE NONCLUSTERED INDEX idx_nc_DesignationID ON HR.Employees(DesignationID);
  	CREATE NONCLUSTERED INDEX idx_nc_DepartmentID ON HR.Employees (DepartmentID);
  	CREATE NONCLUSTERED INDEX idx_nc_LastName ON HR.Employees (LastName);

  --Create Product.PromotionType
  
	CREATE TABLE Product.PromotionType
	(
	PromoTypeID		INT				NOT NULL IDENTITY,
	PromoName		NVARCHAR(25)	NOT NULL,
	LaunchDate		DATE			NOT NULL,
	EndDate			DATE			NOT NULL,
	CONSTRAINT PK_PromotionType	PRIMARY KEY (PromoTypeID)	
	);
	CREATE NONCLUSTERED INDEX idx_nc_PromoName	ON Product.PromotionType (PromoName);

--Create Product.Promotion

	CREATE TABLE Product.Promotion
	(
	PromoID				INT NOT NULL IDENTITY,
	Discount			DECIMAL(4,2) NOT NULL,
	PromoTypeID			INT			NOT NULL,
	CONSTRAINT PK_Promotion PRIMARY KEY (PromoID),
	CONSTRAINT FK_Product_PromotionType FOREIGN KEY (PromoTypeID)
		REFERENCES Product.PromotionType (PromoTypeID)
	);
	CREATE NONCLUSTERED INDEX idx_nc_PromotypeID ON Product.Promotion(PromoTypeID);

 --Create Product.ProductCategory

	CREATE TABLE Product.ProductCategory
	(
	ProductCatID		INT				NOT NULL IDENTITY,
	CategoryName		NVARCHAR(50)	NOT NULL,
	PromoID				INT				NOT NULL,
	CONSTRAINT PK_ProductCategory	PRIMARY KEY (ProductCatID),
	CONSTRAINT FK_Product_Promotion	FOREIGN KEY (PromoID)
		REFERENCES Product.Promotion (PromoID)
	);
	CREATE NONCLUSTERED INDEX idc_nc_CategoryName	ON Product.ProductCategory(CategoryName);
	CREATE NONCLUSTERED INDEX idx_nc_PromoID		ON Product.ProductCategory (PromoID);

---Create Product.Product

	CREATE TABLE Product.Product
	(
	ProductID		INT			NOT NULL IDENTITY,
	ProductName		NVARCHAR(50)	NOT NULL,
	UnitPrice		MONEY		NOT NULL
		CONSTRAINT DFT_UnitPrice DEFAULT(0) ,
	SerialNumber	INT			NOT NULL,
	ProductCatID	INT			NOT NULL,
	CONSTRAINT PK_Product	PRIMARY KEY (ProductID),
	CONSTRAINT FK_Product_ProductCategory	FOREIGN KEY (ProductCatID)
		REFERENCES Product.ProductCategory (ProductCatID),
	CONSTRAINT CHK_Product_UnitPrice CHECK(UnitPrice >= 0)	
	);
	CREATE NONCLUSTERED INDEX idx_nc_ProductName ON Product.Product (ProductName);
	CREATE NONCLUSTERED INDEX idx_nc_ProductCatID ON Product.Product (ProductCatID);

--- Create Sales.Transactions

	CREATE TABLE Sales.Transactions
	(
	TransactionID		INT				NOT NULL IDENTITY,
	PointOfSaleID		INT				NOT NULL,
	CustomerID			NVARCHAR(50)	NOT NULL,
	SalesPersonID		INT				NOT NULL,
	ProductID			INT				NOT NULL,
	Quantity			INT				NOT NULL,
	SalesAmount			MONEY			NOT NULL
	CONSTRAINT DFT_SalesAmount DEFAULT(0),
	TransactionDate		DATE			NOT NULL,
	CONSTRAINT PK_Transactions PRIMARY KEY (TransactionID),
	CONSTRAINT FK_Sales_POS FOREIGN KEY (PointOfSaleID)
		REFERENCES Sales.POS (PointOfSaleID),
	CONSTRAINT FK_Sales_Customer FOREIGN KEY (CustomerID)
		REFERENCES Sales.Customer (Email),
	CONSTRAINT FK_HR_Employees FOREIGN KEY (SalesPersonID)
		REFERENCES HR.Employees (EmployeeID),
	CONSTRAINT FK_Product_Product FOREIGN KEY (ProductID)
		REFERENCES Product.Product (ProductID),
	CONSTRAINT CHK_Transaction_Quantity CHECK(Quantity >= 0)
	);
	CREATE NONCLUSTERED INDEX idc_nc_PointOfSaleID ON Sales.Transactions(PointOfSaleID);
	CREATE NONCLUSTERED INDEX idc_nc_CustomerID ON Sales.Transactions(CustomerID);
	CREATE NONCLUSTERED INDEX idc_nc_SalesPersonID ON Sales.Transactions(SalesPersonID);
	CREATE NONCLUSTERED INDEX idc_nc_TransactionDate ON Sales.Transactions(TransactionDate);
	CREATE NONCLUSTERED INDEX idc_nc_ProductID ON Sales.Transactions(ProductID);
