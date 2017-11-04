# dplyr

#数据概览

library(dplyr)

library(nycflights13)

dim(flights)

#将数据转换为更好处理的tibble表格形式

flights<-tbl_df(flights)

head(flights)


#数据筛选-filter/slice

filter(flights,month==1,day==1)

#选取10行观测

slice(flights,1:10)

#同样功能R基础用法为：

flights[which(flights$month==1&flights$day==1),]

flights[1:10,]



#数据重排-arrange

arrange(flights,year,month,day)

#按照某个变量降序排列

arrange(flights,desc(arr_delay))

#相应功能的R基础用法：

flights[order(flights$year,flights$month,flights$day),]

flights[order(flights$arr_delay,decreasing=TRUE),]





#数据选择-select

select(flights,year,month,day)

#数据选择的R基础用法

subset(flights,select=year:day)



#数据变量重命名-rename

rename(flights,tail_num=tailnum)

#变量重命名的R基础用法

tail_num<-flights$tailnum





#抽取变量中单一观测值-distinct

distinct(flights,tailnum)

#相应功能的R基础用法

unique(flights$tailnum)





#数据变形-mutate

mutate(flights,gain=arr_delay-dep_delay,speed=distance/air_time*60)

#相应功能的R基础用法

transform(flights,gain=arr_delay-dep_delay,speed=distance/air_time*60)

##mutate与transform的区别在于mutate可以即时调用刚刚生成的变量名，但transform函数则不行。

#不过也可以通过transmute函数来仅显示创建的变量数据。

transmute(flights,gain=arr_delay-dep_delay,gain_per_hour=gain/(air_time/60))





#数据归总summarise与随机抽样sample_n/sample_frac

summarise(flights,delay=mean(dep_delay,na.rm=TRUE))

#按个数抽样

sample_n(flights,10)

#按比例抽样

sample_frac(flights,0.01)

##dplyr进行数据处理的几个主要“动词”函数的规律了：

#● 第一个参数一定是需要处理的数据集

#● 第二个参数则形容了我们将要对数据进行怎样的处理，相较于基础的数据操作而言，dplyr函数无需使用$符号。

#● dplyr函数的结果以一个新的数据框形式呈现。





#数据的分组处理-group_by

destinations<-group_by(flights,dest)

summarise(destinations,planes=n_distinct(tailnum),

flights=n())

##分组处理结合汇总函数：

daily<-group_by(flights,year,month,day)



(per_day<-summarise(daily,flights=n()))



(per_month<-summarise(per_day,flights=sum(flights)))



(per_year<-summarise(per_month,flights=sum(flights)))





#管道函数%>%

##dplyr在处理过程必须及时对结果进行保存，所以在进行数据处理时需一步步赋值处理：

a1<-group_by(flights,year,month,day)

a2<-select(a1,arr_delay,dep_delay)

a3<-summarise(a2,

arr=mean(arr_delay,na.rm=TRUE),

dep=mean(dep_delay,na.rm=TRUE))

(a4<-filter(a3,arr>30|dep>30))

###当然，若果你不想进行即时保存，也可以通过对函数的嵌套处理：

filter(

summarise(

select(

group_by(flights,year,month,day),

arr_delay,dep_delay


),
    arr=mean(arr_delay,na.rm=TRUE),

dep=mean(dep_delay,na.rm=TRUE)

),

arr>30|dep>30

)

###上面的嵌套读起来太费劲，dplyr为大家提供了管道操作符号%>%来传递每一层的运行结果：

flights %>%

group_by(year,month,day) %>%

select(arr_delay,dep_delay) %>%

summarise(

arr=mean(arr_delay,na.rm=TRUE),

dep=mean(dep_delay,na.rm=TRUE)

) %>%

filter(arr>30|dep>30)




