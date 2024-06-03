# Inventory Balancing
## Introduction
In this project, I use Python to coordinate inventory at stores within the system. By using historical sales data and current inventory data, I perform analysis and evaluation to ensure products are getting to the right stores where they are in demand, minimizing stockouts, and preventing overcrowding at stores with excessive inventory.
## Business context
The data used pertains to fashion items at a retailer with distribution stores across the country. Products are distributed to these stores for storage, display, and sales.

For fashion items, having a variety of product sizes is crucial to accommodate the diverse needs of customers. The absence of sizes within the same product can potentially hinder product sales, as customers may struggle to find the right size. Therefore, ensuring that stores have adequate stock of products in various sizes is essential to facilitate faster sales, particularly towards the end of the season.

<img src="https://github.com/ducpham131/Inventory-Balancing/assets/169105426/9355a49b-8830-4058-adc6-b66e86030907" alt="..." width="400" />

Based on the inventory data of the stores in the system, I was tasked with creating transfer orders to balance the products across the stores. The objective of this activity is to leverage the available sizes in stock at certain stores to fill size gaps in others. Additionally, the aim is to gather products with low and sparse inventory from various stores to the best-selling stores. This ensures there are enough items on display shelves and increases the likelihood of selling out by the end of season.

This task consumes a lot of time and operations when done on spreadsheets when dealing with large amounts of data. So I decided to replace the spreadsheet with Python to save time and improve performance.
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

- Product Rroup: denoted by the first 2 numbers of product_id.
> For example:
>   40 - Office wear Product group.

- Product line: The third character in “product_id” represents the product line. In this dataset, there are 2 product lines: women's clothes corresponding to the letter W and men's clothes corresponding to the letter M.
- Product number: The next 4 digits are the product code used to distinguish products in the same group.
- Size: The last 2 characters represent the size of the product. There are 5 sizes `Small`, `Medium`, `Large`, `X-Large`, `XX-Large` in the order of `S1`, `S2`, `S3`, `S4`, `S5`.
> For example:
>   S1 - Small size

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
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import math

// Inventory data
inventory_data = pd.read_excel(r"inventory.xlsx")
// Sales data
sales_data = pd.read_excel(r"sales.xlsx")


// Stores sell women's clothes
stores = ['BGI', 'DNA', 'DNG', 'GLA', 'HCM', 'HNO', 'HYE', 'LAN', 'QNA', 'QNH', 'TBH', 'THA', 'TNG', 'VPH', 'VTB']
// Stores sell men's clothes
men_stores = ['BGI', 'DNG', 'GLA', 'HCM', 'HNO', 'LAN', 'THA', 'TNG', 'VPH', 'VTB']
```
### Step 2: Process inventory data
I create `product_code`, `size`, `product_group`, `product_line` columns from `product_id`
```c
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
sales_data['product_code'] = sales_data['product_id'].str[0:7]
sales_data['product_group'] = sales_data['product_id'].str[0:2]
sales_data['product_line'] = sales_data['product_id'].str[2:3]
```
### Step 4: Get sales quantity of each Product Code in stores
I use `pivot_table` to create a `sales` table, with new columns being the number of sales at stores for each *Product Code*.
Next I create the table `inventory_code` including the existing codes in the warehouses.
Finally, I use `left join` to combine the `inventory_code` and `sales` tables to get the sales quantity of each *Product Code*  at the stores.
```c
 //1 Transform Sales data into Wide data 
sales = sales_data.pivot_table(index = ['product_code','product_group','product_line'], 
                               columns = 'store', values = 'quanity', aggfunc = 'sum', fill_value = 0).reset_index()

//2 Create a Dataframe contains all 'product_code' are stocking in stores
inventory_code = inventory_data[['product_code','product_line','product_group']].drop_duplicates()

//3 Merge together 
inventory_sales = inventory_code.merge(sales, on = ['product_code','product_line','product_group'], 
                                       how ='left').fillna(0)
