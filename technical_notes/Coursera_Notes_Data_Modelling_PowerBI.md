- **Cardinality (Table Relationships):**
    
    - Defines how tables connect and ensures correct analysis.
        
    - **Types of relationships:**
        
        - **One-to-One**: Each record in one table maps to a unique record in another.  
            _Example_:  AdventureWorks has a customer table with unique customer IDs and a customer details table containing each customer's tax ID. Each unique customer can be connected to only a single tax ID.
            
        - **One-to-Many (or Many-to-One)**: A record in one table relates to multiple in another.  
            _Example_: AdventureWorks lists its stores in table A and its employees in table B. Since each store has multiple employees, that would be a one-to-many relationship.
          
			## Example: A Restaurant Management System
			### Scenario
			A restaurant needs to manage its employees and the orders they take from customers.
			### Tables
			- **Table A (Employees)**: Contains records of all employees. Each employee has a unique **Employee ID**.  
			- **Table B (Orders)**: Contains all customer orders. Each order has a unique **Order ID** and is linked to the **Employee ID** of the employee who took the order.  
			### Relationship
			**One-to-Many Relationship**:  
			- One employee (from Table A) can take many orders (in Table B).  
			- Each order is associated with only one employee.  
			
			### Relevance to the Learner
			1. **Understanding Relationships**  
			   - Helps organize data effectively.  
			   - In a restaurant management system, linking employees to orders is crucial for performance tracking and customer service.  
			2. **Data Analysis**  
			   - Example questions the system can answer:  
				 - Which employee took the most orders in a week?  
				 - What is the average number of orders per employee?  
				 - How can service be improved based on employee performance?  


            
        - **Many-to-Many**: Records in both tables map to multiple records in each other.  
            _Example_: The AdventureWorks dataset contains two main tables, one for products and another for salespeople. Each salesperson can sell more than one type of bike, and each type of bike can be sold by more than one salesperson. For example, salesperson A might sell mountain bikes, while salesperson B might also sell mountain bikes in addition to road bikes.
            
- **Granularity (Level of Detail):**
    
    - Refers to how detailed the data is, influencing insights and analysis depth.
    - this should align with business question you need to answer
    -  _Example of high granularity_: Customer purchase history with detailed transaction data, or sales data at the level of continents, countries, states, cities, and even individual stores.    
	- _Example of low granularity_: Monthly sales of a product category summarized only on a monthly basis, or the total sales of all the stores combined.
        
- **Schemas:**    
	- **Flat Schema**: Stores all attributes in a single table. Simple and easy to query but leads to redundancy, large datasets, and slower performance. Suitable only for small datasets.
	
	- **Star Schema**: Uses a central **fact table** linked to surrounding **dimension tables** (e.g., customers, products, dates). This reduces redundancy, improves performance, and provides a clearer structure.
	
	- **Snowflake Schema**: Extends the star schema by breaking dimension tables into sub-tables (e.g., product → category → subcategory). It offers better storage efficiency, consistency, and scalability but is more complex to manage and query.
	- **Choosing between Star and Snowflake schema **:
	- **Star Schema – Pros & Cons**
		**Advantages**
		- Central **fact table** with surrounding **dimension tables**.  
		  - Simple structure → easy navigation & faster queries.    
		- Ideal for **smaller datasets** or non-hierarchical relationships.    
		- Great for **dashboards and quick reports**.    
		- Example: Adventure Works analyzing **monthly sales data** (facts linked to product, date, region). 
		
		**Disadvantages**
		- Can oversimplify complex datasets.    
		- Leads to **data redundancy & integrity issues**.    
		- Struggles with **hierarchies** (e.g., region → sub-region → city).    
		- Updates (like region/category changes) must be applied to multiple records.
    
	 **Snowflake Schema – Pros & Cons**
		**Advantages**
				- Breaks dimension tables into **multiple related tables**.    
				- Supports **hierarchies and sub-dimensions** (e.g., region → sub-region → city).    
				- Enables **granular analysis** (sales by city, sub-region, etc.).    
				- Reduces redundancy → each attribute stored once.    
				- Ensures **data integrity** and **lower storage requirements**.   
			
		**Disadvantages**
			    - More **complex structure**, harder to navigate.    
				- Slower to locate fields → can delay quick insights.    
				- Requires more **joins**, which may impact query speed.    
	**Adventure Works Example**
			- **Star schema**: simple but overly denormalized, risks inaccuracies.    
			- **Snowflake schema**:    
			    - Split **Product Dimension** → Product Category & Subcategory.        
			    - Organize customers by **Country → State → City**.        
			    - Enables deeper insights into **sales patterns** and **marketing effectiveness**.        
			    - Supports **hierarchical analysis** without redundancy.
- **Facts and Dimensions:**
	- Facts contains **transactional data**. Say for example, in Adventure Works has a data model that includes the following tables: **Calendar,** which stores date information; **Orders,** where order transactions are tracked; and **Customer,** which holds information about customers. The **Orders** table is the fact table as it contains transactional data that can be analyzed using the other dimension tables
