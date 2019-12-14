场景示例: <br>
          <br>
    APP或web端上, 获取距离当前用户最近的10个用户/最近的网约车/最近的商铺<br>
    说明: 一般使用高德地图之类的服务, 都提供了两点直接的距离计算,但如果样本量足够大,这种方式很受限,大大影响效率。<br>
    比如, 假如一个APP上共有50000个用户,如果想获取距离当前用户最近的10个人,用高德提供的在线API计算需要怎么做? <br>
    需要把当前用户位置与50000个人用户的位置计算相对位置距离,然后把相对位置距离从小到大排序,取前10个,这种做法效率极慢,
    且不适合用让java来做, 所以最好的解决办法是让数据库来做。<br>
    
    __mysql代码操作:__
    
    
      /** 1.首先创建用户坐标表
      
      CREATE TABLE `t_user_location` (
      `t_id` int(11) NOT NULL AUTO_INCREMENT,
      `t_userId` int(11) NOT NULL,
      `t_longitude` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
      `t_latitude` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
      `t_updateTime` datetime DEFAULT NULL,
      PRIMARY KEY (`t_id`)
    ) ENGINE=MyISAM AUTO_INCREMENT=5 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;


    INSERT INTO `t_user_location` VALUES ('1', '1', '116.425249', '39.914504', '2019-10-24 16:58:20');
    INSERT INTO `t_user_location` VALUES ('2', '2', '116.403882', '39.915139', '2019-10-24 16:58:45');
    INSERT INTO `t_user_location` VALUES ('3', '3', '116.382001', '39.913329', '2019-10-24 16:58:59');
    INSERT INTO `t_user_location` VALUES ('4', '4', '116.610389', '35.440957', '2019-10-24 16:59:14');


    /** 2.用mysql内置函数来计算
    
    SELECT 
    t_userId,(6378100 * ACOS(COS(RADIANS(39.914504)) * COS(RADIANS(t_latitude)) * COS(RADIANS(116.425249 - t_longitude)) + SIN(RADIANS(39.914504)) * SIN(RADIANS(t_latitude)))
	   ) AS distance,
    t_longitude, t_latitude FROM  t_user_location ORDER BY distance; 
    
    
    上述sql中, 39.914504:代表当前用户的纬度    116.425249:代表当前当用的经度   t_latitude:代表数据库中用户坐标表的纬度字段 
             t_longitude:代表数据库中用户坐标表的经度字段 
             这样的话,会按相对位置距离从小到大排序,可以取需要的前N个数据
             
             
    可靠度: 上述select查询语句直线距离结果 与 高德API直线距离计算 比较, 误差:每50000米 误差3米   
      
     /**若想要结果按userId为 6,3,1,4,2顺序输出, 不在是默认的1,2,3,4,6输出, 满足部分业务场景需要 */ 
    select * from t_user where   t_id in (6,3,1,4,2)   ORDER BY field (6,3,1,4,2) 
             
    补充:如果仅比较2个人之间的相对距离,可以使用高德API计算
    
    
