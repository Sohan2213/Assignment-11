1) Create the CustomerDim table

CREATE TABLE CustomerDim (
    CustomerID INT,
    CustomerName VARCHAR(100),
    Address VARCHAR(200),
    EffectiveStartDate DATE,
    EffectiveEndDate DATE,
    IsCurrent BIT
);

2) Insert initial data
INSERT INTO CustomerDim (CustomerID, CustomerName, Address, EffectiveStartDate, EffectiveEndDate, IsCurrent)
VALUES 
(1, 'Sohan Mohanty', 'Bhubaneswar', '2023-01-01', '9999-12-31', 1),
(2, 'Yajy Goswami', 'Jharkhand', '2023-01-01', '9999-12-31', 1),
(3, 'Saket Ranjan', 'Bihar', '2023-01-01', '9999-12-31', 1);

3) Create the trigger
CREATE TRIGGER trg_dim
ON CustomerDim
INSTEAD OF INSERT
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @CurrentDate DATE = GETDATE();

    -- Update existing records
    UPDATE cd
    SET 
        cd.EffectiveEndDate = DATEADD(DAY, -1, @CurrentDate),
        cd.IsCurrent = 0
    FROM CustomerDim cd
    INNER JOIN inserted i ON cd.CustomerID = i.CustomerID
    WHERE cd.IsCurrent = 1 AND 
          (cd.CustomerName != i.CustomerName OR cd.Address != i.Address);

    -- Insert new records
    INSERT INTO CustomerDim (CustomerID, CustomerName, Address, EffectiveStartDate, EffectiveEndDate, IsCurrent)
    SELECT 
        i.CustomerID,
        i.CustomerName,
        i.Address,
        @CurrentDate,
        '9999-12-31',
        1
    FROM inserted i
    LEFT JOIN CustomerDim cd ON i.CustomerID = cd.CustomerID AND cd.IsCurrent = 1
    WHERE cd.CustomerID IS NULL OR cd.CustomerName != i.CustomerName OR cd.Address != i.Address;
END;

4) Insert new records
INSERT INTO CustomerDim (CustomerID, CustomerName, Address)
VALUES 
(1, 'Sohan', 'Ajmer'),
(4, 'Sovan', 'Mumbai'),
(3, 'Murty', 'Chennai'),
(5, 'Aniket', 'Mumbai');

5) View the results
SELECT * FROM CustomerDim ORDER BY CustomerID, EffectiveStartDate;
