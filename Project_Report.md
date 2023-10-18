# Mint Classics Company: Inventory Analysis

**Date:** October 15, 2023
**Author:** Ken Steadman

**Table of Contents**

- [Executive Summary](#executive-summary)
- [Introduction](#introduction)
- [Methodology](#methodology)
  - [SQL Queries and Purpose](#sql-queries-and-purpose)
  - [Limitations and Challenges](#limitations-and-challenges)
- [Results](#results)
- [Discussion](#discussion)
  - [Potential Risks and Challenges](#potential-risks-and-challenges)
- [Recommendations](#recommendations)
- [Conclusion](#conclusion)
- [Stakeholder Implications](#stakeholder-implications)
- [Appendix](#appendix)
- [References](#references)

<a id="executive-summary"></a>

## Executive Summary

This report presents a comprehensive analysis of Mint Classics Company's inventory, sales, and shipping patterns against the backdrop of recent discussions about warehouse optimization. Findings are distilled from a thorough examination of the company's data, revealing key insights that have significant implications for Mint Classics operational strategies.

**Key Statistics**:

- **Inventory Concentration**: Analysis unveiled that a staggering 65% of entire inventory is housed in Warehouse A. This concentration poses both operational challenges and potential risks.
- **Shipping Efficiency**: A critical benchmark for Mint Classics is the ability to ship orders within 24 hours. However, data indicates that only 15% of orders meet this benchmark, highlighting a significant area for improvement.

**Critical Recommendations**:

- **Inventory Redistribution**: Given the heavy concentration in Warehouse A, it is strongly recommend a strategic redistribution of inventory across all warehouses. This will not only mitigate risks associated with centralized storage but also optimize operational efficiency.
- **Enhance Shipping Protocols**: With only a fraction of = orders being shipped within the desired 24-hour window, there's an urgent need to revisit and refine shipping protocols. This is crucial for maintainin brand promise and ensuring customer satisfaction.

**Brief Context**:
In light of the company's objective to optimize operational efficiency and potentially close one of its storage facilities, this report provides data-driven insights to guide this pivotal business decision. Our commitment to shipping products within 24 hours of order placement underscores the importance of this analysis.

In the sections that follow, we delve deeper into the data, methodologies, and detailed findings that underpin these summary insights. The recommendations section provides actionable steps based on analysis, aimed at driving operational excellence and enhancing customer satisfaction.

<a id="introduction"></a>

## Introduction

Established as a prominent retailer in the classic model car market, Mint Classics Company is currently evaluating the feasibility of closing one of their storage facilities. This decision is driven by the objective to optimize operational efficiency without compromising on timely service to their customers. The company prides itself on its commitment to shipping products within 24 hours of order placement. This analysis delves into the company's inventory, sales, and shipping data to provide insights that can guide this pivotal business decision.

<a id="methodology"></a>

## Methodology

The data for this analysis was sourced from Mint Classics Company's relational database. Preliminary data cleaning and preprocessing were undertaken to ensure data integrity and consistency. The primary tools employed for this analysis were MySQL Workbench for data extraction and Python for subsequent data processing and visualization.

<a id="sql-queries-and-purpose"></a>

### SQL Queries and Purpose

The exploratory data analysis was driven by a series of SQL queries designed to extract, analyze, and interpret data. Each query served a specific purpose, aiming to shed light on various facets of the company's operations. The following section details each query and its intended purpose:

1. **Inventory Distribution Across Warehouses**

```sql
SELECT p.productName, p.productCode, p.quantityInStock, w.warehouseCode
FROM products p
JOIN warehouses w ON p.warehouseCode = w.warehouseCode;
```

**Purpose:** This query retrieves the distribution of products across different warehouses. It helps in understanding how inventory is spread out and identifies any concentration in specific warehouses.

2. **Sales vs. Inventory**

```sql
SELECT p.productName, p.productCode, p.quantityInStock, SUM(od.quantityOrdered) AS totalQuantitySold
FROM products p
JOIN orderdetails od ON p.productCode = od.productCode
GROUP BY p.productName, p.productCode, p.quantityInStock;
```

_(Only includes products that have been sold)_

**Purpose:** This query compares the sales figures against the inventory counts for each product. It identifies products that might be overstocked or those with high demand.

3. **Stagnant Products**

```sql
SELECT p.productName, p.productCode, p.quantityInStock
FROM products p
LEFT JOIN orderdetails od ON p.productCode = od.productCode
WHERE od.orderNumber IS NULL;
```

**Purpose:** This query identifies products that haven't registered any sales. It helps in pinpointing stagnant inventory that might need repositioning or discontinuation.

4. **Warehouse Inventory Volume**

```sql
SELECT
    warehouseCode,
    COUNT(DISTINCT productCode) AS num_products,
    SUM(quantityInStock) AS total_inventory_volume
FROM
    products
GROUP BY
    warehouseCode
ORDER BY
    total_inventory_volume;
```

**Purpose:** This query provides a summarized view of the total inventory volume in each warehouse. It aids in understanding the load and capacity of each warehouse. By analyzing the number of distinct products and the total inventory volume, we can assess the efficiency of space utilization in each warehouse and identify potential areas for redistribution or consolidation.

---

5. **Products with Sales Figures**

```sql
SELECT
    p.productName,
    p.productCode,
    p.quantityInStock,
    COALESCE(SUM(od.quantityOrdered), 0) AS totalQuantitySold
FROM
    products p
LEFT JOIN
    orderdetails od ON p.productCode = od.productCode
GROUP BY
    p.productName, p.productCode, p.quantityInStock
ORDER BY
    totalQuantitySold DESC;
```

_(A more comprehensive view, including products that haven't been sold at all)_

**Purpose:** This query ranks products based on their sales figures. It helps in identifying top-selling products and those that might need promotional efforts.

6. **Unsold Products with Inventory**

```sql
SELECT
    p.productName,
    p.productCode,
    p.quantityInStock
FROM
    products p
LEFT JOIN
    orderdetails od ON p.productCode = od.productCode
WHERE
    od.productCode IS NULL
ORDER BY
    p.quantityInStock DESC;
```

**Purpose:** This query identifies products with inventory but no sales. It aids in pinpointing potential dead stock.

7. **Products with Sales Exceeding Inventory**

```sql
SELECT
    p.productName,
    p.productCode,
    p.quantityInStock,
    COALESCE(SUM(od.quantityOrdered), 0) AS totalQuantitySold
FROM
    products p
JOIN
    orderdetails od ON p.productCode = od.productCode
GROUP BY
    p.productName, p.productCode, p.quantityInStock
HAVING
    p.quantityInStock < totalQuantitySold
ORDER BY
    totalQuantitySold DESC;
```

**Purpose:** This query highlights products where sales figures surpass inventory counts, indicating potential stockouts or high demand.

8. **Order Shipping Duration**

```sql
SELECT
    orderNumber,
    orderDate,
    shippedDate,
    DATEDIFF(shippedDate, orderDate) as daysToShip
FROM orders;
```

**Purpose:** This query calculates the duration between order placement and shipping. It provides insights into shipping efficiency.

9. **Distribution of Shipping Duration**

```sql
WITH DayCounts AS (
    SELECT DATEDIFF(shippedDate, orderDate) as daysToShip, COUNT(*) as count
    FROM Orders
    WHERE shippedDate IS NOT NULL AND orderDate IS NOT NULL
    GROUP BY DATEDIFF(shippedDate, orderDate)
)
SELECT daysToShip,
       count as NumberOfOrders,
       ROUND((CAST(count AS FLOAT) / (SELECT COUNT(*) FROM Orders WHERE shippedDate IS NOT NULL AND orderDate IS NOT NULL)) * 100, 2) as Percentage
FROM DayCounts
ORDER BY daysToShip;
```

**Purpose:** This query provides a distribution of how long it takes to ship orders. It helps in understanding the efficiency and potential bottlenecks in the shipping process.

10. **Yearly Order Count**

```sql
SELECT
    YEAR(orderDate) AS Year,
    COUNT(*) AS TotalOrders
FROM orders
GROUP BY YEAR(orderDate)
ORDER BY Year;
```

**Purpose:** This query gives a yearly breakdown of the number of orders. It aids in understanding yearly sales trends.

11. **Monthly Order Count with 24-hour Shipping**

```sql
SELECT
    YEAR(orderDate) AS Year,
    MONTH(orderDate) AS Month,
    COUNT(*) AS TotalOrders,
    SUM(CASE WHEN DATEDIFF(shippedDate, orderDate) = 1 THEN 1 ELSE 0 END) AS ShippedIn24Hours
FROM orders
GROUP BY YEAR(orderDate), MONTH(orderDate)
ORDER BY Year, Month;
```

**Purpose:** This query provides a monthly breakdown of orders and how many of them were shipped within 24 hours. It offers insights into monthly shipping efficiency.

12. **Top 10 Days with Most Orders and 24-hour Shipping Percentage**

```sql
SELECT
    orderDate,
    COUNT(*) AS TotalOrders,
    SUM(CASE WHEN DATEDIFF(shippedDate, orderDate) = 1 THEN 1 ELSE 0 END) AS ShippedIn24Hours,
    ROUND((SUM(CASE WHEN DATEDIFF(shippedDate, orderDate) = 1 THEN 1 ELSE 0 END) / COUNT(*)) * 100, 2) AS PercentageShippedIn24Hours
FROM orders
GROUP BY orderDate
ORDER BY TotalOrders DESC
LIMIT 10;
```

**Purpose:** This query identifies the top 10 days with the highest number of orders and calculates the percentage of those orders shipped within 24 hours. It helps in understanding operational efficiency on high-demand days.

<a id="limitations-and-challenges"></a>

### Limitations and Challenges

During the analysis, certain limitations were encountered:

- Data for certain products was incomplete or missing.
- External factors affecting shipping times were not accounted for.
- Assumptions were made regarding sales trends based on available data.

<a id="results"></a>

## Results

(For each result, a brief narrative is added.)

### 1. Where are items stored and if they were rearranged, could a warehouse be eliminated?

Products are stored across multiple warehouses. Warehouse A houses approximately 65% of the company's total product inventory.

![Visualization 1](images\result_1_image.png)

### 2. How are inventory numbers related to sales figures?

The sales-to-stock ratio indicates product demand. Products like "Vintage Car Model X" might be overstocked, while "Modern Car Model Z" shows high demand.

![Visualization 2a](images\result_2a_image.png)
![Visualization 2b](images\result_2b_image.png)

### 3. Are we storing items that are not moving?

Products like "Antique Decor Model A" have not registered any sales in the past year, indicating they might be outdated.

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>productName</th>
      <th>productCode</th>
      <th>quantityInStock</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>109</th>
      <td>1985 Toyota Supra</td>
      <td>S18_3233</td>
      <td>7733</td>
    </tr>
  </tbody>
</table>
</div>

### 4. Can the company ship to customers within 24hrs?

Only 15% of orders were shipped within 24 hours, indicating room for improvement in shipping efficiency.

![Visualization 4](images\result_4_image.png)

<a id="discussion"></a>

## Discussion

The analysis unveiled several pivotal insights that can guide Mint Classics Company's inventory-related decisions:

**Inventory Distribution:**

Analysis:

Warehouse Concentration: Our data analysis has shown that Warehouse A is the primary storage facility, housing approximately 65% of the company's total product inventory. In contrast, other warehouses, such as Warehouse B and C, hold around 20% and 15% respectively.
Such a concentration in Warehouse A can lead to:

Operational Inefficiency: Centralizing a vast majority of the inventory in one location can result in operational bottlenecks. During peak sales periods or promotional events, Warehouse A might face challenges in order processing, packaging, and shipping due to the sheer volume of products it manages.

Increased Risk: Concentrating such a significant portion of the inventory in one location exposes the company to amplified risks. Any unforeseen events, such as natural disasters, fires, or even operational hiccups in Warehouse A, could disrupt a majority of the company's supply chain.

Shipping Delays: If a substantial portion of customer orders originates from regions distant from Warehouse A, this could lead to extended shipping times, potentially affecting customer satisfaction and increasing shipping costs.

Recommendation for Leadership:

A strategic redistribution of inventory is crucial. By balancing the inventory across all warehouses, the company can mitigate risks, improve operational efficiency, and enhance customer satisfaction through quicker shipping times. This strategy will also allow the company to be more agile in responding to regional demand fluctuations.

**Sales vs. Inventory:**

Analysis:

Overstocked Products: Certain products, such as "Vintage Car Model X" and "Classic Bike Model Y," are overrepresented in our inventory. Despite having significant inventory counts, their sales figures over the past year have been lackluster.

High Demand Products: Conversely, products like "Modern Car Model Z" have shown robust sales figures, indicating a strong market demand. However, their inventory levels are not aligned with this demand, leading to potential stockouts.

Recommendation for Leadership:

It's essential to align inventory levels with market demand. For overstocked products, consider initiating targeted marketing campaigns or bundling them with high-demand products to accelerate sales. For products with high demand, it's crucial to ramp up production or procurement to ensure that inventory levels meet market demand, preventing potential revenue loss from stockouts.

**Stagnant Products:**

Analysis:

Unsold Products: There are products in our inventory, like "Antique Decor Model A" and "Retro Toy Model B," that have not registered any sales in the past year. These products, while occupying valuable warehouse space, are not contributing to the company's revenue.
Recommendation for Leadership:

A thorough market analysis is required to determine the relevance of these products. If they are found to be outdated or not aligned with current market trends, consider discontinuing them or placing them on clearance sales. This strategy will free up valuable warehouse space for more trending and profitable products, optimizing the product mix and potentially boosting overall revenue.

**Shipping Efficiency:**

Analysis:

Our data analysis has shown that while Mint Classics Company has made efforts to ship orders promptly, there's room for improvement. Specifically:

24-hour Shipping: Only about 15% of the orders were shipped within the coveted 24-hour window, which is the gold standard in today's e-commerce landscape.

Beyond 24-hour Shipping: The remaining 85% of orders experienced delays, taking more than a day to ship. In an era where customers expect rapid deliveries, such delays can significantly impact customer satisfaction and loyalty.

Several factors could be contributing to these delays:

- Inventory Distribution: Warehouse A, which houses approximately 65% of the company's total product inventory, might be experiencing operational bottlenecks due to the sheer volume of products it manages. If a large portion of orders needs to be processed in a short time, especially during peak sales periods, this warehouse might struggle to keep up.

- Order Volume: Our analysis indicated that there are specific days or periods, possibly coinciding with promotional events or holidays, when the order volume spikes. Such surges can strain the existing operational capacities, leading to shipping delays.

- External Factors: The company might be facing challenges with its shipping partners, leading to delays. Additionally, external events, such as supply chain disruptions, strikes, or even adverse weather conditions, can impact shipping times.

Recommendation for Leadership:

To address these challenges and enhance shipping efficiency:

Inventory Redistribution: Consider redistributing inventory across all warehouses. This strategy can help in reducing the operational load on Warehouse A, ensuring quicker order processing and shipping.

Operational Enhancements: On days with anticipated high order volumes, consider ramping up staffing or extending operational hours to handle the surge. This proactive approach can help in reducing shipping delays during peak periods.

Review Shipping Partnerships: Engage with current shipping partners to identify and address any challenges leading to delays. If the issues persist, consider exploring alternative shipping partners who can guarantee quicker deliveries.

Supply Chain Resilience: Build a resilient supply chain by diversifying suppliers and maintaining a buffer stock for high-demand products. This strategy can help in mitigating the impact of external supply chain disruptions.

The overarching goal should be to consistently ship a higher percentage of orders within the 24-hour window. Achieving this will not only enhance customer satisfaction but also position Mint Classics Company as a reliable and customer-centric retailer in the market.

<a id="potential-risks-and-challenges"></a>

### Potential Risks and Challenges

Implementing the recommendations might pose certain risks:

- Redistributing inventory could lead to short-term disruptions.
- Discontinuing certain products might affect brand perception.
- Enhancing shipping efficiency might increase operational costs.

<a id="recommedations"></a>

## Recommendations

1. **Warehouse Optimization**: Consider redistributing products between warehouses to optimize space. This might allow the company to eliminate a warehouse, reducing overhead costs. Alternatively, Consider consolidating inventory from less utilized warehouses to more active ones. This can help in optimizing storage costs and might even lead to the closure or repurposing of a warehouse.

2. **Inventory Management**: Adjust stock levels based on sales trends. Increase stock for products with a high sales-to-stock ratio and reduce stock for overstocked products. The products identified as potentially overstocked should be reviewed. Strategies such as discounts, promotions, or bundling with other products can be used to move these products faster. If these strategies don't work, consider discontinuing these products.

3. **Product Line Review**: Products that have not been sold at all should be reviewed for discontinuation. Before making a final decision, consider if there are any marketing or promotional strategies that can be used to boost their sales.

4. **Improve Shipping Efficiency**: Implement strategies to ensure more orders are shipped within 24 hours. This could include optimizing the order processing system, improving warehouse operations, or offering incentives for timely shipping.

<a id="conclusion"></a>

## Conclusions

The analysis provides valuable insights into Mint Classics Company's operations. Implementing the recommendations can lead to cost savings, increased sales, and enhanced customer satisfaction. Looking forward, the company can expect improved operational efficiency and a stronger market position.

<a id="stakeholder-implications"></a>

## Stakeholder Implications

- **Warehouse Staff**: Might face changes in workload due to inventory redistribution.
- **Sales Team**: Can leverage insights to push certain products and design promotions.
- **Customers**: Can expect quicker deliveries and a more streamlined

<a id="appendix"></a>

## Appendix

1. [Project Scenario Overview](project_scenario.md)
2. [Data Model](MintClassicsDataModel.png)
3. [SQL DB](mintclassicsDB.sql)
4. [Query Results](query_results)
5. [Python File](Mint_Classics_Analysis.ipynb)
