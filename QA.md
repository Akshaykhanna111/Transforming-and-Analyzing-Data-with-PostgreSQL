What are your risk areas? Identify and describe them.
We saw that in the dataset there are many such scnearios where the integrity and completeness of data is compromised. To quote few examples -
1. Nulls or missing values in critical parameters
2. Data entry issues in the website, where country or city entered is incorrect or junk
3. Ensuring that referential integrity principles are followed at all times. There were SKU records in sessons that were not
   present in the products table
5. Checking for anomalies in the dataset. for example units ordered cannot be negative
6. Conflicting data captured in a table or across tables for example one SKU shall have only one product description which wasn't the case in the dataset provided
7. Implementing check constraints wherever not added, for example details like pincode, email address etc. have to be as per the expected format. Implementing such constraints are very difficult and same can be easily implemented through DQ check functions. 


QA Process:
Describe your QA process and include the SQL queries used to execute it.

Below is a function written to conduct 3 DQ checks in a single go. Everytime it's executed it will check for certain failure 
scenarios and if such cases are found it will store the error messages in an array and return that to the user. 

Code - 

```sql
CREATE OR REPLACE FUNCTION dq_checks()
RETURNS VOID AS $$
DECLARE
	errors text[];
    row_count int;
BEGIN
    -- check for rows where product_sku in sessions table
	-- does not match with that in products table
    SELECT COUNT(*) INTO row_count FROM public.all_sessions
	where "productSKU" NOT IN 
	(SELECT DISTINCT "SKU" FROM public.products);

    IF row_count > 0 THEN
        errors := array_append(errors, 'Product SKU not found in products table');
    END IF;

	row_count := 0;

	SELECT COUNT(*) INTO row_count FROM public.all_sessions
	where country = '(not set)';
	
	IF row_count > 0 THEN
        errors := array_append(errors, 'Invalid country entry');
    END IF;


	row_count := 0;
	
	SELECT COUNT(*) INTO row_count FROM
	(select "productSKU", count(*) from
	(select distinct "productSKU","v2ProductName"
	from public.all_sessions) t1
	group by "productSKU"
	having count(*) > 1 ) table1;
	
	IF row_count > 0 THEN
        errors := array_append(errors, 'Same SKU mapped to multiple product names');
    END IF;

	IF array_length(errors, 1) IS NOT NULL THEN
        RAISE EXCEPTION '%', array_to_string(errors, '; ');
    END IF;

    RAISE NOTICE 'DQ check passed successfully';
END $$ LANGUAGE plpgsql;

select dq_checks()



```
