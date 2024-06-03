# Inventory Balancing
## Introduction
In this project, I use Python to coordinate inventory at stores within the system. By using historical sales data and current inventory data, I perform analysis and evaluation to ensure products are getting to the right stores where they are in demand, minimizing stockouts, and preventing overcrowding at stores with excessive inventory.
## Business context
The data used pertains to fashion items at a retailer with distribution stores across the country. Products are distributed to these stores for storage, display, and sales.

For fashion items, having a variety of product sizes is crucial to accommodate the diverse needs of customers. The absence of sizes within the same product can potentially hinder product sales, as customers may struggle to find the right size. Therefore, ensuring that stores have adequate stock of products in various sizes is essential to facilitate faster sales, particularly towards the end of the season.

<img src="https://github.com/ducpham131/Inventory-Balancing/assets/169105426/9355a49b-8830-4058-adc6-b66e86030907" alt="..." width="400" />

Based on the inventory data of the stores in the system, I was tasked with creating transfer orders to balance the products across the stores. The objective of this activity is to leverage the available sizes in stock at certain stores to fill size gaps in others. Additionally, the aim is to gather products with low and sparse inventory from various stores to the best-selling stores. This ensures there are enough items on display shelves and increases the likelihood of selling out by the end of season.

This task consumes a lot of time and operations when done on spreadsheets when dealing with large amounts of data. So I replaced the spreadsheet with Python to save time and improve performance.
## Datasets

<img src="https://github.com/ducpham131/Inventory-Balancing/assets/169105426/70f63071-4f47-44c5-8cbd-e81bc96492bb" alt="..." width="500" />

### Explain the conventions
#### Store conventions
Stores in the dataset are represented as 3-character codes of cities/provinces.
For excample: 
- DNA: Da Nang city
- HCM: Ho Chi Minh city
#### product_id conventions
The `product_id` consists of 9 characters and contains encrypted information about the product. This information can be deciphered to extract additional details about the product, such as the `Product group`, `Product line`, `Product number` and `Size`.

<img src="https://github.com/ducpham131/Inventory-Balancing/assets/169105426/0f1d4018-7508-498f-865f-f4aa26553e6c" alt="..." width="500" />

- Product Rroup: denoted by the first 2 numbers of product_id. For example: 40 - Office wear Product group.
- Product line: The third character in “product_id” represents the product line. In this dataset, there are 2 product lines: women's clothes corresponding to the letter W and men's clothes corresponding to the letter M.
- Product number: The next 4 digits are the product code used to distinguish products in the same group.
- Size: The last 2 characters represent the size of the product. There are 5 sizes “Small”, “Medium”, “Large”, “X-Large”, “XX-Large” in the order of “S1”, “S2”, “S3”, “S4”, “S5”. For example: S1 - Small size
### Table 1: sales
The sales table contains information about the stores' orders for 6 months, each line records 1 product sold.
|store|order_time|order_id|customer_name|product_id|quantity|
|:----|:--------:|:------:|:-----------:|:--------:|-------:|
|QNH|1/1/23 9:02|QNH01342|N N P N|35W7885S5|1|
|HYE|1/1/23 9:57|HYE01588|C T B V|33W6741S4|1|
|DNA|1/1/23 10:13|DNA02155|M T|35W3595S3|1|
|HNO|1/1/23 10:53|HNO01122|M M|22M5734S4|1|
|VTB|1/1/23 11:05|VTB01351|M H|20M5070S3|1|
|TBH|1/1/23 11:06|TBH01087|L A T|35W3595S1|1|
### Table 2: inventory
The inventory table provides information about the inventory status of stores within the system.
Columns BGI to VTB represent the inventory levels at stores, each identified by its corresponding 3-letter code.
Additionally, it's important to note that some stores only sell women's clothing and do not carry men's clothing. Therefore, caution is needed when transfering to avoid any misleading. Stores that exclusively sell women's clothing and do not carry men's clothing include: DNA, HYE, QNA, QNH, and TBH.
|product_id|product_name|total_inventory|BGI|DNA|DNG|GLA|HCM|HNO|HYE|LAN|QNA|QNH|TBH|THA|TNG|VPH|VTB|
|:---------|:----------:|:-------------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|--:|
|10W7740S2|Short-sleeve shirt|191|14|17|15|11|12|11|15|16|13|11|11|10|12|11|12|
|20W4085S2|Shorts|167|10|13|8|7|8|10|13|14|11|16|9|10|14|8|16|
|31W3580S2|Knit dress|152|9|12|12|1|11|11|13|11|11|10|8|8|15|10|10|
|11W8040S1|Long-sleeve shirt|129|3|14|11|1|7|3|4|8|2|9|12|8|4|17|26|
|20W4085S1|Shorts|125|10|8|8|5|4|7|11|9|9|10|6|8|10|10|10|
## Implementation steps
### Step 1: Prepare
The first step I took was to import the necessary libraries, import data from the **inventory** and **sales** tables, and declare the stores in lists.
```c
//1 Import libraries
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import math

//2 Import Inventory data and Sales data
// Inventory data
inventory_data = pd.read_excel(r"inventory.xlsx")
// Sales data
sales_data = pd.read_excel(r"sales.xlsx")


//3 Declare stores's 3 letter code in lists
// Declare stores sell the main product line - women fashion
stores = ['BGI', 'DNA', 'DNG', 'GLA', 'HCM', 'HNO', 'HYE', 'LAN', 'QNA', 'QNH', 'TBH', 'THA', 'TNG', 'VPH', 'VTB']
// Declare stores sell secondary product line - men fashion
men_stores = ['BGI', 'DNG', 'GLA', 'HCM', 'HNO', 'LAN', 'THA', 'TNG', 'VPH', 'VTB']
```
### Step 2: Process inventory data
First, I create `product_code`, `size`, `product_group`, `product_line` columns from `product_id`
```c
//Extract 'product_code', 'size', 'product_line' and 'product_group' from 'product_id'
inventory_data['product_code'] = inventory_data['product_id'].str[0:7]
inventory_data['size'] = inventory_data['product_id'].str[8:9]
inventory_data['product_line'] = inventory_data['product_id'].str[2:3]
inventory_data['product_group'] = inventory_data['product_id'].str[0:2]
```
To provide a quick review, I draw a column chart to show the “Total Inventory by Sizes”. The three sizes *Small*, *Medium*, and *Large* have the highest inventory, while the sizes *X-large* and *XX-large* have relatively limited inventory compared to the other sizes. This is appropriate because the demand for the first three sizes usually accounts for a higher proportion than the "extra" sizes.
```c
size_inventory = inventory_data.groupby('size').agg({'total_inventory':sum}).reset_index()
plt.bar(size_inventory['size'], size_inventory['total_inventory'],
color=['skyblue','gold', 'lightgreen', 'coral', 'salmon'])
plt.title('Total Inventory by Sizes')
plt.xlabel('Sizes')
plt.ylabel('Total Inventory')
plt.show()
```
![size plot](https://github.com/ducpham131/Inventory-Balancing/assets/169105426/e4b70e32-1da6-4dcb-87d5-2e35c89cdcaf)
### Step 3: Process sales data
Similar to the inventory table above, I also created the columns `product_code`, `size`, `product_group`, `product_line` from `product_id`.
```c
// Extract 'product_code', 'size', 'product_line' and 'product_group' from 'product_id'
sales_data['product_code'] = sales_data['product_id'].str[0:7]
sales_data['product_group'] = sales_data['product_id'].str[0:2]
sales_data['product_line'] = sales_data['product_id'].str[2:3]
```
