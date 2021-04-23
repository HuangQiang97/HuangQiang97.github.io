# Matlab

[toc]

## 1:Basic

```matlab

%% 
% 获得函数帮助说明
help randi

%%
result=[log(2.7),log2(2),log10(10),exp(1)];
disp(result)
% 2*3:[0,1]间均匀分布矩阵
rand(2,3)
% 3*3:[0,1]间均匀分布矩阵
rand(3)
% randi([min,max],m,n):m*n均匀分布随机数矩阵，含两端
choic=randi([0,2],1,4)
% 复数
2+i*4
%% 
% 运算优先级：() > ^ > * / > + -
% 变量使用前无需申明
% 数字默认double 类型
% i,j 指向复数的虚部
i
j
% 无穷大
Inf
% log(0)
eps
% not a number
NaN
pi
% 指向最近一个匿名结果
ans

% 变量优先级：变量>内建函数>子函数
cos='string'
cos(3) %'r'
%%
% 格式化输出,四舍五入式
% format bank:2位小数， short:4位小数，long :15位小数，shortE:short 的科学计数法形式，longE:long的科学计数法形式
% format hex:16进制，rat:分数形式
format bank
a=12/6.5

%%
a=[1,2,3];% 行向量
b=[1;2;3];%列向量
a*b% 矩阵乘法

%%
% matlab 下标都是从1 开始

a=[1 2 3; 4 5 6; 7 8 9 ];
a(3) % 矩阵从左到右按列分组，每组从上到下依次从1开始编号
a([1 2; 3 4]) % 第1 2;3 4个元素构成新的数组
a(:) %按序号的矩阵所有元素列向量
a(2,3)% 数组下标访问
a([1 3],[2 3])% 第1 3行于第  2 3列相交处元素构成新的2*2矩阵

%% 
%向量
a=1:10
a=1:2:10
a=linspace(1,10,3)

%%
clc
a=magic(5)
a(1,:) %':'表示该列所有行，或者改行所有列
a(:,1)
a(:,1)=[] %[]表式删除改行或列，矩阵维度发生变化
a(1,:)=[]

%%
% 矩阵拼接
clc
a=randi(3,4)
b=randi(3,4)
c=[a b] %横向拼接a,b行数相同
d=[a;b] %纵向拼接a,b列数相同

%%
clc
a=randi([1,10],3,3)
b=randi([1,10],3,3)
c=2
a+a% + 运算，元素与元素相加
a+c 
a-a 
a-c
a*b  % * 矩阵乘法，.* 对应元素间相乘
a*c 
a.*a
a/c % a/b=>a乘以b的逆矩阵，a./b 对应元素相除，c.\a=>a./c , a.\b=>b./a
a/b 
a./b  
c.\a  
a.\b
a^2 % a^2=>a*a , .^ 每个元素做乘方运算
a.^2 
a' % 转置

%%
clc
eye(3) % 3*3单位矩阵
zeros(2,3) %2*3 全0阵
ones(2,3) %2*3 全1阵
diag([1:2:10]) %把向量转化为对角矩阵

%%
clc
a=randi([1,10],3,4)

max(a)  % 多维矩阵返回每列最大值的行向量，行/列向量返回向量的最大值，
max(max(a))
max(a,[],2) %返回a中每行元素最大值的列向量

min(a) %同上
min(min(a))
min(a,[],2)

sum(a)%同上
sum(sum(a))
sum(a,2)

mean(a)%同上
mean(mean(a))
mean(a,2)

sort(a) %每列排升序
sort(a,2) %每行排升序

sortrows(a) %按第一列大小拍升序，其余列元素同第第一列同步变化

size(a) %m*n
size(a,1) %m
size(a,2) %n

length(a) %列数

find(a==3) % 得到所有3所在的位置序号
```

## 2:Struct Programming