```
### Step 5: Rank stores by their sales quantity
In this step, I sorted the stores by decreasing sales volume for each Product Code. My idea is to use these sorted lists to allocate the *Product Code* to the stores that are most likely to sell.

However, in some cases where stores have the same sales quantity or 0 sales quantity, the arrangement of stores will default to alphabetical order based on the first character of the store code. This could cause an imbalance and may not be accurate, as stores with codes that begin with letters later in the alphabet will be pushed to the end of the sorted list.

To solve this problem, I use an additional criterion, which is based on the sales quantity of each *Product Group*, to refine the sorting conditions. The heatmap graph below shows the sales volume of stores for each Product Group. For example, the QNH store sells well in Product Group 31 - Knit dress, as indicated by the darker colors, whereas the GLA store, which appears earlier alphabetically, has lower sales for this group. Adding this filter makes the sorting more accurate because it is based on the stores' ability to sell according to each "Product Group" rather than alphabetical order.
![Sale product group](https://github.com/ducpham131/Inventory-Balancing/assets/169105426/6584bfb1-c44c-4901-b6f7-7390bdf10084)

Going back to how I arranged the stores. First, as mentioned, I create an ordered list of stores according to the *Product Group* using the `product_group_sales` table. This ensures that stores are prioritized based on their sales performance for each specific product group.
```c
// Rank stores by their sales of each "product_group"
product_group_sales['stores_sort_by_sales_group'] = product_group_sales[stores].apply(lambda row: row.nlargest(len(stores)).index.to_list(), axis=1)
product_group_sales['stores_sort_by_sales_group'] = product_group_sales['stores_sort_by_sales_group'].apply(tuple)
stores_sort_by_product_group = product_group_sales[['product_group','stores_sort_by_sales_group']]
print(stores_sort_by_product_group)
```
Next, I create the `stores_rank` Data Frame by merging the `inventory_sales` and `product_group_sales`. Now, we have a Data Frame that contains the sales quantity by each product code for the stores, as well as a list of stores sorted by the sales quantity of each *Product Group*.

Finally, I write a function to sort the stores. The stores are sorted in descending order of sales volume. If stores have the same sales volume, the secondary criterion is used, which is the list in the column `stores_sort_by_sales_group`.
```c
// Merge "inventory_sales" and "product_group_sales".
stores_rank = inventory_sales.merge(stores_sort_by_product_group, on = 'product_group', how = 'left')

// Write a function to sort stores by their sales of each "product_code"
def sort_store(row):
    // Sort stores that sell men clothes
    if row['product_line'] == 'M':
        sorted_shops = sorted(zip(men_stores, row[men_stores]), 
                              key=lambda x: (-x[1], row['stores_sort_by_sales_group'].index(x[0]) 
                                             if x[0] in row['stores_sort_by_sales_group'] else float('inf')
                                            ))
    // Sort stores that sell women clothes                                        
    else:
        sorted_shops = sorted(zip(stores, row[stores]), 
                              key=lambda x: (-x[1], row['stores_sort_by_sales_group'].index(x[0]) 
                                             if x[0] in row['stores_sort_by_sales_group'] else float('inf')
                                            ))
    return [shop[0] for shop in sorted_shops]

stores_rank['stores_rank'] = stores_rank.apply(lambda row: sort_store(row), axis = 1).apply(tuple)
stores_rank = stores_rank[['product_code','stores_rank']]
```
### Step 6: Create Data Frame for calculations
First, I create the `size_of_product` Data Frame by using a pivot table to convert `inventory_data` to a wide format. The newly created columns will represent the inventory of each size for each *Product Code*.

Next, I combine `size_of_product` and `stores_rank` to create the `product_inventory` Data Frame for further calculations. I added an additional column, `total_inventory`, to calculate the total inventory for each product code. Now, we have a Data  containing inventory information for each size and a list of distribution priorities according to the *Product Code* for the stores.

Finally, I create additional columns to show information from the highest to the lowest inventory and the sizes with the highest to the lowest inventory. I will use these columns in the following calculation steps.
```c
// 1 Convert "inventory_data" to Wide data by "size"
size_of_product = inventory_data.pivot_table(index=['product_code','product_line','product_group'], 
                                             columns='size', values='total_inventory', fill_value=0, 
                                             aggfunc = 'sum').reset_index()
