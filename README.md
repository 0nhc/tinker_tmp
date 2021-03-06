# 使用说明：

* 使用前记得替换launch文件里的地图

* roslaunch tinker_navigation hzx_nav.launch

## 1.定位

* 定位功能包使用iris_lama_ros替换cartographer

### 1.1替换原因

* [这个定位包的IEEE论文](Efficient_localization_based_on_scan_matching_with_a_continuous_likelihood_field.pdf)

* 大概意思是先描述了一下这个定位方法是基于scan matching的（ICP也是），然后说这个算法解决了ICP的一些问题，论文最后还对比了在不同数据集上定位的速度，比AMCL快很多。

* 不用cartographer是因为，之前实际测试的时候，会出现高速运动突然停下之后定位会来回摆动几秒才能稳定下来的情况，严重影响定位速度。

## 1.2 补充

* 但是我没有实际测试过amcl的效果，因为论文里对比了这个算法比起AMCL的定位速度，有时间可以自己再测试一下。

## 2.里程计

* 上面的定位只是一个单独的定位包，这个包像amcl一样，还是需要依靠里程计数据的输入，在这里我使用了laser_scan_matcher包。

### 2.1思路

* 为了解决被撞击轮子打滑的问题，不能使用轮速计，因此需要使用激光里程计。在这里我使用了laser_scan_matcher包，因为20.04系统里能运行的只有这一个包。。（rf2o在20.04里运行不起来）
* 本质上和cartographer是一个实现思路，不同的是cartographer是一个包里自带定位和激光里程计，我们现在只不过是分别把这两个功能用别的包来替换，为了解决cartographer在定位时的问题（上面有写）

### 2.2补充

* 其实我想是以后把轮速计和激光里程计融合起来，比如轮速计的节点只发布/odometry1话题，激光里程计的节点只发布/odometry2的话题，然后这两个节点都不发布odom->base_footprint的坐标变换。然后自己写一个节点订阅/odometry1和/odometry2，取这两个消息的平均值然后最终发布成/odom话题，然后再用这个平均值去发布odom->base_footprint的变换。这样做是因为轮速计在旋转的时候相对激光里程计准确很多，而激光里程计在机器打滑或撞击的时候相对轮速计准确很多，两者互补一下就能得到一个相对准确的里程计。那个定位包所依赖的里程计数据不需要特别准确，所以在高速旋转或者撞击打滑这种只有只有一种里程计准确的情况下也能定位准确。
