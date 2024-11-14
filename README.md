# Fetch Take-Home Case Study  
### By Brandon Sung  
#### Date: 11/09/2024  

## SQL  
This assignment uses PySpark, my preferred language. It has better processing for large datasets and a simpler transition between Python and SQL.

---

## Problems with the Data  

To identify data quality issues, I took two approaches: **bottom-up** and **top-down**.

### Bottom-Up Approach  
For the bottom-up analysis, I looked for issues in the raw data:

- **Duplicate Barcodes in the Brand Table**: There are duplicate barcodes in the brand table - this becomes imporant in the top-down analysis.
- **Duplication in the User Table**: Significant number of duplicates were found in the user table.
- **Null `lastLogin` for Active Users**: Some users have a `null` value in the `lastLogin` field, even though the user is active.
- **`rewardsReceiptItemList` Holds an Array**: This field holds an array used to derive the items table.  
  We need to create a new **items** table to:
  1. Explode the line items so they can be properly structured.
  2. Bridge the receipt table to the brands table using the barcode (unconclusive).
- **Unstructured Items Table**: The items table derived from `rewardsReceiptItemList` is very unstructured. There are a lot of columns with less than a 1% non-null rate. I decided to drop anything with a 10% non-null rate.
- **Null Prices and Quantities**: Some prices and item quantities are null, which leads to inaccurate spend calculations.
- **Discrepancy Between `totalSpent` and Item Spend**: We need to check the `totalSpent` field from receipts against the sum of spend for the items. There is a 64% MAPE, which implies that an assumption I made is wrong. However, this could also be due to other reasons, like missing prices for items, or OCR issues.
- **Large Receipts and Suspicious Data**: Some receipts are showing totals of $20k+ (when summing the spend for items) and over 600 quantities. These users must either be "Pro" users, committing fraud, or an error with the data system.

### Top-Down Approach  
For the top-down analysis, I looked at the results of the ER diagram and reviewed the output of the queries:

- **Weak Join Between Items and Brands**:  
  The join between the items and brands tables using the barcode field is weak:
  - Less than ~1% of items are linked to a brand via barcode.
  - About 55% of barcodes are `NULL`, and over 100 items have the barcode `4011` (items not found).
  - There are many test brands in the brand table (>30%), which could be due to the sampling methodology.
  - There are also duplicates for barcodes in the brands table, which causes issues downstream. I question if the barcode is the best or correct solution to the ER diagram. 

- **Conclusion**:  
  The barcode join is problematic. Resolving this issue depends on understanding the business use case. I would like to know how the brand table is created and whether a barcode can be linked to multiple brands. From a database standpoint, we could consider using other fields that allow this join to work.  
  - Can we use `brandCode` and `partnerItemId` (which has a 100% fill rate)?  
  - Alternatively, we could create an intersection table to resolve the many-to-many relationship.
  
However, I cannot make a final decision without a larger dataset. This sample might not include the barcodes that actually match the items, so we may need more data to make a proper assessment.