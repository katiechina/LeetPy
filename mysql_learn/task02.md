## 数据导入
本来应该是task1中完成数据导入，当时没有想要跟着操作，所有没有尝试导入。
这次导入时遇到不少问题，导入的正确方法：
1. 启动mysql服务端进程
> '''  /usr/local/mysql '''
> '''  sudo ./support-files/mysql.server start '''
2. DBeaver 创建默认localhost连接
> mysql driver 要选择8.0.x以上的版本，否则密码加密格式不同，不能识别
> mysql driver 的 properties 中 allowPublicKeyRetrieval 要设置为true，否则也会报错
3. 执行shop.sql之后要创建新的连接，数据库名填写shop。 新的数据库不在原localhost下

## 基础查询与排序
2.1
> select product_name,  regist_date from product 
where regist_date > '2009-04-28';

2.2
~~1) 价格为空的行 ~~
结果为空
~~2) 价格非空的行 ~~
结果为空
~~3) 价格非空的行 ~~
执行报错，不用用大于号比较NULL

2.3
> SELECT product_name, sale_price, purchase_price
  FROM product
 WHERE sale_price - purchase_price >= 500;
 
> SELECT product_name, sale_price, purchase_price
  FROM product
 WHERE sale_price >= purchase_price + 500;
 
> SELECT product_name, sale_price, purchase_price
  FROM product
 WHERE sale_price -500 >= purchase_price;

2.4
> SELECT product_name, product_type, (sale_price * 0.9 - purchase_price) AS profit
  FROM product
where profit >= 100 and (product_type= '办公用品' or product_type= '厨房用具' );


2.5
- where 子语句应该在group by 之前
- sum 的列必须时数值列，不能时字符串列


2.6
> SELECT product_type, sum(sale_price) as 'sum', sum(purchase_price) as 'sum'
FROM product
group by product_type
having sum(sale_price) > sum(purchase_price)*1.5;

2.7
- regist_dates 时间倒序排列，但是null在最前面
> SELECT *
FROM product
order by - regist_date; 
