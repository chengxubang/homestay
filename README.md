基于ssm框架实现民宿管理系统


项目介绍
----
>  本系统使用Spring+SpringMVC+MyBatis架构，数据库使用MySQL,前端页面使用jsp，专为名宿经营打造的管理系统，主要给民宿经营人员、门店前台人员等使用。通过本系统，民宿经营者可以方便的管理自己的房间、房型，灵活的制定的价格方案，直观查看民宿经营的核心数据，合理分配科学管理民宿内贩售的其他商品。总之，能够帮助民宿主提升民宿管理效率。

功能简介
----
主要分为两个端：店铺端 和总后台

一、店铺端主要功能：

	1.房间管理
		添加/修改房间以及房间信息
		添加修改名宿内售卖的商品
	2.旅客管理
		添加/修改旅客以及团队信息
	3.住宿管理
		添加/修改预定房间的信息
		添加/修改客户住宿信息
		房间结算
	4.财务管理
		显示今日店铺的入住信息以及收入情况
	5.个人设置
		修改个人基本信息
		意见反馈
		

二、后台主要功能

	1.收益预览
		查看每个类别收入多少钱
	2.用户信息
		查看各个店铺的帐号信息
	3.意见反馈
		查看各个店铺的反馈信息

项目适用人群
----
正在做毕设的学生，或者需要项目实战练习的Java学习者

开发环境：
-----
1. jdk 8
2. intellij idea
3. tomcat 8.5.40
4. mysql 5.7

所用技术：
-----
1. Spring+SpringMVC+MyBatis
2. layui
3. jsp

项目访问地址
----
前端访问地址
```
http://localhost:8090/login
```
 
项目截图
----

- 系统功能
![](/src/image/系统功能.jpg)
- 平台-用户信息
![](/src/image/平台-用户信息.png)
- 平台-收益预览
![](/src/image/平台-收益预览.png)
- 旅客管理-旅客信息
![](/src/image/旅客管理-旅客信息.jpg)
- 房间设置-添加房间
![](/src/image/房间设置-添加房间.png)
- 商品设置-添加商品
![](/src/image/商品设置-添加商品.png)
- 住宿登记-结算
![](/src/image/住宿登记-结算.png)

数据库配置
----
1. 数据库配置信息
```
#配置文件
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost/homestay?useUnicode=true&characterEncoding=utf-8&useSSL=false
jdbc.username=root
jdbc.password=root123
```
2. 数据库配置加载
```
 <!-- 数据源dataSource -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
    <!--maxActive: 最大连接数量 -->
    <property name="maxActive" value="150" />
    <!--minIdle: 最小空闲连接 -->
    <property name="minIdle" value="5" />
    <!--maxIdle: 最大空闲连接 -->
    <property name="maxIdle" value="20" />
    <!--initialSize: 初始化连接 -->
    <property name="initialSize" value="30" />
    <!--maxWait: 超时等待时间以毫秒为单位 1000等于60秒 -->
    <property name="maxWait" value="1000" />
    <!-- 在空闲连接回收器线程运行期间休眠的时间值,以毫秒为单位. -->
    <property name="timeBetweenEvictionRunsMillis" value="10000" />
    <!-- 在每次空闲连接回收器线程(如果有)运行时检查的连接数量 -->
    <property name="numTestsPerEvictionRun" value="10" />
    <!-- 1000 * 60 * 30 连接在池中保持空闲而不被空闲连接回收器线程 -->
    <property name="minEvictableIdleTimeMillis" value="10000" />
    <property name="validationQuery" value="SELECT NOW() FROM DUAL" />
</bean>
```
3. 资源配置
```
<mvc:interceptors>
<mvc:interceptor>
    <!-- 需要拦截的url路径 :/order/**  这个是订单系统的url格式 -->
    <mvc:mapping path="/**" />
    <mvc:exclude-mapping path="/login"/>
    <mvc:exclude-mapping path="/*.js"/>
    <mvc:exclude-mapping path="/register"/>
    <mvc:exclude-mapping path="/logout"/>
    <bean class="com.chengxubang.interceptor.LoginInterceptor"></bean>
</mvc:interceptor>
</mvc:interceptors>
<!-- 支持aop的注解 -->
<aop:aspectj-autoproxy/>
<!-- 启动SpringMVC的注解功能 -->
<mvc:annotation-driven/>

<!--静态资源放行-->
<mvc:default-servlet-handler/>

<!-- 定义跳转的文件的前后缀 ，视图解析器配置-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/views/"/>
    <property name="suffix" value=".jsp"/>
</bean>

<!-- 配置文件上传解析器 -->
<bean id="multipartResolver"
      class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 默认编码 -->
    <property name="defaultEncoding" value="utf-8"/>
    <!-- 文件大小最大值 -->
    <property name="maxUploadSize" value="10485760000"/>
</bean>
```

