---
title: 牛客网SQL题目答案
date: 2018-4-20 21:18:40
categories:
	- MySQL
tags:
	- MySQL
---
摘要： NEWCODER SQL
<!-- more -->
牛客SQL一直用的就是那几张表
1. 查找最晚入职员工的所有信息
	CREATE TABLE `employees` (
	`emp_no` int(11) NOT NULL,
	`birth_date` date NOT NULL,
	`first_name` varchar(14) NOT NULL,
	`last_name` varchar(16) NOT NULL,
	`gender` char(1) NOT NULL,
	`hire_date` date NOT NULL,
	PRIMARY KEY (`emp_no`));


sql:  
`
select * from employees where hire_date = (select max(hire_date) from employees);`

2. 查找入职员工时间排名倒数第三的员工所有信息
	CREATE TABLE `employees` (
	`emp_no` int(11) NOT NULL,
	`birth_date` date NOT NULL,
	`first_name` varchar(14) NOT NULL,
	`last_name` varchar(16) NOT NULL,
	`gender` char(1) NOT NULL,
	`hire_date` date NOT NULL,
	PRIMARY KEY (`emp_no`));


`select * from employees where hire_date = (select hire_date from employees order by hire_date desc limit 2,1); `

3. 查找各个部门当前(to_date='9999-01-01')领导当前薪水详情以及其对应部门编号dept_no
	CREATE TABLE `dept_manager` (
	`dept_no` char(4) NOT NULL,
	`emp_no` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`dept_no`));
	CREATE TABLE `salaries` (
	`emp_no` int(11) NOT NULL,
	`salary` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`from_date`));

`select s.*,d.dept_no from salaries s ,dept_manager d  where s.emp_no = d.emp_no and d.to_date='9999-01-01' and s.to_date='9999-01-01';
`
4. 查找所有已经分配部门的员工的last_name和first_name
	CREATE TABLE `dept_emp` (
	`emp_no` int(11) NOT NULL,
	`dept_no` char(4) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`dept_no`));
	CREATE TABLE `employees` (
	`emp_no` int(11) NOT NULL,
	`birth_date` date NOT NULL,
	`first_name` varchar(14) NOT NULL,
	`last_name` varchar(16) NOT NULL,
	`gender` char(1) NOT NULL,
	`hire_date` date NOT NULL,
	PRIMARY KEY (`emp_no`));

`select e.last_name, e.first_name,d.dept_no from dept_emp d,employees e where d.emp_no = e.emp_no;`

5. 查找所有员工的last_name和first_name以及对应部门编号dept_no，也包括展示没有分配具体部门的员工
	CREATE TABLE `dept_emp` (
	`emp_no` int(11) NOT NULL,
	`dept_no` char(4) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`dept_no`));
	CREATE TABLE `employees` (
	`emp_no` int(11) NOT NULL,
	`birth_date` date NOT NULL,
	`first_name` varchar(14) NOT NULL,
	`last_name` varchar(16) NOT NULL,
	`gender` char(1) NOT NULL,
	`hire_date` date NOT NULL,
	PRIMARY KEY (`emp_no`));

`select e.last_name, e.first_name ,d.dept_no from  employees e left join  dept_emp d on e.emp_no = d.emp_no;`

6. 查找所有员工入职时候的薪水情况，给出emp_no以及salary， 并按照emp_no进行逆序
	CREATE TABLE `employees` (
	`emp_no` int(11) NOT NULL,
	`birth_date` date NOT NULL,
	`first_name` varchar(14) NOT NULL,
	`last_name` varchar(16) NOT NULL,
	`gender` char(1) NOT NULL,
	`hire_date` date NOT NULL,
	PRIMARY KEY (`emp_no`));
	CREATE TABLE `salaries` (
	`emp_no` int(11) NOT NULL,
	`salary` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`from_date`));

`select e.emp_no, s.salary from employees e, salaries s   
where e.emp_no  = s.emp_no and e.hire_date = s.from_date 
order by e.emp_no DESC ;`

7. 查找薪水涨幅超过15次的员工号emp_no以及其对应的涨幅次数t
	CREATE TABLE `salaries` (
	`emp_no` int(11) NOT NULL,
	`salary` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`from_date`));

`select salaries.emp_no, count(salaries.emp_no) t  from salaries
group by salaries.emp_no having t > 15;`

8. 找出所有员工当前(to_date='9999-01-01')具体的薪水salary情况，对于相同的薪水只显示一次,并按照逆序显示
	CREATE TABLE `salaries` (
	`emp_no` int(11) NOT NULL,
	`salary` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`from_date`));

`select  distinct salaries.salary  from salaries where salaries.to_date = '9999-01-01'
order by salaries.salary DESC`

9. 获取所有部门当前manager的当前薪水情况，给出dept_no, emp_no以及salary，当前表示to_date='9999-01-01'
	CREATE TABLE `dept_manager` (
	`dept_no` char(4) NOT NULL,
	`emp_no` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`dept_no`));
	CREATE TABLE `salaries` (
	`emp_no` int(11) NOT NULL,
	`salary` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`from_date`));

`
select dept_manager.dept_no, dept_manager.emp_no, salaries.salary from  salaries , dept_manager 
where salaries.to_date='9999-01-01' and dept_manager.to_date='9999-01-01' and dept_manager.emp_no = salaries.emp_no;`

10. 获取所有非manager的员工emp_no
	CREATE TABLE `dept_manager` (
	`dept_no` char(4) NOT NULL,
	`emp_no` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`dept_no`));
	CREATE TABLE `employees` (
	`emp_no` int(11) NOT NULL,
	`birth_date` date NOT NULL,
	`first_name` varchar(14) NOT NULL,
	`last_name` varchar(16) NOT NULL,
	`gender` char(1) NOT NULL,
	`hire_date` date NOT NULL,
	PRIMARY KEY (`emp_no`));

明显可以使用很多方法来写SQl 
* 使用NOT IN选出在employees但不在dept_manager中的emp_no记录 
`SELECT emp_no FROM employees
WHERE emp_no NOT IN (SELECT emp_no FROM dept_manager)`
* 先使用LEFT JOIN连接两张表，再从此表中选出dept_no值为NULL对应的emp_no记录 

`select e.emp_no from employees e left join dept_manager d
on e.emp_no = d.emp_no where d.dept_no is null;`
