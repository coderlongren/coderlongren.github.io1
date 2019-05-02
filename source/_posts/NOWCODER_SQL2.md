---
title: NOWCODER_SQL题目
date: 2018-4-23 23:18:40
categories:
	- MySQL
tags:
	- MySQL
---
摘要：NEWCODER SQL
<!-- more -->
1. 获取所有员工当前的manager，如果当前的manager是自己的话结果不显示，当前表示to_date='9999-01-01'。
结果第一列给出当前员工的emp_no,第二列给出其manager对应的manager_no。
CREATE TABLE `dept_emp` (
	`emp_no` int(11) NOT NULL,
	`dept_no` char(4) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`dept_no`));
	CREATE TABLE `dept_manager` (
	`dept_no` char(4) NOT NULL,
	`emp_no` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`dept_no`));

`select dept_emp.emp_no,dept_manager.emp_no as manager_no from dept_emp,dept_manager 
where dept_emp.dept_no = dept_manager.dept_no
and dept_emp.emp_no <> dept_manager.emp_no
and dept_emp.to_date='9999-01-01' and dept_manager.to_date='9999-01-01'`

2. 获取所有部门中当前员工薪水最高的相关信息，给出dept_no, emp_no以及其对应的salary
	CREATE TABLE `dept_emp` (
	`emp_no` int(11) NOT NULL,
	`dept_no` char(4) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`dept_no`));
	CREATE TABLE `salaries` (
	`emp_no` int(11) NOT NULL,
	`salary` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`from_date`));

`Group by语句应用`

`select d.dept_no, d.emp_no, max(s.salary) from dept_emp d, salaries s 
where d.emp_no = s.emp_no and s.to_date = '9999-01-01' and d.to_date = '9999-01-01' 
group by d.dept_no;`

3.查找employees表所有emp_no为奇数，且last_name不为Mary的员工信息，并按照hire_date逆序排列
	CREATE TABLE `employees` (
	`emp_no` int(11) NOT NULL,
	`birth_date` date NOT NULL,
	`first_name` varchar(14) NOT NULL,
	`last_name` varchar(16) NOT NULL,
	`gender` char(1) NOT NULL,
	`hire_date` date NOT NULL,
	PRIMARY KEY (`emp_no`));
`select * from employees e where e.emp_no % 2 = 1 and e.last_name <> 'Mary'
order by e.hire_date DESC;`

4. 统计出当前各个title类型对应的员工当前薪水对应的平均工资。结果给出title以及平均工资avg。
	CREATE TABLE `salaries` (
	`emp_no` int(11) NOT NULL,
	`salary` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`from_date`));
	CREATE TABLE IF NOT EXISTS "titles" (
	`emp_no` int(11) NOT NULL,
	`title` varchar(50) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date DEFAULT NULL);


`select t.title as title, avg(s.salary)as avg  from titles t, salaries s 
where t.emp_no = s.emp_no and t.to_date = '9999-01-01' and s.to_date = '9999-01-01'
group by t.title;`

5. 获取当前（to_date='9999-01-01'）薪水第二多的员工的emp_no以及其对应的薪水salary

`select s.emp_no, s.salary from salaries s 
where s.to_date = '9999-01-01' order by s.salary DESC
limit 1, 1;`


6. 查找所有员工的last_name和first_name以及对应的dept_name，也包括暂时没有分配部门的员工
	CREATE TABLE `departments` (
	`dept_no` char(4) NOT NULL,
	`dept_name` varchar(40) NOT NULL,
	PRIMARY KEY (`dept_no`));
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
1.第一次LEFT JOIN连接employees表与dept_emp表，得到所有员工的last_name和first_name以及对应的dept_no，也包括暂时没有分配部门的员工
2. 第二次LEFT JOIN连接上表与departments表，即连接dept_no与dept_name，得到所有员工的last_name和first_name以及对应的dept_name，也包括暂时没有分配部门的员工 


`select e.last_name, e.first_name ,de.dept_name from 
employees e left join dept_emp dp on e.emp_no = dp.emp_no
left join departments de on de.dept_no = dp.dept_no;`

7. 查找员工编号emp_now为10001其自入职以来的薪水salary涨幅值growth
	CREATE TABLE `salaries` (
	`emp_no` int(11) NOT NULL,
	`salary` int(11) NOT NULL,
	`from_date` date NOT NULL,
	`to_date` date NOT NULL,
	PRIMARY KEY (`emp_no`,`from_date`));

`select ((select salary from salaries where emp_no = 10001 order by to_date DESC limit 0,1) - (select salary from salaries where emp_no = 10001 order by to_date limit 0,1))
as growth;`

8. 