size_of_product['total_inventory'] = size_of_product[['1','2','3','4','5']].sum(axis = 1)

// 2 Create "product_inventory" by merging "size_of_product" and "stores_rank"
product_inventory = size_of_product.merge(stores_rank, on = 'product_code', how = 'left')


// 3 Create new columns
// Create columns return stock quantity by ranking stock quantity
product_inventory['1st_stock'] = product_inventory[['1','2','3','4','5']].apply(lambda row: row.max(), axis=1)
product_inventory['2nd_stock'] = product_inventory[['1','2','3','4','5']].apply(lambda row: row.nlargest(2).iloc[-1], axis=1)
product_inventory['3rd_stock'] = product_inventory[['1','2','3','4','5']].apply(lambda row: row.nlargest(3).iloc[-1], axis=1)
product_inventory['4th_stock'] = product_inventory[['1','2','3','4','5']].apply(lambda row: row.nlargest(4).iloc[-1], axis=1)
product_inventory['5th_stock'] = product_inventory[['1','2','3','4','5']].apply(lambda row: row.nlargest(5).iloc[-1], axis=1)

// Create columns return sizes by ranking stock quantity
product_inventory['1st_size'] = product_inventory[['1','2','3','4','5']].apply(lambda row: row.nlargest(1).index[0], axis=1)
product_inventory['2nd_size'] = product_inventory[['1','2','3','4','5']].apply(lambda row: row.nlargest(2).index[1], axis=1)
product_inventory['3rd_size'] = product_inventory[['1','2','3','4','5']].apply(lambda row: row.nlargest(3).index[2], axis=1)
product_inventory['4th_size'] = product_inventory[['1','2','3','4','5']].apply(lambda row: row.nlargest(4).index[3], axis=1)
product_inventory['5th_size'] = product_inventory[['1','2','3','4','5']].apply(lambda row: row.nlargest(5).index[4], axis=1)
```
### Step 7: Calculate minimum size run of each Product Code
A size run consists of all the sizes a particular product is manufactured in. In this project, I assume a complete size run includes the three sizes with the highest stock quantities. Normally, a size run could be *S-M-L* or *M-L-XL*. However, for some Product Codes, there is a significant difference between the 2nd highest stock quantity and the 3rd highest stock quantity. In that case, I consider the complete size run for these product codes to include just two sizes.

In this step, I create a `minimum` column to calculate the minimum size run of each *Product Code*. This means the value in the `minimum` column represents the minimum complete size run for that *Product Code* in system. I use values in the `3rd_stock` column to calculate the minimum size run. In some specific cases, as mentioned, I consider using values from the `2nd_stock` column or even the `1st_stock` column.
```c
// Create a blank list
minimum = []

// Add values into list
for index, row in product_inventory.iterrows():
    if row['3rd_stock'] != 0:
        if row['2nd_stock']/ row['3rd_stock'] >= 3 or row['2nd_stock'] - row['3rd_stock'] > 15:
            minimum.append(row['2nd_stock'])
        else:
            minimum.append(row['3rd_stock'])
    elif row['3rd_stock'] == 0:
        if row['2nd_stock'] != 0 and row['1st_stock']/ row['2nd_stock'] <= 2:
            minimum.append(row['2nd_stock']) 
        else:
            minimum.append(row['1st_stock']) 

// Create 'minimum' column
product_inventory['minimum'] = minimum
```
### Step 8: Calculate number of stores that are distributed
In this step, I calculate the number of stores to be distributed using the `total_inventory` and `minimum` columns. For Product Codes with a small total inventory, I will gather them to distribute to a few stores. Additionally, there are 15 stores selling women's clothes and 10 stores selling men's clothes. Therefore, the maximum number of stores allocated according to the product line will be 15 for women's clothes and 10 for men's clothes.
```c
# Calculate number of stores that are distributed
gather_num = []

