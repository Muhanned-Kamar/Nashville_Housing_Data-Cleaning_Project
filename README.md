# Nashville_Housing_Data_Cleaning_Project

## Introducation

In this Project I will be cleaning the [Nashville Houseing Data](https://github.com/AlexTheAnalyst/PortfolioProjects/blob/main/Nashville%20Housing%20Data%20for%20Data%20Cleaning.xlsx) and try to make this data more relaible.

### SQL Query

#### Selecting the database

    USE NashvilleProject

#### Viewing the data

    SELECT *

    FROM Table1

#### Sales date take time off

    SELECT SaleDate	,SaleDateConverted
    
    FROM Table1

#### Converting format type to DATE

    ALTER TABLE Table1
    ADD SaleDateConverted DATE;
    
    UPDATE Table1
    SET SaleDateConverted = CONVERT(DATE, SaleDate)

#### Populate Property Adderss Date

    SELECT * 
    
    FROM Table1

    SELECT 
    F.ParcelID , F.PropertyAddress, 
    S.ParcelID , S.PropertyAddress
    
    FROM Table1 AS F
    JOIN Table1 AS S
         ON F.ParcelID = S.ParcelID
	       AND F.[UniqueID ] <> S.[UniqueID ]

    UPDATE F
    SET PropertyAddress = ISNULL(F.PropertyAddress,S.PropertyAddress)
    FROM Table1 AS F
    JOIN Table1 AS S
         ON F.ParcelID = S.ParcelID
	       AND F.[UniqueID ] <> S.[UniqueID ]
	       WHERE F.PropertyAddress IS NULL 


#### Breaking Out Address Into Individual Columns (Address, City, State) seperating with (,)

    SELECT PropertyAddress
    FROM Table1

    SELECT 
    SUBSTRING(PropertyAddress, 1, CHARINDEX(',',PropertyAddress)-1) AS Address,
    SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1, LEN(PropertyAddress)) AS  CITY -- CHATINDEX defines the position in which the output we want to get so the (,) is the delimiter the name of the column and the +1 is for the postion to get the (,) out
    
    FROM Table1

#### Creating new columns for the new data

    ALTER TABLE Table1
    ADD PropertySplitAddress NVARCHAR(255);
    
    UPDATE Table1
    SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',',PropertyAddress)-1)
    
    ALTER TABLE Table1
    ADD PropertySplitCity NVARCHAR(255);
    
    UPDATE Table1
    SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1, LEN(PropertyAddress))
    
    SELECT *
    
    FROM Table1

#### Using Parsename to seperate the owner address

    SELECT 
    PARSENAME(REPLACE(OwnerAddress,',','.'),3), -- parsename takes the string of the place '1,2,3' 
    PARSENAME(REPLACE(OwnerAddress,',','.'),2), -- Replace makes the (,) to a . because parsename default delimiter is (.) 
    PARSENAME(REPLACE(OwnerAddress,',','.'),1)
    
    FROM Table1

#### Adding new Columns for the new data

##### Address

    ALTER TABLE Table1
    ADD OwnerSplitAddress NVARCHAR(255);
    
    UPDATE Table1
    SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress,',','.'),3)

##### City

    ALTER TABLE Table1
    ADD OwnerSplitCity NVARCHAR(255);
    
    UPDATE Table1
    SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress,',','.'),2)

##### State

    ALTER TABLE Table1
    ADD OwnerSplitState NVARCHAR(255);

    UPDATE Table1
    SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress,',','.'),1)

    SELECT *
    
    FROM Table1

#### Changing Y and N in the SoldAsVacant to YES and NO

    SELECT SoldAsVacant,
    CASE WHEN SoldAsVacant = 'Y' THEN 'YES'
         WHEN SoldAsVacant = 'N' THEN 'NO'
	       ELSE SoldAsVacant
	  END

    FROM Table1

#### Update the column

    UPDATE Table1
    SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'YES'
        WHEN SoldAsVacant = 'N' THEN 'NO'
	      ELSE SoldAsVacant
	  END

    SELECT DISTINCT (SoldAsVacant), COUNT(SoldAsVacant)
    
    FROM Table1
    GROUP BY SoldAsVacant
    ORDER BY 2

#### Remove Duplicates ' Not recommended to use it without asking the supervisor' 

##### First identify the duplicates

    WITH RowNumCTE AS (
    SELECT *,
    ROW_NUMBER() OVER(
    PARTITION BY ParcelID,
                 PropertyAddress,
	         SalePrice,
		 SaleDate,
		 LegalReference
		 ORDER BY UniqueID
		      ) AS Row_num
    
    FROM Table1
                     )
                     
    SELECT *
    
    FROM RowNumCTE

#### Delete Duplicates

    DELETE
    
    FROM RowNumCTE
    WHERE Row_num > 1

#### Delete Unused Columns (PropertAddress , OwnerAddress , TaxDistrict ,SaleDate)

    ALTER TABLE Table1
    DROP COLUMN TaxDistrict 

    ALTER TABLE Table1
    DROP COLUMN SaleDate
    
    ALTER TABLE Table1
    DROP COLUMN PropertyAddress , OwnerAddress

#### Viewing the data after cleaning

    SELECT *
    FROM Table1


