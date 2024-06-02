# Inventory Balancing
## Introduction
In this project, I use Python to coordinate inventory at stores within the system. By using historical sales data and current inventory data, I perform analysis and evaluation to ensure products are getting to the right stores where they are in demand, minimizing stockouts, and preventing overcrowding at stores with excessive inventory.
## Business context
The data used pertains to fashion items at a retailer with distribution stores across the country. Products are distributed to these stores for storage, display, and sales.
For fashion items, having a variety of product sizes is crucial to accommodate the diverse needs of customers. The absence of sizes within the same product can potentially hinder product sales, as customers may struggle to find the right size. Therefore, ensuring that stores have adequate stock of products in various sizes is essential to facilitate faster sales, particularly towards the end of the season.

![sizes](https://github.com/ducpham131/Inventory-Balancing/assets/169105426/9355a49b-8830-4058-adc6-b66e86030907)
<img src="https://github.com/ducpham131/Inventory-Balancing/assets/169105426/9355a49b-8830-4058-adc6-b66e86030907" alt="..." width="250" />
Based on the inventory data of the stores in the system, I was tasked with creating transfer orders to balance the products across the stores. The objective of this activity is to leverage the available sizes in stock at certain stores to fill size gaps in others. Additionally, the aim is to gather products with low and sparse inventory from various stores to the best-selling stores. This ensures there are enough items on display shelves and increases the likelihood of selling out by the end of season.
This task consumes a lot of time and operations when done on spreadsheets when dealing with large amounts of data. So I replaced the spreadsheet with Python to save time and improve performance.
## Datasets
![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/718ac802-254c-4789-ad12-a0881c781f94/a09e6675-21f9-47e7-b2fd-73b832668775/Untitled.png)
### Explain the conventions
#### Store conventions
Stores in the dataset are represented as 3-character codes of cities/provinces.
For excample: 
- DNA: Da Nang city
- HCM: Ho Chi Minh city
#### product_id conventions
The "product_id" consists of 9 characters and contains encrypted information about the product. This information can be deciphered to extract additional details about the product, such as the "Product group," "Product line," "Product number," and "Size.”
![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/718ac802-254c-4789-ad12-a0881c781f94/32da113a-199b-46b7-b9e1-37c06b208926/Untitled.png)
- Product Rroup: denoted by the first 2 numbers of product_id. For example: 40 - Office wear Product group.
- Product line: The third character in “product_id” represents the product line. In this dataset, there are 2 product lines: women's clothes corresponding to the letter W and men's clothes corresponding to the letter M.
- Product number: The next 4 digits are the product code used to distinguish products in the same group.
- Size: The last 2 characters represent the size of the product. There are 5 sizes “Small”, “Medium”, “Large”, “X-Large”, “XX-Large” in the order of “S1”, “S2”, “S3”, “S4”, “S5”. For example: S1 - size S