for index, row in product_inventory.iterrows():
    if row['total_inventory'] <= 5:
        gather_num.append(1)
    elif row['total_inventory'] <= 12:
        gather_num.append(2)
    elif row['total_inventory'] <= 20:
        gather_num.append(3) 
    elif row['product_line'] == 'M' and row['minimum'] >= len(men_stores):
        gather_num.append(len(men_stores))
    elif row['product_line'] == 'W' and row['minimum'] >= len(stores):
        gather_num.append(len(stores))
    else:
        gather_num.append(row['minimum'])
        
product_inventory['distribution_num'] = gather_num
```
### Step 9: Determine how stocks are distributed
For *Product Codes* with low inventory and a small number of size run, I will gather these *Product Codes* to the best-selling stores. For *Product Codes* with large inventories, the priority is to balance the inventory by transferring excess products to stores that are lacking.
```c
decision = []

for index, row in product_inventory.iterrows():
    if row['product_line'] == 'M' and row['distribution_num'] <= (len(men_stores) - 5):
        decision.append('gather')
    elif row['product_line'] == 'W' and row['distribution_num'] <= (len(stores) -  5):
        decision.append('gather')
    else:
        decision.append('balance')
        
product_inventory['decision'] = decision
```
### Step 10: Calculate the ideal stock quantity in each distributed stores
Based on the `distribution_num` column and stock quantity in each size, I write a function to calculate the ideal stock quantity by size in each store. The outcome will be columns containing dictionaries with the size as the key and the stock quantity that should be stored as the value.
```c
// Write function
def ideal_num(row, n):
    n = int(n)
    if row == 0:
        num = 0
    else:
        if math.floor(row/n) == 0:
            num = 1
        else:
            num = math.floor(row/n)
        
    return num

// Apply function to Data Frame
product_inventory['ideal_1st_size'] = product_inventory.apply(lambda row:{row['1st_size']:ideal_num(row['1st_stock'],row['distribution_num'])}, axis =1)
product_inventory['ideal_2nd_size'] = product_inventory.apply(lambda row:{row['2nd_size']:ideal_num(row['2nd_stock'],row['distribution_num'])}, axis =1)
product_inventory['ideal_3rd_size'] = product_inventory.apply(lambda row:{row['3rd_size']:ideal_num(row['3rd_stock'],row['distribution_num'])}, axis =1)
product_inventory['ideal_4th_size'] = product_inventory.apply(lambda row:{row['4th_size']:ideal_num(row['4th_stock'],row['distribution_num'])}, axis =1)
product_inventory['ideal_5th_size'] = product_inventory.apply(lambda row:{row['5th_size']:ideal_num(row['5th_stock'],row['distribution_num'])}, axis =1)
```
> For example, `product_inventory` has values below:
> **`1st_size`**: "1"
> **`1st_stock`**: 15
> **`distribution_num`**: 10
> After executing, it will return a dictionary`{ "1": 1 }` because 15 /10 = 1
### Step 11: Determine which stores should be distributed
Based on the `store_rank` and `distribution_num` columns, I write a function to return lists of stores with high sales quantities. This means these stores should be fully stocked. The outcome will be dictionaries with the size as the key and the lists of stores as the value.
```c
# Write function
def get_top_rank(row,n):
    n = int(n)
    if isinstance(row, tuple):
        top = row[:n] if row[0] else []
    else:
        top = []
    return top


