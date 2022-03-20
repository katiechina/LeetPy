## 视图
视图不是表，视图是虚表，视图依赖于表
修改视图也会修改对应的表，一般不会去修改视图
`CREATE VIEW <视图名称>(<列名1>,<列名2>,...) AS <SELECT语句>`
`ALTER VIEW <视图名> AS <SELECT语句>`

#### 3.1
`CREATE VIEW ViewPractice5_1  
AS 
SELECT product_name ,sale_price ,regist_date
FROM product
where sale_price>=1000 and regist_date='2009-09-20'`


#### 3.2
会失败，因为在尝试向product中加入新行，但是有的行是非NULL，又没有给到数据，插入失败

#### 3.3
`SELECT product_id,
       product_name,
       sale_price,
       sale_price,
       (SELECT AVG(sale_price)
          FROM product) AS avg_price
  FROM product;`

#### 3.4
` SELECT product_type, product_name, sale_price,
 	(SELECT AVG(sale_price)
   	FROM product AS p2
   	WHERE p1.product_type =p2.product_type
   	GROUP BY product_type)
  FROM product AS p1;`
