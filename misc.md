# Mysql on Mac
1. Download [Mac OS X 10.12 (x86, 64-bit), DMG Archive](https://dev.mysql.com/downloads/mysql/) and install.
2. Download [Mysql Workbench](https://dev.mysql.com/downloads/workbench/) and install.
3. While installing, record the password, e.g. root@localhost: poIrw.GlP7i.
4. In System Preferences, navigate to MySQL, click Start MySQL Server
5. Open Workbench, click on the circle plus sign on the right side of MysQL Connections, Connection Name: local, click Password and put in the password obtained above.
6. Test Connection would fail; all you need to do is to create the connection and login. It will prompt you for a new password.
7. Right click the empty area on the left panel under schemas and create database `testdb` which is essentially 
	```
	CREATE SCHEMA `testdb` ;
	ALTER DATABASE testdb CHARACTER SET utf8 COLLATE utf8_general_ci;
	```
	
	```
	CREATE TABLE `testdb`.`tb_recruitment` (
  `Employer` VARCHAR(100) NOT NULL,
  `Source` VARCHAR(50) NULL,
  `Position` VARCHAR(50) NOT NULL,
  `MinPay` INT NULL,
  `MaxPay` INT NULL,
  `UnitPay` VARCHAR(10) NULL,
  `Currency` VARCHAR(10) NULL,
  `MinExp` INT NULL,
  `MaxExp` INT NULL,
  `PositionType` VARCHAR(50) NULL,
  `Keywords` VARCHAR(50) NULL,
  `Industry` VARCHAR(50) NULL,
  PRIMARY KEY (`Employer`, `Position`));
  ```
 
  ```
SET CHARACTER SET 'utf8';
SET collation_connection = 'utf8_general_ci';
load data local infile '/Users/osbertngok/Downloads/cleansed_data.txt' 
into table testdb.tb_recruitment
CHARACTER SET 'utf8'
       fields terminated by '\t' lines terminated by '\n' 
       (@col1, @col2, @col3, @col4, @col5, @col6, @col7, @col8, @col9, @col10, @col11, @col12) 
       set 
       `Employer`=@col1,
       `Source`=@col2,
       `Position`=@col3,
       `MinPay`=@col4,
       `MaxPay`=@col5,
       `UnitPay`=@col6,
       `Currency`=@col7,
       `MinExp`=@col8,
       `MaxExp`=@col9,
       `PositionType`=@col10,
       `Keywords`=@col11,
       `Industry`=@col12;
   ```
 
# Example Queries:

### Find the average min pay for all positions, where Position is either 工程师 or 研发, and the minPay makes sense (e.g. > 10)

``` 
SELECT AVG(`MinPay`) 
FROM `testdb`.`tb_recruitment` 
WHERE `Position` in ('工程师','研发')
AND `MinPay` > 10;
```

### For the records where Position is either 工程师 or 研发, and the minPay makes sense (e.g. > 10), Find the average min pay and number of records per Employer.

```
SELECT `Employer`, AVG(`MinPay`), COUNT(1)
FROM `testdb`.`tb_recruitment` 
WHERE `Position` in ('工程师','研发')
AND `MinPay` > 10
GROUP BY `Employer`
ORDER BY AVG(`MinPay`) ASC;
```

### Select all positions where Position contains keyword 工程师

```
SElECT * FROM `testdb`.`tb_recruitment` 
WHERE `Position` LIKE '%工程师%'
```

### Append business registration record to all Positions

```
SELECT *
FROM tb_recruitment, tb_business_registration
WHERE tb_recruitment.enterprise = tb_business_registration.enterprise
```

### Count number of Positions that provides OT meal, high tea, and dinner per enterpise

```
SELECT enterprise,
COUNT(NULLIF(ot, '')),
COUNT(NULLIF(ht, '')),
COUNT(NULLIF(dinner, '')),
FROM tb_recruitment
GROUP BY enterprise
```

### Return all positions whose salary is more than the average of the enterprise it belongs to

```
SELECT tb1.*
FROM tb_recruitment AS tb1,
(SELECT enterprise, AVG(salary) AS avg_salary
FROM tb_recruitment
GROUP BY enterprise) AS tb2
WHERE tb1.enterprise = tb.enterprise
AND tb1.salary > tb2.avg_salary
```