# Apply function to Data Frame
product_inventory['top_1_distribution'] = product_inventory.apply(lambda row: {row['1st_size']:get_top_rank(row['stores_rank'],row['distribution_num'])}, axis = 1)
product_inventory['top_2_distribution'] = product_inventory.apply(lambda row: {row['2nd_size']:get_top_rank(row['stores_rank'],row['distribution_num'])}, axis = 1)
product_inventory['top_3_distribution'] = product_inventory.apply(lambda row: {row['3rd_size']:get_top_rank(row['stores_rank'],row['distribution_num'])}, axis = 1)
product_inventory['top_4_distribution'] = product_inventory.apply(lambda row: {row['4th_size']:get_top_rank(row['stores_rank'],row['4th_stock'])}, axis = 1)
product_inventory['top_5_distribution'] = product_inventory.apply(lambda row: {row['5th_size']:get_top_rank(row['stores_rank'],row['5th_stock'])}, axis = 1)
```
> For example, `product_inventory` has values below:
> `1st_size`: "1"
> `store_rank`:  (DNG, BGI, LAN, TNG, THA, VPH, HCM, VTB, GLA)
> `distribution_num`: 4
> After executing, it will return a dictionary `{ "1":("DNG", "BGI", "LAN", "TNG",)}`
### Step 12: Create Stock Level Data Frame 
We have all the necessary information: the ideal stock quantity of each size for each product, the list of stores that need to be distributed, and distribution methods (gather or balance). Let's synthesize them all into a unified Data Frame.

First, I converted **`inventory_data`*** to Long format, with each record representing each store's inventory. Similarly, I also created the Data Frames **`ideal_stock`** and **`stores_distribution`** by converting **`product_inventory`** to Long format. Now, both Data Frames contain a `value` column with dictionaries. I separated the keys and values of the dictionaries to create corresponding columns. Next, I copied the two columns `product_code` and `decision` from **`product_inventory`** to create a new Data Frame called **`product_type`**. Finally, I merged them all into a unified Data Frame called **`stock_level`**.

Now we have a Data Frame containing information about the stores's inventory, the ideal stock levels, the distribution list, and the distribution method.
```c
// 1 Convert "inventory_data" to Long data
inventory_melt = inventory_data.melt(id_vars = ['product_code','size'], value_vars = stores, var_name = 'store',value_name = 'stock')

// 2 From "product_inventory", extract "Ideal stock quantity" information
ideal_stock = product_inventory.melt(id_vars = 'product_code', value_vars = ['ideal_1st_size','ideal_2nd_size','ideal_3rd_size','ideal_4th_size','ideal_5th_size'])
// Extract "size" and "ideal stock" from dictionaries
ideal_stock['size'] = ideal_stock['value'].apply(lambda x: next(iter(x.keys())))
ideal_stock['ideal_stock'] = ideal_stock['value'].apply(lambda x: next(iter(x.values())))
ideal_stock = ideal_stock[['product_code','size','ideal_stock']]

// 3 From "product_inventory", extract "Store distribution" information
stores_distribution = product_inventory.melt(id_vars = 'product_code', value_vars = ['top_1_distribution','top_2_distribution','top_3_distribution',
                                                           'top_4_distribution','top_5_distribution'])
// Extract "size" and "stores distribution list" from dictionaries
stores_distribution['size'] = stores_distribution['value'].apply(lambda x: next(iter(x.keys())))
stores_distribution['stores_distribution'] = stores_distribution['value'].apply(lambda x: next(iter(x.values())))
stores_distribution = stores_distribution[['product_code','size','stores_distribution']]


// 4 From "product_inventory", create "product_type" DataFrame
product_type = product_inventory[['product_code','decision']]

                                     
// 5 Merge "inventory_melt", "ideal_stock", "stores_distribution" and "product_type"
stock_level = inventory_melt.merge(
                                ideal_stock, on = ['product_code','size'], how = 'left').merge(
                                stores_distribution, on = ['product_code','size'], how = 'left').merge(
                                product_type, on = 'product_code', how = 'left')
```
### Step 13: Create Sending Data Frame
In this step, I create a Data Frame containing products that can be sent to other stores. I apply the following conditions to filter from the **`stock_level`** Data Frame:

- When the store is not on the distribution list and has a stock level greater than 0.
- When the store is on the distribution list and the inventory is greater than the ideal stock level.

After creating the Data Frame, I calculate the number of products that could be sent.

- For balance-products: If the store is on the distribution list, the quantity transferred will be the excess quantity compared to the ideal inventory level. If the store is not on the distribution list, keep 1 product so the store still has the opportunity to sell.
- For gather-products: If the store is on the distribution list, the quantity transferred will be the excess quantity compared to the ideal inventory level. If the store is not on the distribution list, transfer all inventory.
```c
// 1 Create DataFrame contains transferable products in stores
send_stores_list = []