- ### **Best Practices for Ethical Data Use**
	**1. Data Anonymization & Masking**	
	- Protect PII and sensitive data; comply with GDPR/CCPA.	    
	- **Static masking** → replace with placeholders (e.g., `***@example.com`).	    
	- **Role-level security (RLS)** → restrict data visibility by user roles.	    
	- **Dynamic masking** → adjust visibility based on access rights.	    
	- Ensures privacy while allowing meaningful analysis.    
	
	**2. Ensuring Data Accuracy**	
	- **Data profiling** → check quality, outliers, duplicates, missing values.	    
	- **Validation rules** → enforce standards (e.g., valid product IDs, numeric scores).	    
	- **Regular data audits** → detect errors, anomalies, or outdated records.	    
	- **Regular refreshes** → keep datasets up to date for accurate reporting.    
	
	**3. Ensuring Data Integrity**	
	- **Data lineage tracking** → monitor data flow from source → transformations → reports.	    
	- **Version control** → maintain report history, revert changes if needed.	    
	- **Transparent reporting** → unbiased visuals, proper scales, consistent color schemes.	    
	- **Automated monitoring & alerts** → detect refresh issues and trigger notifications.
- ### **DAX**
	- **DAX Basics**: Syntax involves naming a column/table, using `=` followed by a function, and referencing columns/tables (e.g., `'Sales'[Quantity]`).
	- **Functions in DAX**: Function names in capital letters, followed by parentheses with parameters. Example: distinct count of customer keys in a sales table.
	- **Variables**: Declared with `VAR` and finalized with `RETURN` to simplify formulas, improve readability, and reduce complexity. Example: define variables for sales amount and product number, then return their ratio.
	- DAX is not case-sensitive, but it does distinguish between blanks and zeros.    
	- Use comments to explain your code. You can use // for a single-line comment and /* ... */ for a multi-line comment. These comments do not affect the functionality of the function or formula. They do, however, help other team members understand your code and can later serve as a reminder of your thinking.    
	- Remember, many DAX functions require an **existing relationship between tables**, so ensure your data model is set up correctly.
	- **Row Context**:
		- Refers to the **current row being evaluated** in a table.    
		- Used in calculated columns where formulas iterate row by row.
		- Does **not automatically propagate** through relationships, but you can use functions like `RELATED` or `RELATEDTABLE` to bring in values from related tables.
	- **Filter Context**:
		- Refers to the **set of filters applied** to the data before evaluation.
		-  Example: calculating total sales only for products in category **X** or restricting sales to a specific region.    
		- Filters can be combined (logical AND) and **automatically propagate** across table relationships depending on cross-filter direction.
	- ## 1. **Row Context Example**
		**Scenario**: Calculate _Total Sales per row_ in the **Sales** table.
		**Formula (Calculated Column):**
		`Total Sales = Sales[Quantity] * Sales[Unit Price]`
		**Explanation:**
			- This is a **calculated column**.    
			- For each row in the **Sales** table, DAX multiplies the `Quantity` value by `Unit Price`.    
			- The calculation is done **row by row** → that’s **Row Context**.    
			- Result: Each row gets its own “Total Sales” value.

	- ## 2. **Filter Context Example**
		 **Scenario**: Calculate _Total Sales across the entire dataset_.
		**Formula (Measure):**
		`Total Sales = SUM(Sales[Quantity] * Sales[Unit Price])`
		**Explanation:**
			- This is a **measure**, not a column.    
			- The result depends on the **filters applied**.    
			- If you place this measure in a visual filtered by `Product Category = Bikes`, the calculation only includes bike sales.        
			- Here, **Filter Context** comes from slicers, rows, columns, or filters applied in Power BI visuals.
	- ## Measures
	- **What are Measures?**    
	    - Smart, real-time calculations created with DAX.        
	    - Perform tasks like totals, averages, counts, and advanced calculations.        
	    - Automatically adapt to filters and report interactions. 
	    - Useful for creating **Calculated Tables**.
	- **Benefits of Measures:**    
	    1. **Dynamic** – update automatically based on filters/visual context.        
	    2. **Reusable** – one measure can be used across multiple reports/visuals.        
	    3. **Performance Tracking** – essential for creating KPIs.        
	    4. **Consistency** – standardized calculations ensure reliable results.
	- ### Types of Measures
	1. **Additive Measures**    
	    - Can be summed across all dimensions.        
	    - Example: _Revenue_ → can be aggregated by product, region, or month.        
	2. **Non-Additive Measures**    
	    - Cannot be meaningfully aggregated across dimensions.        
	    - Example: _Average Sales per Customer_ → averages for January and February can’t just be added; instead, total sales ÷ total customers must be recalculated.        
	3. **Semi-Additive Measures**    
	    - Can be aggregated across some dimensions, but not all.        
	    - Typically used for values at a specific point in time.        
	    - Example: _Inventory Balance_ → can be summed across product categories or locations, but not across time periods (e.g., 50 units in Jan + 60 in Feb ≠ 110 units).
