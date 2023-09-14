# DataBase
#SQL_Lodziarnia(projekt na DB, lodzirnia Goodlood)
'USE [master]
GO
/****** Object:  Database [Goodlood]    Script Date: 14.09.2023 15:32:58 ******/
CREATE DATABASE [Goodlood]
 CONTAINMENT = NONE
 ON  PRIMARY 
( NAME = N'Goodlood', FILENAME = N'C:\Program Files\Microsoft SQL Server\MSSQL11.MSSQLSERVER\MSSQL\DATA\Goodlood.mdf' , SIZE = 10240KB , MAXSIZE = UNLIMITED, FILEGROWTH = 1024KB )
 LOG ON 
( NAME = N'Goodlood_log', FILENAME = N'C:\Program Files\Microsoft SQL Server\MSSQL11.MSSQLSERVER\MSSQL\DATA\Goodlood_log.ldf' , SIZE = 10240KB , MAXSIZE = 2048GB , FILEGROWTH = 10%)
GO
ALTER DATABASE [Goodlood] SET COMPATIBILITY_LEVEL = 110
GO
IF (1 = FULLTEXTSERVICEPROPERTY('IsFullTextInstalled'))
begin
EXEC [Goodlood].[dbo].[sp_fulltext_database] @action = 'enable'
end
GO
ALTER DATABASE [Goodlood] SET ANSI_NULL_DEFAULT OFF 
GO
ALTER DATABASE [Goodlood] SET ANSI_NULLS OFF 
GO
ALTER DATABASE [Goodlood] SET ANSI_PADDING OFF 
GO
ALTER DATABASE [Goodlood] SET ANSI_WARNINGS OFF 
GO
ALTER DATABASE [Goodlood] SET ARITHABORT OFF 
GO
ALTER DATABASE [Goodlood] SET AUTO_CLOSE OFF 
GO
ALTER DATABASE [Goodlood] SET AUTO_CREATE_STATISTICS ON 
GO
ALTER DATABASE [Goodlood] SET AUTO_SHRINK OFF 
GO
ALTER DATABASE [Goodlood] SET AUTO_UPDATE_STATISTICS ON 
GO
ALTER DATABASE [Goodlood] SET CURSOR_CLOSE_ON_COMMIT OFF 
GO
ALTER DATABASE [Goodlood] SET CURSOR_DEFAULT  GLOBAL 
GO
ALTER DATABASE [Goodlood] SET CONCAT_NULL_YIELDS_NULL OFF 
GO
ALTER DATABASE [Goodlood] SET NUMERIC_ROUNDABORT OFF 
GO
ALTER DATABASE [Goodlood] SET QUOTED_IDENTIFIER OFF 
GO
ALTER DATABASE [Goodlood] SET RECURSIVE_TRIGGERS OFF 
GO
ALTER DATABASE [Goodlood] SET  DISABLE_BROKER 
GO
ALTER DATABASE [Goodlood] SET AUTO_UPDATE_STATISTICS_ASYNC OFF 
GO
ALTER DATABASE [Goodlood] SET DATE_CORRELATION_OPTIMIZATION OFF 
GO
ALTER DATABASE [Goodlood] SET TRUSTWORTHY OFF 
GO
ALTER DATABASE [Goodlood] SET ALLOW_SNAPSHOT_ISOLATION OFF 
GO
ALTER DATABASE [Goodlood] SET PARAMETERIZATION SIMPLE 
GO
ALTER DATABASE [Goodlood] SET READ_COMMITTED_SNAPSHOT OFF 
GO
ALTER DATABASE [Goodlood] SET HONOR_BROKER_PRIORITY OFF 
GO
ALTER DATABASE [Goodlood] SET RECOVERY SIMPLE 
GO
ALTER DATABASE [Goodlood] SET  MULTI_USER 
GO
ALTER DATABASE [Goodlood] SET PAGE_VERIFY CHECKSUM  
GO
ALTER DATABASE [Goodlood] SET DB_CHAINING OFF 
GO
ALTER DATABASE [Goodlood] SET FILESTREAM( NON_TRANSACTED_ACCESS = OFF ) 
GO
ALTER DATABASE [Goodlood] SET TARGET_RECOVERY_TIME = 0 SECONDS 
GO
USE [Goodlood]
GO
/****** Object:  StoredProcedure [dbo].[CalculateOrders]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[CalculateOrders] -- считвет цену всего заказа(даже еали несколько строк) и делит на два в случае если дискаунт = 1
	-- Add the parameters for the stored procedure here
	@OrderID int
AS
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;

    -- Insert statements for procedure here
	SELECT SUM(TEMP.ProductSUM)*Discounts.[Discount%]/100 FROM (SELECT Quantity*UnitPrice as ProductSUM, Orders.DiscountID  FROM Orders JOIN OrderDetails On 
	Orders.OrderID=OrderDetails.OrderID JOIN Products ON OrderDetails.ProductID=Products.ProductID Where Orders.OrderID=@OrderID) as TEMP LEFT JOIN Discounts ON Discounts.DiscountID=TEMP.DiscountID group by Discounts.[Discount%]

END

GO
/****** Object:  StoredProcedure [dbo].[ClearHistory]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[ClearHistory]
	-- Add the parameters for the stored procedure here
	@Date DATETIME -- удаление устории на определенное количестно лней (назад)
	
AS
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;

    -- Insert statements for procedure here
	DELETE FROM History WHERE date < GETDATE()-@Date
END

GO
/****** Object:  StoredProcedure [dbo].[CreateOrder]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[CreateOrder] 
	-- Add the parameters for the stored procedure here--заполнение одновременно ордера и ордердитеилс
	  @CustomerID int,
	  @ProductID int,
	  @Quantity int = 1, -- по умолчанию 
      @EmployeeID int = 1,
      @ShopID int = 1,
      @DiscountID int = NULL
AS
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;
	INSERT INTO Orders(CustomerID, EmployeeID, ShopID, DiscountID, OrderDate) VALUES (@CustomerID, @EmployeeID, @ShopID, @DiscountID, GETDATE())
	INSERT INTO OrderDetails (OrderID, ProductID, Quantity) VALUES (SCOPE_IDENTITY(), @ProductID, @Quantity) --айди орера это айди предыдущего инсерта
    -- Insert statements for procedure here
	
END
GO
/****** Object:  StoredProcedure [dbo].[MostExpensiveProducts]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE procedure [dbo].[MostExpensiveProducts] AS -- самые дорогие 4 продукта  exec [dbo].[MostExpensiveProducts]
SET ROWCOUNT 4
SELECT Products.ProductName AS TenMostExpensiveProducts, Products.UnitPrice
FROM Products
ORDER BY Products.UnitPrice DESC

GO
/****** Object:  StoredProcedure [dbo].[OnlyCategoryPrice]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[OnlyCategoryPrice] -- exec OnlyCategoryPrice 1-- цены только на мороженое или на добавки 
	-- Add the parameters for the stored procedure here
	@CategoryID int
AS
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;

    -- Insert statements for procedure here
	Select ProductName, UnitPrice from Products join ProductCategories ON
 [Goodlood].[dbo].[Products].[CategoryID] = [Goodlood].[dbo].[ProductCategories].[CategoryID] 
 where [Goodlood].[dbo].[ProductCategories].[CategoryID] = @CategoryID
END

GO
/****** Object:  UserDefinedFunction [dbo].[CUBE]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date, ,>
-- Description:	<Description, ,>
-- =============================================
CREATE FUNCTION [dbo].[CUBE] 
(
	-- Add the parameters for the function here
	@num int
)
RETURNS int
AS
BEGIN
	-- Declare the return variable here
	DECLARE @result int

	-- Add the T-SQL statements to compute the return value here
	SELECT @result = @num*@num*@num

	-- Return the result of the function
	RETURN @result

END

GO
/****** Object:  UserDefinedFunction [dbo].[LN]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date, ,>
-- Description:	<Description, ,>
-- =============================================
CREATE FUNCTION [dbo].[LN] -- натуральный логарифм числа  Select dbo.LN(8)
(
	-- Add the parameters for the function here
	@num float
)
RETURNS float
AS
BEGIN
	-- Declare the return variable here
	DECLARE @result float

	-- Add the T-SQL statements to compute the return value here
	SELECT @result = LOG(@num,2.72)

	-- Return the result of the function
	RETURN @result

END

GO
/****** Object:  Table [dbo].[Customers]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Customers](
	[CustomerID] [int] IDENTITY(1,1) NOT NULL,
	[CustomerName] [nvarchar](50) NULL,
	[PhoneNumber] [nvarchar](20) NULL,
 CONSTRAINT [PK_Customers] PRIMARY KEY CLUSTERED 
(
	[CustomerID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO
/****** Object:  Table [dbo].[Discounts]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Discounts](
	[DiscountID] [int] IDENTITY(1,1) NOT NULL,
	[Discount%] [decimal](3, 0) NULL,
	[Description] [nvarchar](50) NULL,
 CONSTRAINT [PK_Discounts] PRIMARY KEY CLUSTERED 
(
	[DiscountID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO
/****** Object:  Table [dbo].[Employees]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Employees](
	[EmployeeID] [int] IDENTITY(1,1) NOT NULL,
	[LastName] [nvarchar](30) NOT NULL,
	[FirstName] [nvarchar](20) NOT NULL,
	[Title] [nvarchar](30) NULL,
	[HireDate] [datetime] NULL,
	[ShopID] [int] NULL,
 CONSTRAINT [PK_Employees] PRIMARY KEY CLUSTERED 
(
	[EmployeeID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO
/****** Object:  Table [dbo].[History]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
SET ANSI_PADDING ON
GO
CREATE TABLE [dbo].[History](
	[id] [int] IDENTITY(1,1) NOT NULL,
	[log] [varchar](255) NULL,
	[date] [datetime] NULL,
 CONSTRAINT [PK_History] PRIMARY KEY CLUSTERED 
(
	[id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO
SET ANSI_PADDING OFF
GO
/****** Object:  Table [dbo].[OrderDetails]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[OrderDetails](
	[OrderID] [int] NOT NULL,
	[ProductID] [int] NOT NULL,
	[Quantity] [int] NULL
) ON [PRIMARY]

GO
/****** Object:  Table [dbo].[Orders]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Orders](
	[OrderID] [int] IDENTITY(1,1) NOT NULL,
	[CustomerID] [int] NULL,
	[EmployeeID] [int] NULL,
	[OrderDate] [datetime] NULL,
	[ShopID] [int] NULL,
	[DiscountID] [int] NULL,
 CONSTRAINT [PK_Orders_1] PRIMARY KEY CLUSTERED 
(
	[OrderID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO
/****** Object:  Table [dbo].[ProductCategories]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
SET ANSI_PADDING ON
GO
CREATE TABLE [dbo].[ProductCategories](
	[CategoryID] [int] IDENTITY(1,1) NOT NULL,
	[CategoryName] [varchar](30) NULL,
 CONSTRAINT [PK_Categories] PRIMARY KEY CLUSTERED 
(
	[CategoryID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO
SET ANSI_PADDING OFF
GO
/****** Object:  Table [dbo].[Products]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
SET ANSI_PADDING ON
GO
CREATE TABLE [dbo].[Products](
	[ProductID] [int] IDENTITY(1,1) NOT NULL,
	[ProductName] [varchar](30) NULL,
	[UnitPrice] [money] NULL,
	[CategoryID] [int] NULL,
 CONSTRAINT [PK_Products] PRIMARY KEY CLUSTERED 
(
	[ProductID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO
SET ANSI_PADDING OFF
GO
/****** Object:  Table [dbo].[ProductsHistory]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
SET ANSI_PADDING ON
GO
CREATE TABLE [dbo].[ProductsHistory](
	[HisoryID] [int] IDENTITY(1,1) NOT NULL,
	[ProductID] [varchar](255) NULL,
	[Operation] [nchar](255) NULL,
	[date] [datetime] NULL,
 CONSTRAINT [PK_ProductsHistory] PRIMARY KEY CLUSTERED 
(
	[HisoryID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO
SET ANSI_PADDING OFF
GO
/****** Object:  Table [dbo].[Reviews]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Reviews](
	[ReviewID] [int] IDENTITY(1,1) NOT NULL,
	[CustomerID] [int] NULL,
	[ProductID] [int] NULL,
	[MarkFrom_0_to_10] [int] NULL,
 CONSTRAINT [PK_Reviews] PRIMARY KEY CLUSTERED 
(
	[ReviewID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO
/****** Object:  Table [dbo].[Stores]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Stores](
	[ShopID] [int] IDENTITY(1,1) NOT NULL,
	[Address] [nvarchar](30) NULL,
	[City] [nvarchar](20) NULL,
 CONSTRAINT [PK_ChainStores] PRIMARY KEY CLUSTERED 
(
	[ShopID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO
/****** Object:  UserDefinedFunction [dbo].[CustomerOrders]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE FUNCTION [dbo].[CustomerOrders]-- заказы оформленные конкретным employee
(	
	-- Add the parameters for the function here
  @EmployeeID int 
)
RETURNS TABLE 
AS
RETURN 
(
	-- Add the SELECT statement with parameter references here
	SELECT OrderID from Orders where CustomerID = @EmployeeID
)

GO
/****** Object:  UserDefinedFunction [dbo].[EmployeeTitle]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE FUNCTION [dbo].[EmployeeTitle]
(	
	-- Add the parameters for the function here
	@Title nvarchar(30)
)
RETURNS TABLE 
AS
RETURN 
(
	-- Add the SELECT statement with parameter references here
	SELECT EmployeeID,  Title from Employees where Title = @Title
)

GO
/****** Object:  UserDefinedFunction [dbo].[GetCustomerPhone]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE FUNCTION [dbo].[GetCustomerPhone] --  НОМЕРА телефонов клиентов(айди клиента)
(	
	-- Add the parameters for the function here
	@CustomerID int
)
RETURNS TABLE 
AS
RETURN 
(
	-- Add the SELECT statement with parameter references here
	SELECT phonenumber, CustomerName from Customers WHERE CustomerID=@CustomerID
)

GO
/****** Object:  UserDefinedFunction [dbo].[GetCustomersPhones]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE FUNCTION [dbo].[GetCustomersPhones] -- все известные номера клиентов (клиенты которые оставляли номер телефона)
(	

)
RETURNS TABLE 
AS
RETURN 
(
	-- Add the SELECT statement with parameter references here
	SELECT phonenumber, CustomerName from Customers WHERE phonenumber IS NOT NULL
)
GO
/****** Object:  UserDefinedFunction [dbo].[GetEmployersFromStore]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE FUNCTION [dbo].[GetEmployersFromStore] 
(	
	-- Add the parameters for the function here
	@ShopID int
	
)
RETURNS TABLE 
AS
RETURN 
(
	-- Add the SELECT statement with parameter references here
	SELECT Employees.EmployeeID,ShopID from Employees WHERE Employees.ShopID = @ShopID
)

GO
/****** Object:  UserDefinedFunction [dbo].[OrderDate]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE FUNCTION [dbo].[OrderDate] -- даты заказов котнкретного клиента
(	
 @OrderID int
)
RETURNS TABLE 
AS
RETURN 
(
	-- Add the SELECT statement with parameter references here
	SELECT OrderDate, OrderID from Orders where CustomerID = @OrderID
)
GO
/****** Object:  UserDefinedFunction [dbo].[ProductsWithMark]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE FUNCTION [dbo].[ProductsWithMark] -- продукты отзывы у которых больше или равняются ...
(	
	@Mark int
)
RETURNS TABLE 
AS
RETURN 
(
	-- Add the SELECT statement with parameter references here
	SELECT ProductName, MarkFrom_0_to_10 From dbo.Reviews Join dbo.Products
	 ON [Goodlood].[dbo].[Reviews].[ProductID] = [Goodlood].[dbo].[Products].[ProductID]
    Where  MarkFrom_0_to_10 >= @Mark
)

GO
/****** Object:  UserDefinedFunction [dbo].[ShopinCity]    Script Date: 14.09.2023 15:32:58 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
create FUNCTION [dbo].[ShopinCity]
(	
	-- Add the parameters for the function here
	@City nvarchar(20)
)
RETURNS TABLE 
AS
RETURN 
(
	-- Add the SELECT statement with parameter references here
	SELECT ShopId from Stores where city = @City
)
GO
SET IDENTITY_INSERT [dbo].[Customers] ON 

INSERT [dbo].[Customers] ([CustomerID], [CustomerName], [PhoneNumber]) VALUES (1, N'Amelia', N'+48739850209')
INSERT [dbo].[Customers] ([CustomerID], [CustomerName], [PhoneNumber]) VALUES (2, N'Robert', N'+48309789414')
INSERT [dbo].[Customers] ([CustomerID], [CustomerName], [PhoneNumber]) VALUES (3, N'Aleksandra', NULL)
INSERT [dbo].[Customers] ([CustomerID], [CustomerName], [PhoneNumber]) VALUES (4, N'Leon', N'+48862789414')
INSERT [dbo].[Customers] ([CustomerID], [CustomerName], [PhoneNumber]) VALUES (5, N'Amelia', NULL)
INSERT [dbo].[Customers] ([CustomerID], [CustomerName], [PhoneNumber]) VALUES (6, N'Bartosz', N'+4868493073')
INSERT [dbo].[Customers] ([CustomerID], [CustomerName], [PhoneNumber]) VALUES (7, N'Elizabet', NULL)
INSERT [dbo].[Customers] ([CustomerID], [CustomerName], [PhoneNumber]) VALUES (8, N'Krzystof', NULL)
INSERT [dbo].[Customers] ([CustomerID], [CustomerName], [PhoneNumber]) VALUES (9, N'Jogan', NULL)
SET IDENTITY_INSERT [dbo].[Customers] OFF
SET IDENTITY_INSERT [dbo].[Discounts] ON 

INSERT [dbo].[Discounts] ([DiscountID], [Discount%], [Description]) VALUES (1, CAST(50 AS Decimal(3, 0)), N'For a first purchase')
SET IDENTITY_INSERT [dbo].[Discounts] OFF
SET IDENTITY_INSERT [dbo].[Employees] ON 

INSERT [dbo].[Employees] ([EmployeeID], [LastName], [FirstName], [Title], [HireDate], [ShopID]) VALUES (1, N'Zielinska', N'Jadwiga', N'Sales Representative', CAST(N'2020-08-01 00:00:00.000' AS DateTime), 1)
INSERT [dbo].[Employees] ([EmployeeID], [LastName], [FirstName], [Title], [HireDate], [ShopID]) VALUES (2, N'Wojcik', N'Kazimierz', N'Sales Representative', CAST(N'2020-10-01 00:00:00.000' AS DateTime), 1)
INSERT [dbo].[Employees] ([EmployeeID], [LastName], [FirstName], [Title], [HireDate], [ShopID]) VALUES (3, N'Kowalczyk', N'Ignacy', N'Sales Representative', CAST(N'2020-11-01 00:00:00.000' AS DateTime), 2)
INSERT [dbo].[Employees] ([EmployeeID], [LastName], [FirstName], [Title], [HireDate], [ShopID]) VALUES (4, N'Wielczyk', N'Anna', N'Sales Manager', CAST(N'2020-11-28 00:00:00.000' AS DateTime), 2)
INSERT [dbo].[Employees] ([EmployeeID], [LastName], [FirstName], [Title], [HireDate], [ShopID]) VALUES (5, N'Chodak', N'Tadeusz', N'Sales Representative', CAST(N'2021-01-04 00:00:00.000' AS DateTime), 2)
INSERT [dbo].[Employees] ([EmployeeID], [LastName], [FirstName], [Title], [HireDate], [ShopID]) VALUES (6, N'Kowalska', N'Paulina', N'Sales Representative', CAST(N'2021-02-25 00:00:00.000' AS DateTime), 3)
SET IDENTITY_INSERT [dbo].[Employees] OFF
SET IDENTITY_INSERT [dbo].[History] ON 

INSERT [dbo].[History] ([id], [log], [date]) VALUES (20, N'Added order:53                             for Customer:Amelia                        ', CAST(N'2022-02-14 23:33:59.923' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (21, N'Added order:54                             for Customer:6                             ', CAST(N'2022-02-15 13:38:40.490' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (22, N'Added order:55                             for Customer:7                             ', CAST(N'2022-02-15 14:41:26.920' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (23, N'Added order:56                             for Customer:8                             ', CAST(N'2022-02-15 14:41:52.477' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (24, N'Added order:58                             for Customer:8                             ', CAST(N'2022-02-15 16:30:29.850' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (25, N'Added order:59                             for Customer:8                             ', CAST(N'2022-02-15 16:35:52.350' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (27, N'Added order:62                             for Customer:2                             ', CAST(N'2022-02-15 21:29:10.450' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (28, N'Added order:63                             for Customer:2                             ', CAST(N'2022-02-15 21:32:21.790' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (29, N'Added order:64                             for Customer:2                             ', CAST(N'2022-02-15 22:10:33.073' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (30, N'Added order:65                             for Customer:2                             ', CAST(N'2022-02-15 22:11:05.130' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (31, N'Added order:66                             for Customer:2                             ', CAST(N'2022-02-15 22:13:38.963' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (32, N'Added order:68                             for Customer:2                             ', CAST(N'2022-02-15 22:16:11.213' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (33, N'Added order:69                             for Customer:4                             ', CAST(N'2022-02-15 22:22:44.490' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (34, N'Added order:70                             for Customer:1                             ', CAST(N'2022-02-15 22:28:32.213' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (35, N'Added order:71                             for Customer:1                             ', CAST(N'2022-02-15 22:59:02.390' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (36, N'Added order:72                             for Customer:3                             ', CAST(N'2022-02-16 11:43:06.090' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (37, N'Added order:73                             for Customer:9                             ', CAST(N'2022-02-16 15:17:21.880' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (38, N'Added order:74                             for Customer:9                             ', CAST(N'2022-02-16 16:04:15.863' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (39, N'Added order:75                             for Customer:1                             ', CAST(N'2022-02-16 16:06:19.300' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (40, N'Added order:76                             for Customer:1                             ', CAST(N'2022-02-16 16:08:34.970' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (41, N'Added order:77                             for Customer:1                             ', CAST(N'2022-02-16 16:12:17.650' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (42, N'Added order:78                             for Customer:1                             ', CAST(N'2022-02-16 16:13:17.957' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (43, N'Added order:79                             for Customer:1                             ', CAST(N'2022-02-16 16:15:23.640' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (44, N'Added order:80                             for Customer:1                             ', CAST(N'2022-02-16 16:23:42.370' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (45, N'Added order:81                             for Customer:1                             ', CAST(N'2022-02-16 16:25:05.103' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (46, N'Added order:82                             for Customer:1                             ', CAST(N'2022-02-16 16:27:59.500' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (47, N'Added order:83                             for Customer:2                             ', CAST(N'2022-02-16 16:30:16.617' AS DateTime))
INSERT [dbo].[History] ([id], [log], [date]) VALUES (48, N'Added order:84                             for Customer:6                             ', CAST(N'2022-02-16 16:32:34.477' AS DateTime))
SET IDENTITY_INSERT [dbo].[History] OFF
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (62, 2, 10)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (62, 6, 5)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (62, 6, 5)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (66, 2, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (68, 2, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (74, 1, 2)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (75, 3, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (76, 3, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (77, 3, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (78, 4, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (79, 4, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (75, 4, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (75, 4, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (75, 1, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (75, 1, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (75, 1, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (82, 4, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (83, 3, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (84, 3, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (69, 3, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (70, 6, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (71, 6, 1)
INSERT [dbo].[OrderDetails] ([OrderID], [ProductID], [Quantity]) VALUES (73, 1, 4)
SET IDENTITY_INSERT [dbo].[Orders] ON 

INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (45, 1, 1, CAST(N'2022-02-10 00:00:00.000' AS DateTime), 1, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (46, 1, 1, CAST(N'2022-02-13 17:13:10.413' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (48, 2, 3, CAST(N'2022-02-14 16:58:36.033' AS DateTime), 2, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (49, 3, 2, CAST(N'2022-02-14 17:52:25.893' AS DateTime), 1, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (50, 3, 2, CAST(N'2022-02-14 17:54:07.550' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (51, 2, 4, CAST(N'2022-02-14 18:36:14.390' AS DateTime), 2, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (52, 4, 1, CAST(N'2022-02-14 18:37:44.913' AS DateTime), 1, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (53, 5, 6, CAST(N'2022-02-14 23:33:59.900' AS DateTime), 3, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (54, 6, 2, CAST(N'2022-02-15 13:38:40.480' AS DateTime), 1, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (55, 7, 2, CAST(N'2022-02-15 14:41:26.910' AS DateTime), 1, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (56, 8, 3, CAST(N'2022-02-15 14:41:52.477' AS DateTime), 2, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (58, 8, 2, CAST(N'2022-02-15 16:30:29.840' AS DateTime), 2, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (59, 8, 2, CAST(N'2022-02-15 16:35:52.330' AS DateTime), 2, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (62, 2, 6, CAST(N'2022-02-15 21:29:10.443' AS DateTime), 1, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (63, 2, 6, CAST(N'2022-02-15 21:32:21.783' AS DateTime), 1, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (64, 2, 1, NULL, 1, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (65, 2, 1, NULL, 1, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (66, 2, 1, NULL, 1, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (68, 2, 1, NULL, 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (69, 4, 1, NULL, 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (70, 1, 1, NULL, 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (71, 1, 1, CAST(N'2022-02-15 22:59:02.383' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (72, 3, 6, CAST(N'2022-02-16 11:43:06.077' AS DateTime), 3, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (73, 9, 4, CAST(N'2022-02-16 15:17:21.867' AS DateTime), 2, 1)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (74, 9, 1, CAST(N'2022-02-16 16:04:15.840' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (75, 1, 1, CAST(N'2022-02-16 16:06:19.287' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (76, 1, 1, CAST(N'2022-02-16 16:08:34.953' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (77, 1, 1, CAST(N'2022-02-16 16:12:17.630' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (78, 1, 1, CAST(N'2022-02-16 16:13:17.940' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (79, 1, 1, CAST(N'2022-02-16 16:15:23.590' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (80, 1, 1, CAST(N'2022-02-16 16:23:42.310' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (81, 1, 1, NULL, 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (82, 1, 1, CAST(N'2022-02-16 16:27:59.490' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (83, 2, 1, CAST(N'2022-02-16 16:30:16.603' AS DateTime), 1, NULL)
INSERT [dbo].[Orders] ([OrderID], [CustomerID], [EmployeeID], [OrderDate], [ShopID], [DiscountID]) VALUES (84, 6, 1, CAST(N'2022-02-16 16:32:34.457' AS DateTime), 1, NULL)
SET IDENTITY_INSERT [dbo].[Orders] OFF
SET IDENTITY_INSERT [dbo].[ProductCategories] ON 

INSERT [dbo].[ProductCategories] ([CategoryID], [CategoryName]) VALUES (1, N'Ice_cream')
INSERT [dbo].[ProductCategories] ([CategoryID], [CategoryName]) VALUES (2, N'Supplements')
SET IDENTITY_INSERT [dbo].[ProductCategories] OFF
SET IDENTITY_INSERT [dbo].[Products] ON 

INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (1, N'Chocolate_lood', 5.0000, 1)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (2, N'Milk_lood', 5.0000, 1)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (3, N'Vanilla_lood', 6.0000, 1)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (4, N'Strawberry_lood', 6.0000, 1)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (5, N'Banana_lood', 6.0000, 1)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (6, N'Blueberries_lood', 6.0000, 1)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (7, N'Fistachio_lood', 6.0000, 1)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (8, N'Chocolate_syrup', 2.0000, 2)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (9, N'Сaramel_syrup', 2.0000, 2)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (10, N'Vanilla_syrup', 2.0000, 2)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (11, N'Hazelnut_syrup', 2.0000, 2)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (12, N'Lavender_syrup', 2.0000, 2)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (36, N'Сaramel_сream_lood', 32.0000, 1)
INSERT [dbo].[Products] ([ProductID], [ProductName], [UnitPrice], [CategoryID]) VALUES (37, N'Pineapple_lood', 10.0000, 1)
SET IDENTITY_INSERT [dbo].[Products] OFF
SET IDENTITY_INSERT [dbo].[ProductsHistory] ON 

INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (1, N'Added ProductID:20', N'ProductName:Caramel_cream_lood                                                                                                                                                                                                                                 ', CAST(N'2022-02-16 11:51:34.177' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (2, N'Added ProductID:22', N'ProductName:Caramel_cream_lood                                                                                                                                                                                                                                 ', CAST(N'2022-02-16 12:05:28.277' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (3, N'Deleted ProductID:23', N'ProductName:Caramel_cream_lood                                                                                                                                                                                                                                 ', CAST(N'2022-02-16 12:05:28.277' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (4, N'Deleted ProductID:22', N'ProductName:Caramel_cream_lood                                                                                                                                                                                                                                 ', CAST(N'2022-02-16 12:05:28.277' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (5, NULL, NULL, CAST(N'2022-02-16 12:06:30.813' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (6, NULL, NULL, CAST(N'2022-02-16 12:06:42.590' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (7, N'Added ProductID:24', N'ProductName:Car                                                                                                                                                                                                                                                ', CAST(N'2022-02-16 12:07:49.760' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (8, NULL, NULL, CAST(N'2022-02-16 12:08:31.873' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (9, N'Added ProductID:26', N'ProductName:Cbtbjffff                                                                                                                                                                                                                                          ', CAST(N'2022-02-16 12:11:17.097' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (10, N'Added ProductID:28', N'ProductName:oppdkffff                                                                                                                                                                                                                                          ', CAST(N'2022-02-16 12:12:50.083' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (11, N'Added ProductID:30', N'ProductName:op321ff                                                                                                                                                                                                                                            ', CAST(N'2022-02-16 12:14:13.770' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (12, N'Added ProductID:32', N'ProductName:op321ff                                                                                                                                                                                                                                            ', CAST(N'2022-02-16 12:16:29.300' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (13, N'Added ProductID:34', N'ProductName:Caramel_cream_lood                                                                                                                                                                                                                                 ', CAST(N'2022-02-16 12:17:36.783' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (14, NULL, NULL, CAST(N'2022-02-16 12:18:36.540' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (15, N'Added ProductID:35', N'ProductName:op321ff                                                                                                                                                                                                                                            ', CAST(N'2022-02-16 12:18:59.940' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (16, N'Deleted ProductID:35', N'ProductName:op321ff1                                                                                                                                                                                                                                           ', CAST(N'2022-02-16 12:24:11.007' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (17, N'Added ProductID:36', N'ProductName:Сaramel_сream_lood1                                                                                                                                                                                                                                ', CAST(N'2022-02-16 13:08:13.943' AS DateTime))
INSERT [dbo].[ProductsHistory] ([HisoryID], [ProductID], [Operation], [date]) VALUES (18, N'Added ProductID:37', N'ProductName:Pineapple_lood1                                                                                                                                                                                                                                    ', CAST(N'2022-02-16 13:10:24.020' AS DateTime))
SET IDENTITY_INSERT [dbo].[ProductsHistory] OFF
SET IDENTITY_INSERT [dbo].[Reviews] ON 

INSERT [dbo].[Reviews] ([ReviewID], [CustomerID], [ProductID], [MarkFrom_0_to_10]) VALUES (1, 1, 4, 10)
INSERT [dbo].[Reviews] ([ReviewID], [CustomerID], [ProductID], [MarkFrom_0_to_10]) VALUES (2, 2, 4, 6)
INSERT [dbo].[Reviews] ([ReviewID], [CustomerID], [ProductID], [MarkFrom_0_to_10]) VALUES (3, 2, 5, 9)
SET IDENTITY_INSERT [dbo].[Reviews] OFF
SET IDENTITY_INSERT [dbo].[Stores] ON 

INSERT [dbo].[Stores] ([ShopID], [Address], [City]) VALUES (1, N'Dygasinskiego 21', N'Krakow')
INSERT [dbo].[Stores] ([ShopID], [Address], [City]) VALUES (2, N'Kasztanowa 9', N'Krakow')
INSERT [dbo].[Stores] ([ShopID], [Address], [City]) VALUES (3, N'Wladyslawa Andersa 2', N'Krakow')
SET IDENTITY_INSERT [dbo].[Stores] OFF
/****** Object:  Index [UnitPriceIndex]    Script Date: 14.09.2023 15:32:58 ******/
CREATE NONCLUSTERED INDEX [UnitPriceIndex] ON [dbo].[Products]
(
	[UnitPrice] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
GO
ALTER TABLE [dbo].[Employees]  WITH CHECK ADD  CONSTRAINT [FK_ShopID] FOREIGN KEY([ShopID])
REFERENCES [dbo].[Stores] ([ShopID])
GO
ALTER TABLE [dbo].[Employees] CHECK CONSTRAINT [FK_ShopID]
GO
ALTER TABLE [dbo].[OrderDetails]  WITH CHECK ADD  CONSTRAINT [FK_OrderID_Orders] FOREIGN KEY([OrderID])
REFERENCES [dbo].[Orders] ([OrderID])
GO
ALTER TABLE [dbo].[OrderDetails] CHECK CONSTRAINT [FK_OrderID_Orders]
GO
ALTER TABLE [dbo].[OrderDetails]  WITH CHECK ADD  CONSTRAINT [FK_ProductID_Products] FOREIGN KEY([ProductID])
REFERENCES [dbo].[Products] ([ProductID])
GO
ALTER TABLE [dbo].[OrderDetails] CHECK CONSTRAINT [FK_ProductID_Products]
GO
ALTER TABLE [dbo].[Orders]  WITH CHECK ADD  CONSTRAINT [FK_CustomerID] FOREIGN KEY([CustomerID])
REFERENCES [dbo].[Customers] ([CustomerID])
ON UPDATE SET DEFAULT
GO
ALTER TABLE [dbo].[Orders] CHECK CONSTRAINT [FK_CustomerID]
GO
ALTER TABLE [dbo].[Orders]  WITH CHECK ADD  CONSTRAINT [FK_DiscountID] FOREIGN KEY([DiscountID])
REFERENCES [dbo].[Discounts] ([DiscountID])
GO
ALTER TABLE [dbo].[Orders] CHECK CONSTRAINT [FK_DiscountID]
GO
ALTER TABLE [dbo].[Orders]  WITH CHECK ADD  CONSTRAINT [FK_EmployeeID] FOREIGN KEY([EmployeeID])
REFERENCES [dbo].[Employees] ([EmployeeID])
GO
ALTER TABLE [dbo].[Orders] CHECK CONSTRAINT [FK_EmployeeID]
GO
ALTER TABLE [dbo].[Products]  WITH CHECK ADD  CONSTRAINT [FK_Category] FOREIGN KEY([CategoryID])
REFERENCES [dbo].[ProductCategories] ([CategoryID])
ON DELETE SET DEFAULT
GO
ALTER TABLE [dbo].[Products] CHECK CONSTRAINT [FK_Category]
GO
ALTER TABLE [dbo].[Reviews]  WITH CHECK ADD  CONSTRAINT [FK_Product_Products] FOREIGN KEY([ProductID])
REFERENCES [dbo].[Products] ([ProductID])
GO
ALTER TABLE [dbo].[Reviews] CHECK CONSTRAINT [FK_Product_Products]
GO
ALTER TABLE [dbo].[Reviews]  WITH CHECK ADD  CONSTRAINT [FK_Reviews_Customers] FOREIGN KEY([CustomerID])
REFERENCES [dbo].[Customers] ([CustomerID])
GO
ALTER TABLE [dbo].[Reviews] CHECK CONSTRAINT [FK_Reviews_Customers]
GO
USE [master]
GO
ALTER DATABASE [Goodlood] SET  READ_WRITE 
GO'