for index, row in stock_level.iterrows():
    if row['store'] not in row['stores_distribution'] and row['stock'] > 0 :
        send_stores_list.append(row)
    elif row['store'] in row['stores_distribution'] and row['stock'] > row['ideal_stock']:
        send_stores_list.append(row)

send_stores = pd.DataFrame(send_stores_list)


// 2 Create "Sending quantity" column
send_quantity = []

for index, row in send_stores.iterrows():
    if row['decision'] == 'balance':
        if row['store'] in row['stores_distribution']:
            send_quantity.append(row['stock'] - row['ideal_stock'])
        else:
             send_quantity.append(row['stock'] - 1)
    else:
        if row['store'] in row['stores_distribution']:
            send_quantity.append(row['stock'] - row['ideal_stock'])
        else:
            send_quantity.append(row['stock'])

send_stores['send_quantity'] = send_quantity
send_stores = send_stores[send_stores['send_quantity'] != 0]
send_stores = send_stores.reset_index(drop = True)
print(send_stores)
```
### Step 14: Create Receiving Data Frame
Similar to the above step, I create a Data Frame containing missing products that need to be replenished in stores. I apply the following condition to filter from the **`stock_level`** Data Frame:

- When the store is on the distribution list and has inventory lower than the ideal inventory level.

Then, I create a column to calculate the amount of stock that can be received.
```c
// 1 Apply filter
receive_stores = stock_level[(stock_level.apply(lambda row: row['store'] in row['stores_distribution'], axis = 1))
                            & (stock_level['stock']< stock_level['ideal_stock'])].reset_index(drop = True)

// 2 Calculate "Receiving quantity"
receive_stores['receive_quantity'] = receive_stores['ideal_stock'] - receive_stores['stock']
print(receive_stores)
```
### Step 15: Split Sending and Receiving Data Frame by stores
I separate the sending and receiving Data Frames into individuals Data Frame by stores. 
```c
// 1 Sending Data Frame
for store in send_stores['store'].unique():
    globals()[f'{store}_send'] = send_stores[send_stores['store'] == store].copy().reset_index(drop=True)


// 2 Receiving Data Frame
for store in receive_stores['store'].unique():
    globals()[f'{store}_receive'] = receive_stores[receive_stores['store'] == store].copy().reset_index(drop=True)
```
### Step 16: Write functions to transfer products between stores 
In this step, I write functions to help quickly create transfer orders between stores.

First, I write the function ***transfer_raw***. This function allows me to create a Data Frame that merges all records from the Sending Store and Receiving tore and calculates the transfer quantity of each size for each Product Code. While calculating the transfer quantity, to improve efficiency when gathering in stores, I assume that the Receiving stores can hold 3 units more than the ideal inventory quantity of the gather-products from stores not in the distribution list. Because most of the gather-products are at stores that are not on the list and have inventory levels of 1 or 2 products (based on the plot below). This ensures that gather products can be quickly gathered to stores after each transfer.

![histogram](https://github.com/ducpham131/Inventory-Balancing/assets/169105426/05b09cd5-e651-458c-8414-2791466b898a)

```c
// Fucntion to create a raw "Sending - Receiving" DataFrame 
def transfer_raw(send_store, receive_store):
    df = send_store.merge(receive_store, on =['product_code','size','ideal_stock','stores_distribution','decision']
                            , how = 'outer', suffixes = ['_send','_receive']).fillna(0)
    
    transfer_quantity = []
    
    for index, row in df.iterrows():
        if row['store_send'] != 0 and row['store_receive'] != 0:
            if row['decision'] == 'gather' and row['store_send'] not in row['stores_distribution']:
                ## Receiving store could store 3 units greater than ideal stock quantity
                if row['send_quantity'] <= row['receive_quantity'] + 3:
                    transfer_quantity.append(row['send_quantity'])
                else:
                    transfer_quantity.append(row['receive_quantity'] + 3)
            else:
                if row['send_quantity'] <= row['receive_quantity']:
                    transfer_quantity.append(row['send_quantity'])
                else:
                    transfer_quantity.append(row['receive_quantity'])
        else:
            transfer_quantity.append(0)
    
    df['transfer_quantity'] = transfer_quantity
    df['send_stock_after_transfer'] = df['send_quantity'] - df['transfer_quantity']
    df['receive_stock_after_transfer'] = df['receive_quantity'] - df['transfer_quantity']
    
    return df