- ## Cross-Filter Direction in Power BI
	- In Power BI, **cross-filter direction** controls how filters flow between related tables in a data model. This is critical for analyzing data across multiple tables without writing complex queries.
		- **Single Direction Filtering (default)**    
		    - Filters flow **one way**, usually from dimension → fact table.        
		    - Example: Filtering the _Salesperson_ table automatically filters the _Sales_ table for that person’s sales.        
		- **Bidirectional Filtering**    
		    - Filters flow **both ways** between tables.   (facts -> dimensions)     
		    - Example: Filtering _Sales_ also filters _Salesperson_ and _Product_ tables, letting you see which products each salesperson sold.        
		    -  More flexible but can reduce performance and create ambiguous filter paths. 
		- **Example**
		- ## Tables
			**Salesperson** (Dimension)
			- SalespersonID (PK)    
			- Name    
			- Region   
			
			**Product** (Dimension)
			- ProductID (PK)    
			- ProductName    
			- Category   
			
			**Sales** (Fact)
			- SalesID (PK)    
			- SalespersonID (FK)    
			- ProductID (FK)    
			- SalesAmount    
			- Date  
		- ##  Case 1: **Single Direction (Default in Power BI)**
		
		`Salesperson  --->  Sales  <---  Product    (1)                  (Many)       (1)`
		- Arrow direction: **from dimensions → fact table**    
		- Meaning:    
			- Filter Sales by Salesperson (works ✅)        
			- Filter Sales by Product (works ✅)        
			- But cannot filter Salesperson based on filtered Sales (❌)        
			- Cannot filter Product based on filtered Sales (❌)      
		
		👉 Example: _"Top sales performers overall"_ → Works fine (filter flows Salesperson → Sales).
		- ## 🔹 Case 2: **Bidirectional Filtering**
			`Salesperson  <-->  Sales  <-->  Product    (1)                  (Many)        (1)`
			- Arrow direction: **both ways**    
			- Meaning:    
			    - Can filter Sales by Salesperson or Product (✅ same as single)        
			    - Can also filter **Salesperson** based on filtered Sales (✅)        
			    - Can also filter **Product** based on filtered Sales (✅)    
			
			👉 Example: _"Top sales performers for Electronics category"_ → Works because:
			- Product filter → filters Sales → flows back to Salesperson → only those salespeople are ranked.	
- **CROSSFILTER() DAX function**     
		- In Power BI, table relationships and filter directions affect how you analyze data. Normally, filters flow in a single direction (from dimension to fact table), which can sometimes limit analysis. Permanently switching relationships to **bidirectional** may solve this but can hurt performance or impact the entire model.
		- The **`CROSSFILTER` function** solves this by temporarily changing the filter direction **only for a specific measure**, without altering the model.
		- **Example (AdventureWorks):**
			- Sales and Products tables are connected one-to-many with a single direction.    
			- By default, Product filters Sales but not vice versa.    
			- To analyze **products sold by year**, AdventureWorks uses `CROSSFILTER` inside a `CALCULATE` measure (e.g., with `DISTINCTCOUNT` on ProductKey).    
			- ** With CROSSFILTER (correct result)**
				- `Products Sold by Year = CALCULATE (     DISTINCTCOUNT ( Sales[ProductKey] ),     CROSSFILTER ( Sales[ProductKey], Product[ProductKey], BOTH ) )`
				- Here’s what happens:
					- `CROSSFILTER` forces the relationship between `Sales` and `Product` to behave as **bidirectional** only for this calculation.    
					- Now, when `Date[Year]` filters `Sales`, it flows through to `Product` and counts products sold accurately.
- ## Role-Playing Dimensions:
	- For example, in **AdventureWorks**, 
		- Sales records track **three dates** – order date, shipping date, and delivery date.    
		- Instead of creating three separate date tables, a single **date dimension** is reused in different roles.    
		- This enables analysis by order trends, shipping efficiency, or delivery performance using the same table.
		- Only **one active relationship** exists between two tables at a time (shown as a solid line).  
		- Other valid links remain **inactive** (shown as dotted lines).    
		- By default, order date is active.    
		- To analyze by shipping or delivery date, you can use the **`USERELATIONSHIP`** function inside a DAX measure.
		- DAX formula
		- #### 1. Default Sales by Order Date (uses **active** relationship)
			`Total Sales by Order Date = SUM ( Sales[SalesAmount] )`
			👉 This works directly because `OrderDate` → `Date` is the **active** relationship.
			
		- #### 2. Sales by Shipping Date (requires `USERELATIONSHIP`)
			`Total Sales by Ship Date = CALCULATE (     SUM ( Sales[SalesAmount] ),     USERELATIONSHIP ( Sales[ShipDate], Date[DateKey] ) )`
			👉 Here, `USERELATIONSHIP` temporarily **activates the ShipDate relationship** for this calculation.
		- #### 3. Sales by Delivery Date (another inactive relationship)
			`Total Sales by Delivery Date = CALCULATE (     SUM ( Sales[SalesAmount] ),     USERELATIONSHIP ( Sales[DeliveryDate], Date[DateKey] ) )`
			👉 Same idea, but now using DeliveryDate instead.
			
		    
	
	
		
		
	
