# DataCleaning_SQL
Data cleaning using SQL.

## Objective
Perform a cleaning process on a data set using the SQL Server tool. Carrying out different tasks such as data visualizations, value corrections, table updates, elimination of unnecessary spaces, etc.

This is a project that is part of the playlist of projects for the development of a portfolio of Alex The Analyst. 

In the archives you will find:

1. [Nashville Housing Data for Data Cleaning.xlsx](https://github.com/FelipePuyolBesa/DataCleaning_SQL/blob/main/Nashville%20Housing%20Data%20for%20Data%20Cleaning.xlsx)--> Excel file with the data to clean.
2. [Data Cleaning.sql](https://github.com/FelipePuyolBesa/DataCleaning_SQL/blob/main/Data%20Cleaning.sql)--> File containing the executed queries.
3. [Sheet SQL Data Preparation.jpg](https://github.com/FelipePuyolBesa/DataCleaning_SQL/blob/main/Sheet%20SQL%20Data%20Preparation.jpg)--> Image containing some of the most used queries for preparing data in SQL.



Link to Alex's playlist. https://www.youtube.com/watch?v=8rO7ztF4NtU&list=PLUaB-1hjhk8H48Pj32z4GZgGWyylqv85f&index=3&ab_channel=AlexTheAnalyst


## Query for first data visualization

	SELECT TOP 1000 *
	FROM Nashville..Houses;
	
!Casas1


## Date standardization (SaleDate)
   Removal of hour values ​​as it only contains date information

  ### Data visualization SaleDate

	  SELECT SaleDate
	  FROM Nashville..Houses;
!Casas2

  ### Display of modification of the data type to Date

	  SELECT SaleDate , CONVERT(date, SaleDate)
	  FROM Nashville..Houses;

  ### Updating the SaleDate field to date format
  
	  UPDATE Nashville..Houses
	  SET SaleDate = CONVERT(date, SaleDate);

  ### In the case that does not allow updating, you can create a new column and delete the other

	  ALTER TABLE Nashville..Houses
	  ADD SaleDateConverted Date;

	  UPDATE Nashville..Houses
	  SET SaleDateConverted = CONVERT(date, SaleDate);

	  ALTER TABLE Nashville..Houses
	  DROP COLUMN SaleDate;
!Casas3


  ### Checking and counting null values ​​for SaleDate

	  SELECT SaleDateConverted
	  FROM Nashville..Houses
	  WHERE SaleDateConverted IS NULL;

##  Data transformation for the PropertyAddress field
### Data display for the PropertyAddress field

	SELECT PropertyAddress
	FROM Nashville..Houses;

### Checking and counting null values ​​for SaleDate

	SELECT PropertyAddress
	FROM Nashville..Houses
	WHERE PropertyAddress IS NULL;

	SELECT COUNT(*)
	FROM Nashville..Houses
	WHERE PropertyAddress IS NULL;

### Checking for null values ​​in PropertyAddress
	SELECT *
	FROM Nashville..Houses
	WHERE PropertyAddress IS NULL;
	
!Casas4

!Casas5


## Query records with more than one ParcelID
	SELECT COUNT(PropertyAddress), ParcelID, PropertyAddress
	FROM Nashville..Houses
	GROUP BY ParcelID, PropertyAddress
	HAVING COUNT (PropertyAddress) > 1
	ORDER BY ParcelID
	
!Casas6


### ParcelID and PropertyAddress query from the above query
	SELECT ParcelID, PropertyAddress
	FROM Nashville..Houses
	WHERE ParcelID IN 
		(SELECT ParcelID
		FROM Nashville..Houses
		GROUP BY ParcelID, PropertyAddress
		HAVING COUNT (PropertyAddress) > 1)
	ORDER BY ParcelID

### ParcelID and PropertyAddress query for null assignment, join to the same table

	SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
	FROM Nashville..Houses AS a
	JOIN Nashville..Houses AS b
		ON a.ParcelID = b.ParcelID
		AND a.[UniqueID ]<> b.[UniqueID ]
	WHERE a.PropertyAddress IS NULL;
	
!Casas7


### Updating ParcelID addresses with null addresses

	UPDATE a
	SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
	FROM Nashville..Houses AS a
	JOIN Nashville..Houses AS b
		ON a.ParcelID = b.ParcelID
		AND a.[UniqueID ]<> b.[UniqueID ]
	WHERE a.PropertyAddress IS NULL;
	
!Casas8

!Casas9
 


## Columnarization of the data in PropertyAddress

### Consultation of the data in the addresses

		SELECT DISTINCT(PropertyAddress)
		FROM Nashville..Houses;
		
!Casas10


### Query for address division, value before comma

		SELECT
		SUBSTRING (PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1) as Direccion
		FROM Nashville..Houses;
		
!Casas11


### Query for city division, value after comma
		SELECT
		SUBSTRING (PropertyAddress, CHARINDEX(',', PropertyAddress) + 2, LEN(PropertyAddress)) as Ciudad
		FROM Nashville..Houses;

### Add the column and values ​​for address

		ALTER TABLE Nashville..Houses
		ADD PropertyDireccion NVARCHAR(255)

		UPDATE Nashville..Houses
		SET PropertyDireccion = (SUBSTRING (PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1));

!Casas12

### Add the column and values ​​for city

		ALTER TABLE Nashville..Houses
		ADD PropertyCiudad NVARCHAR(255)

		UPDATE Nashville..Houses
		SET PropertyCiudad = SUBSTRING (PropertyAddress, CHARINDEX(',', PropertyAddress) + 2, LEN(PropertyAddress));


		SELECT TOP 100 *
		FROM Nashville..Houses;



## Columnarization of the data in OwnerAddress

### Consultation of the data in the addresses

		SELECT DISTINCT(OwnerAddress)
		FROM Nashville..Houses;
		
!Casas13


### Query for the division of values ​​in OwnerAddress

		SELECT  PARSENAME(REPLACE(OwnerAddress, ',', '.'),3),
				PARSENAME(REPLACE(OwnerAddress, ',', '.'),2),
				PARSENAME(REPLACE(OwnerAddress, ',', '.'),1)
		FROM Nashville..Houses;
		
!Casas14



### Add column and values ​​for owner address

		ALTER TABLE Nashville..Houses
		ADD OwnerDireccion NVARCHAR(255)

		UPDATE Nashville..Houses
		SET OwnerDireccion = PARSENAME(REPLACE(OwnerAddress, ',', '.'),3);
		
!Casas15


### Add column and values ​​for owner city

		ALTER TABLE Nashville..Houses
		ADD OwnerCiudad NVARCHAR(255)

		UPDATE Nashville..Houses
		SET OwnerCiudad = PARSENAME(REPLACE(OwnerAddress, ',', '.'),2);


### Add column and values ​​for owner state

		ALTER TABLE Nashville..Houses
		ADD OwnerEstado NVARCHAR(255)

		UPDATE Nashville..Houses
		SET OwnerEstado = PARSENAME(REPLACE(OwnerAddress, ',', '.'),1);


##  Review of values ​​in SoldAsVacant
### Query values ​​and frequency count

		SELECT DISTINCT (SoldAsVacant), COUNT (SoldAsVacant) AS Frecuencia
		FROM Nashville..Houses
		GROUP BY SoldAsVacant
		ORDER BY Frecuencia;
		
!Casas16

### Query values ​​allowed in SoldAsVacant

		SELECT SoldAsVacant, 
		CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
			 WHEN SoldAsVacant = 'N' THEN 'No'
			 ELSE SoldAsVacant
			 END
		FROM Nashville..Houses

### Updating allowed values ​​in SoldAsVacant

		UPDATE Nashville..Houses
		SET SoldAsVacant = (CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
			 WHEN SoldAsVacant = 'N' THEN 'No'
			 ELSE SoldAsVacant
			 END)
!Casas17
!Casas18


## Duplicate removal

 Query duplicate records based on columns:
 ParcelId, PropertyAddress, SalePrice, SaleDate, LegalReference

		WITH TablaTem AS(
		SELECT *,
			ROW_NUMBER() OVER(
			PARTITION BY ParcelId,
						 PropertyAddress,
						 SalePrice,
						 SaleDateConverted,
						 LegalReference
						 ORDER BY
							UniqueID
							) AS DUPLICADO
		FROM Nashville..Houses
		)
		SELECT *
		FROM TablaTem
		WHERE DUPLICADO >1


### Duplicate removal

		WITH TablaTem AS(
		SELECT *,
			ROW_NUMBER() OVER(
			PARTITION BY ParcelId,
						 PropertyAddress,
						 SalePrice,
						 SaleDateConverted,
						 LegalReference
						 ORDER BY
							UniqueID
							) AS DUPLICADO
		FROM Nashville..Houses
		)
		DELETE
		FROM TablaTem
		WHERE DUPLICADO >1
		
!Casas19


### Remove unnecessary columns ###

		ALTER TABLE Nashville..Houses
		DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress
		
!Casas20


##  Remove unnecessary spaces in string attributes
### LandUse attribute validation with values ​​with extra spaces 

		SELECT LandUse, TRIM(REPLACE(LandUse,'  ','')), LEN(LandUse), LEN(TRIM(REPLACE(LandUse,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(LandUse) <> LEN(TRIM(REPLACE(LandUse,'  ','')))
		
!Casas21


### Value update CONDOMINIUM OFC  OR OTHER COM CONDO ###> CONDOMINIUM OFC OR OTHER COM CONDO

		UPDATE Nashville..Houses
		SET LandUse = (CASE WHEN LandUse = 'CONDOMINIUM OFC  OR OTHER COM CONDO' THEN 'CONDOMINIUM OFC OR OTHER COM CONDO'
			 ELSE LandUse
			 END)
			 
!Casas22


### Update validation

		SELECT LandUse, COUNT(LandUse) AS Frecuencia
		FROM Nashville..Houses
		GROUP BY LandUse
		ORDER BY Frecuencia


### Validation of UniqueId attribute with values ​​with extra spaces, does not have.

		SELECT [UniqueID], TRIM(REPLACE([UniqueID],'  ','')), LEN([UniqueID]), LEN(TRIM(REPLACE([UniqueID],'  ','')))
		FROM Nashville..Houses
		WHERE LEN([UniqueID]) <> LEN(TRIM(REPLACE([UniqueID],'  ','')))

### ParcelID attribute validation with values ​​with extra spaces, does not have.

		SELECT ParcelId, TRIM(REPLACE(ParcelId,'  ','')), LEN(ParcelId), LEN(TRIM(REPLACE(ParcelId,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(ParcelId) <> LEN(TRIM(REPLACE(ParcelId,'  ','')))

### LegalReference attribute validation with values ​​with extra spaces, does not have.

		SELECT LegalReference, TRIM(REPLACE(LegalReference,'  ','')), LEN(LegalReference), LEN(TRIM(REPLACE(LegalReference,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(LegalReference) <> LEN(TRIM(REPLACE(LegalReference,'  ','')))

### Validation of OwnerName attribute with values ​​with extra spaces, double spaces inside the string.

		SELECT OwnerName, TRIM(REPLACE(OwnerName,'  ','')), LEN(OwnerName), LEN(TRIM(REPLACE(OwnerName,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(OwnerName) <> LEN(TRIM(REPLACE(OwnerName,'  ','')));


### Update the value in OwnerName by removing double spaces
		UPDATE Nashville..Houses
		SET OwnerName = TRIM(REPLACE(OwnerName,'  ',' '));

### Update validation

		SELECT OwnerName, TRIM(REPLACE(OwnerName,'  ','')), LEN(OwnerName), LEN(TRIM(REPLACE(OwnerName,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(OwnerName) <> LEN(TRIM(REPLACE(OwnerName,'  ','')));


### Validation of PropertyAddress attribute with values ​​with additional spaces, 55788 values ​​lines with double, triple, leading or trailing spaces.

		SELECT PropertyDireccion, TRIM(REPLACE(PropertyDireccion,'  ','')), LEN(PropertyDireccion), LEN(TRIM(REPLACE(PropertyDireccion,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(PropertyDireccion) <> LEN(TRIM(REPLACE(PropertyDireccion,'  ','')));

### Updating the value in PropertyAddress removing triple spaces
		UPDATE Nashville..Houses
		SET PropertyDireccion = TRIM(REPLACE(PropertyDireccion,'   ',' '));

### Updating the value in PropertyAddress removing double spaces
		UPDATE Nashville..Houses
		SET PropertyDireccion = TRIM(REPLACE(PropertyDireccion,'  ',' '));


### Validation of PropertyCity attribute with values ​​with extra spaces, there are none.

		SELECT PropertyCiudad, TRIM(REPLACE(PropertyCiudad,'  ','')), LEN(PropertyCiudad), LEN(TRIM(REPLACE(PropertyCiudad,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(PropertyCiudad) <> LEN(TRIM(REPLACE(PropertyCiudad,'  ','')));

### Validation of OwnerAddress attribute with values ​​with additional spaces, 25273 values ​​lines with double, triple spaces, at the beginning or end.

		SELECT OwnerDireccion, TRIM(REPLACE(OwnerDireccion,'  ','')), LEN(OwnerDireccion), LEN(TRIM(REPLACE(OwnerDireccion,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(OwnerDireccion) <> LEN(TRIM(REPLACE(OwnerDireccion,'  ','')));

### Updating the value in OwnerAddress removing triple spaces
		UPDATE Nashville..Houses
		SET OwnerDireccion = TRIM(REPLACE(OwnerDireccion,'   ',' '));

### Updating the value in OwnerAddress removing double spaces
		UPDATE Nashville..Houses
		SET OwnerDireccion = TRIM(REPLACE(OwnerDireccion,'  ',' '));

### Validation of OwnerCity attribute with values ​​with additional spaces, 25969 values ​​lines with double, triple spaces, at the beginning or end.

		SELECT OwnerCiudad, TRIM(REPLACE(OwnerCiudad,'  ','')), LEN(OwnerCiudad), LEN(TRIM(REPLACE(OwnerCiudad,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(OwnerCiudad) <> LEN(TRIM(REPLACE(OwnerCiudad,'  ','')));

		SELECT DISTINCT(OwnerCiudad)
		FROM Nashville..Houses
		WHERE LEN(OwnerCiudad) <> LEN(TRIM(REPLACE(OwnerCiudad,'  ','')));

!Casas23

/*
Cities with errors that should be corrected in the database
 OLD HICKORY
 WHITES CREEK
 MOUNT JULIET
 JOELTON
 GOODLETTSVILLE
 ANTIOCH
 BELLEVUE
 MADISON
 NASHVILLE
 NOLENSVILLE
 HERMITAGE
 BRENTWOOD
 */
 
 ### Update of the value in OwnerCiudad eliminating double spaces
		
	UPDATE Nashville..Houses
	SET OwnerCiudad = TRIM(REPLACE(OwnerCiudad,'  ',''));
		

 ### Updating the value type to the YearBuilt attribute from float to int
		
	ALTER TABLE Nashville.dbo.Houses ALTER COLUMN YearBuilt int;  
	GO 
	
!Casas24