```
Next, I write the function ***transfer*** to process the DataFrame from the function ***transfer_raw***. It help removing unnecessary columns and “0” values to create a complete transfer order.
```c
// 2 Function to create a "Seding - Receiving" DataFrame
def transfer(send_store, receive_store):
    df_raw = transfer_raw(send_store, receive_store)
    
    df = df_raw[df_raw['transfer_quantity'] > 0]
    df = df[['product_code','size','store_send','store_receive','transfer_quantity']]
    
    return df
```
Because sending and receiving Data Frames can still continue to participate in transfers with other stores, I continue writing the functions ***after_send*** and ***after_receive*** to recalculate the remaining number of products that can be sent and received, and to delete products that cannot be further transferred or have received enough products.
```c
// 3 Function to process "Sending Store" after transfering
def after_send(send_store, receive_store):
    df_raw = transfer_raw(send_store, receive_store)
    
    df = df_raw[(df_raw['store_send'] != 0) & (df_raw['send_stock_after_transfer'] > 0)]
    df = df[['product_code','size','store_send','send_stock_after_transfer',
             'ideal_stock','stores_distribution','decision']]
    df = df.rename(columns = {'store_send':'store','send_stock_after_transfer':'send_quantity'})
    
    return df


// 4 Function to process "Receiving Store" after transfering
def after_receive(send_store, receive_store):
    df_raw = transfer_raw(send_store, receive_store)
    
    df = df_raw[(df_raw['store_receive'] != 0) & (df_raw['receive_stock_after_transfer'] > 0)]
    df = df[['product_code','size','store_receive','receive_stock_after_transfer',
             'ideal_stock','stores_distribution','decision']]
    df = df.rename( columns = {'store_receive':'store','receive_stock_after_transfer':'receive_quantity'})
    
    return df
```
Finally, to quickly check the transfer results, I write the function ***result*** to calculate the total number of transferred products.
```c
#5 Function to quickly check the result
def result(send_store, receive_store):
    df = transfer(send_store, receive_store)
    
    x = sum(df['transfer_quantity'])
    
    return print(x)
```
### Step 17: Draw plots
Now we have enough tools to transfer products between stores. However, it is difficult to decide which stores to transfer products to, how many products to transfer, and the priority order of stores for distribution. Because the source of transferable products and the resources used to move goods are limited, priority should be given to filling stores with good sales performance within the system. To solve this problem, I use plots to quickly make decisions about moving products between stores.

I use a heatmap to represent the transfer quantities between stores. 
```c
send_receive = transfer(send_stores, receive_stores)

heatmap_matrix = send_receive.pivot_table(index = 'store_send', columns = 'store_receive', 
                                           values = 'transfer_quantity', aggfunc = 'sum')

plt.figure(figsize=(15, 12))
sns.heatmap(heatmap_matrix, annot=True, cmap='YlGnBu', fmt='g', linewidths=.5, cbar_kws={'label': 'Transfer Quantity'})

plt.yticks(rotation=0)
plt.xticks(rotation=0)
plt.title('Transfer Quantity between Stores')
plt.xlabel('Receiving Stores')
plt.ylabel('Sending Stores')