```matlab

% 结构化编程，
% %%：节标志，可以以节为单位运行。
%%
% rem(a,2)=>q%2,prod(1:n)=>1*2*3*...*n
% randi([min,max],m,n):m*n均匀分布随机数矩阵，含两端
% ';'不输出当前行结果
choic=randi([0,2],1,1);
switch choic
    case 0
        disp('0');
    case 1
        disp('1');
    case 2
        disp('2');
end

for i=1:10
    % [0,10]均分101份，含头和尾
    x=linspace(0,10,101);
    plot(x,sin(x+i));
    print(gcf,'-deps',strcat('plot',num2str(i),'.ps'));
end
%%
% 清空输出
clc ;
% 关闭所有子窗口
close all
i=1;
result=0;
while i<=999
    result=result+i;
    i=i+1;
end
disp("result:"+result)

%%
% a下标从1开始
a=zeros(1,5);
% start:step:end->含头尾[start,end]
for index=1:2:10
    a((index+1)/2)=2^index;
end
disp(a)

%%
clc
% 清空数据
clear a
% 开始计时 
tic
for i=1:2000
    for j=1:2000
        a(i,j)=i+j;
    end
end
% 结束计时并输出
toc

clear a
tic
% 预分配内存以获得更高的运算速度，之后给变量赋值时不用判断内存是否足够，不够就要从新分配内存
a=zeros(2000,2000);
for i=1:2000
    for j=1:2000
        a(i,j)=i+j;
    end
end
toc

%%
clear 
tic
a=[0 -1 4; 9 -14 25 ;-34 49 64 ;1 2 3];
%[列数，行数]
arraySize=size(a);
disp(a)
disp(arraySize)
b=zeros(arraySize);
for i=1:arraySize(1)
    for j=1:arraySize(2)
        if a(i,j)<0
            b(i,j)=0;
        else
            b(i,j)=a(i,j);
        end
    end
end
disp(b)
toc

%%
 for i=1:10000
    % 0~1 均匀分布随机数
     num=rand();
     if num>=0.999
        % num2str:123->'123';'123'+""->"123";"123"+1->"1231"
         disp(num2str(num)+" "+i);
         % 跳出循环
         break;
     end
 end

 %%
 % 可一次输入多组数据，同一个函数参数用[]
 % acc(12,343,4545,34,3) acc(43,354,555,5,45)
 % 结果变量同样数组返回
[a,f]=acc([12 43],[343 354],[4545 555],[34 5],[3 45]);
disp("a:\n"+a)
disp("f\n"+f)
%%
a=1;
T=2^0.5*pi*a/4;
% 函数句柄，注意'.'运算
% 单行定义，多参数，单一结果，隐式返回
f=@(x)2^0.5*a./2-(a./2)./cos(fix(x./T)*pi./2+pi./4-2^0.5*x./a) ;
x=0:0.005:4*T;
plot(x,f(x))
% 坐标轴参数
title('x-y');
ylim([0,1])
ylabel('y');
xlabel('x');
%%
% 输入数字，数组，函数名rand()
x=input('fuck me:\n');
num2str(x)
% 输入字符向量
x=input('fuck me:\n','s');
isempty(x)
%%
f2c()

```

```matlab
% acc函数名要和文件名相同
function [a,f] = acc(v_0,v_1,t_0,t_1,m)
    % 使用.运算，满足多组数据输入的运算
    % 返回的参数在函数声明时只是申明，在必要时要定义或单独分配内存
    a=(v_1-v_0)./(t_1-t_0);
    f=m.*a;
    % 以[a,f]形式返回结果，如果一次输入多组数据，单个变量本身就是一个数组。
end


```

```matlab

function [outputArg1,outputArg2] = funDefVar(name,age,gender)
    %获得函数基础信息
    outputarg1=name;
    outputarg2=age;
    % 当前.m文件名
    mfilename
    % 函数输入参数数目
    nargin
    % 函数输出参数数目
    nargout
end


```

```matlab

function [ ] = f2c()

    while 1
        f=input('f:\n');
        if isempty(f)
            break
        end
        c=(f-32)*5/9;
        disp("c:"+num2str(c))
    end

end


```