业务代码
-----
1. 民宿后台首页，展示收益情况
```diff
@RequestMapping("/welcome")
public ModelAndView welcome(ModelAndView modelAndView){
    User user=getUser();
    HotelRegisterQuery query=new HotelRegisterQuery();
    if(!user.getUsername().equals("admin")){
        query.setUserId(getUserId());
    }
    query.setType(0);//旅客收益
    List<HotelRegister> type0list=hotelRegisterService.getEarningsByPtype(query);
    query.setType(1);//团队收益
    List<HotelRegister> type1list=hotelRegisterService.getEarningsByPtype(query);
    String iStr="";
    int list0Index=0;	//旅客数据下标
    int list1Index=0;	//团队数据下标
    String type0Data="";//旅客每月收益
    String type1Data="";//团队每月收益
    String month="";
        for(int i=1;i<13;i++){
            if(i<10){	//小10的情况下拼接0，方便与查询数据对比
                iStr="0"+i;
            }else{
                iStr=i+"";
            }
            month+=iStr+",";
            if(type0list.size()>0){
                if(iStr.equals(type0list.get(list0Index).getSettleTime())){
                    type0Data=type0Data+type0list.get(list0Index).getAllFee()+",";//如果匹配上月份，就拼接当前月的收益
                    if(!(list0Index+1>=type0list.size())){		//大于等于list大小，说明已经没有数据
                        list0Index++;
                    }
                }else{
                    type0Data=type0Data+"0,";
                }
            }else{
                type0Data=type0Data+"0,";
            }
            if(type1list.size()>0){
                if(iStr.equals(type1list.get(list1Index).getSettleTime())){
                    type1Data=type1Data+type1list.get(list1Index).getAllFee()+",";
                    if(!(list1Index+1>=type1list.size())){			//大于等于list大小，说明已经没有数据
                        list1Index++;
                    }
                }else{
                    type1Data=type1Data+"0,";
                }
            }else{
                type1Data=type1Data+"0,";
            }
        }
    type0Data=type0Data.substring(0,type0Data.length()-1);
    type1Data=type1Data.substring(0,type1Data.length()-1);
    month=month.substring(0,month.length()-1);

    modelAndView.addObject("month",month);
    modelAndView.addObject("type0Data",type0Data);
    modelAndView.addObject("type1Data",type1Data);
    modelAndView.setViewName("welcome");
    return modelAndView;
}
//jsp数据渲染
<html>
<head>
    <script type="text/javascript" src="${ctx}/static/js/echarts-all.js"></script>
</head>
<body>

<div id="main" style="width: 95%;height:666px"></div>
<script type="text/javascript">
    var myChart = echarts.init(document.getElementById('main'));

    // 指定图表的配置项和数据
    var option = {
        title: {
            text: '收益金额折线图'
        },
        tooltip: {
            trigger: 'axis'
        },
        legend: {
            data:['旅客','团队']
        },
        grid: {
            left: '3%',
            right: '4%',
            bottom: '1%',
            containLabel: true
        },
        toolbox: {
            feature: {
                saveAsImage: {}
            }
        },
        xAxis: {
            type: 'category',
            boundaryGap: false,
            data: [${month}]
        },
        yAxis: {
            type: 'value'
        },
        series: [
            {
                name: '旅客',
                type: 'line',
                stack: '总量',
                data: [${type0Data}]
            },
            {
                name: '团队',
                type: 'line',
                stack: '总量',
                data: [${type1Data}]
            }
        ]
    };

    // 使用刚指定的配置项和数据显示图表。
    myChart.setOption(option);
</script>
</body>
</html>
```

2. 房间添加
```diff
@RequestMapping("/save")
@ResponseBody
public AjaxResult save(Room room){
    try {
        if(room.getId()!=null){
            roomService.update(room);
        }else{
            RoomQuery query=new RoomQuery();
            query.setRoomNum(room.getRoomNum());
            query.setUserId(getUserId());
            PageList pageList= roomService.getByQuery(query);
            if(pageList.getCount()>0){
                return new AjaxResult("保存失败，房间号已经存在!",-10002);
            }else{
                room.setUserId(getUserId());
                roomService.add(room);
            }
        }
        return new AjaxResult("保存成功!");
    } catch (Exception e) {
        e.printStackTrace();
        return new AjaxResult("保存失败:"+e.getMessage(),-10002);
    }
}
```

2. 商品添加
```diff
@RequestMapping("/save")
@ResponseBody
public AjaxResult save(Goods goods){
    try {
        if(goods.getId()!=null){
            goodsService.update(goods);
        }else{
            GoodsQuery goodsQuery=new GoodsQuery();
            goodsQuery.setGoodName(goods.getGoodName());
            goodsQuery.setUserId(getUserId());
            PageList pageList=goodsService.getByQuery(goodsQuery);
            if(pageList.getCount()>0){
                return new AjaxResult("保存失败,改商品名已经存在",-10002);
            }else{
                goods.setUserId(getUserId());
                goodsService.add(goods);
            }
        }
        return new AjaxResult("保存成功!");
    } catch (Exception e) {
        e.printStackTrace();
        return new AjaxResult("保存失败:"+e.getMessage(),-10002);
    }
}
```

项目后续
----
其他ssh，springboot版本后续迭代更新，持续关注
程序有问题联系[程序帮](http://suo.nz/530ijn)