plt.show()
```
![heatmap](https://github.com/ducpham131/Inventory-Balancing/assets/169105426/5003d67f-6392-469a-988b-ab5023fac955)

In this plot, I can quickly decide which transfer pairs will be optimal based on the total quantity for each transfer. 
>For example, for the receiving store DNA, I can choose the HCM or VTB store to transfer goods from, instead of other stores with lower efficiency. Additionally, I can identify stores that have a large number of transferable products, which means these stores are holding more inventory than others.

Next, I use a scatter plot to represent the relationship between revenue and inventory quantity of each store. 
```c
inventory_group = inventory_melt.groupby(by = 'store').agg({'stock':'sum'}).reset_index()
sales_group = sales_data.groupby(by = 'store').agg({'quantity':'sum'}).reset_index()
sales_and_inventory = inventory_group.merge(sales_group, on = 'store', how = 'left')

// Define colors for each store group
stores_sale_women_only = [ x for x in stores if x not in men_stores]
stores_sale_women_and_men = [ x for x in stores if x in men_stores]

colors = []
for store in sales_and_inventory['store']:
    if store in stores_sale_women_only:
        colors.append('blue')
    elif store in stores_sale_women_and_men:
        colors.append('green')

plt.figure(figsize=(10, 6))
plt.scatter(sales_and_inventory['stock'], sales_and_inventory['quantity'], c=colors, alpha=0.7)

plt.ylabel('Sales')
plt.xlabel('Stock Level')
plt.title('Sales vs Stock Level by Stores')

// Add annotations for each data point
for i, txt in enumerate(sales_and_inventory['store']):
    plt.annotate(txt, (sales_and_inventory['stock'].iloc[i], sales_and_inventory['quantity'].iloc[i]), 
                 textcoords="offset points", xytext=(0,5), ha='center')

// Create legend labels
legend_labels = {
    'Store sales Women clothes': 'blue',
    'Store sles Women and Men clothes': 'green',
    }

handles = [plt.Line2D([], [], marker='o', markersize=10, color=color, linestyle='None', label=label) 
           for label, color in legend_labels.items()]
plt.legend(handles=handles)

plt.show()
```
![scatter](https://github.com/ducpham131/Inventory-Balancing/assets/169105426/453dd0a6-266a-4ea1-a67b-91ff493a975b)

I can observe which store is performing the best in sales, which store needs to be restocked, and which store is holding too much inventory compared to its sales capacity.
> For example, the DNA store has the best sales but a low inventory level, so it should be prioritized for restocking. Conversely, the VTB and HCM stores have average sales but are holding nearly double the inventory compared to other stores in the same segment, so their excess inventory should be redistributed to optimize resources.

### Step 18: Perform transfering
In this step, I pair the stores to create Data Frames for sending and receiving. By using the functions above, I can quickly perform the transfer. Here is an example of how I work:
```c
// Compairing example:
TBH_HYE = transfer(TBH_send, HYE_receive)
TBH_send_1 = after_send(TBH_send, HYE_receive)
HYE_receive_1 = after_receive(TBH_send, HYE_receive)
result(TBH_send, HYE_receive)
```
Use the ***transfer*** function to create a transfer DataFrame. I name the transferred DataFrame according to the structure "SendingStore_ReceivingStore".

I reprocess the sent and received DataFrames using the functions ***after_send*** and ***after_receive***. I number them to distinguish them from the original DataFrames.

The ***result*** function helps me quickly check the number of transfers. If the number of transfers is reasonable, I save the DataFrames and repeat the process with other pairs of stores.
### Step 19: Create “Tranfering Order”
After completing the pairing of the stores, I synthesize the sending and receiving Data Frames into a unified Data Frame.

Finally, I will export the results into an Excel file containing the transfer order and send it to the stores to carry out the transfer.
```c
// Concatenate the send-receive DataFrames into a unified DataFrame
transfer_table = pd.DataFrame()
for send_store in stores:
    for receive_store in stores:
        if send_store != receive_store:
            name = f"{send_store}_{receive_store}"
            if name in locals():
                transfer_table = pd.concat([transfer_table, locals()[name]], ignore_index=True)
// Export result                
transfer_table.to_excel('Transfer Order.xlsx', index = None)
