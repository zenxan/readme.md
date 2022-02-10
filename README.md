# 代码流程讲解

**10 Feb 2022:** 第一次书写

**Author:** qingxuan zeng

## 1.程序入口

### 1.1 first step
**./pipeline/par_run.py**

默认参数：--csv_path {path} --clip_root_dir {path}

从csv文件中获取**clip**的值，作为参数传入到下一个脚本 **./run_slam3.sh**

### 1.2 second step
**./run_slam3.sh**

其中涉及到多个路径，需要对这些路径一一整理：

**pcl_third_part_folder** &nbsp; &nbsp; &nbsp; 存放一些共享库  
**clip_dir** &nbsp; &nbsp; &nbsp; 从par_run.py传入的文件路径  
**npz_dir** &nbsp; &nbsp; &nbsp; clip_dir路径下的PANDAR128文件夹，存放的是numpy数据  
**pcd_dir** &nbsp; &nbsp; &nbsp; clip_dir路径下的sub_pcds文件夹(之前是pcds文件夹)，存放的是txt文件（里面是pcd文件的命名）
**pcd_files** &nbsp; &nbsp; &nbsp; sub_pcds下所有的txt文件
**calibration_folder** &nbsp; &nbsp; &nbsp; 标定文件夹，存在json和txt两种格式

以上是涉及到的基本路径，需要每个文件夹都浏览一遍，知道存放的内容是什么。

#### 进入该脚本的for循环
**输入:**  
rtk_pose_file:&nbsp; &nbsp; &nbsp;0.txt-9.txt  
pcd_list:&nbsp; &nbsp; &nbsp;0.txt-9.txt

**输出:**  
clip_out_dir:&nbsp; &nbsp; &nbsp;clip_dir路径下的output文件夹  
sub_map_out_dir:&nbsp; &nbsp; &nbsp;clip_out_dir路径下的0-9文件夹  
sub_map_pcd_out_dir:&nbsp; &nbsp; &nbsp;sub_map_out_dir路径下的pcds文件夹  
sub_lidar_pose_out:&nbsp; &nbsp; &nbsp;雷达的位姿（四元素）
sub_lidar_pose_4f_out:&nbsp; &nbsp; &nbsp;雷达的位姿（变换矩阵）
sub_cam_pose_out:&nbsp; &nbsp; &nbsp;相机的位姿（变换矩阵）
sub_map_ply:&nbsp; &nbsp; &nbsp;


### 1.3 third step
**./mv_pcd.py**

默认参数：--pcd_dir {path} --pcd_name_list {path} --target_dir {path}

将**pcd_dir**中的pcd文件，按照**pcd_name_list**的对应名称，复制到**target_dir**中


### 1.4 fourth step
**./lidar_odometry/apps/build/main_lo_app**  
源文件：./lidar_odometry/apps/main_lo_app.cpp

**lo_compensated_pcds** &nbsp; &nbsp; &nbsp; 存放pcd点云的真实值和测试值 

默认参数: --pcd_dir {path} --lo_pose {path} --deskew_dir {path} ......


### 1.5 fifth step
**./ground_extraction/scripts/pipeline_ground_extraction.py**  
默认参数: --clip_name {path} --pcd_folder {path} --calib_cfg {path } ......

功能：提取ground trueth


### 1.6 sixth step
**./lidar_odometry/apps/build/main_lo_app**  
与**1.4**程序一致，参数不同

功能：scan-stiching


### 1.7 seventh step
**./point_cloud_registration/scripts/generate_height_map.py**
参数: lo_local_map, lo_to_label

**lo_local_map** &nbsp; &nbsp; &nbsp;  输出文件夹中的lidar_pose.txt.ply文件
**lo_to_label** &nbsp; &nbsp; &nbsp;  输出文件夹中的lo_to_label文件夹

功能：generate height map


### 1.8 eighth step
**./point_cloud_registration/scripts/rgb_render.py**  
默认参数: --calib_path {path} --image_dir {path} ......

**sub_image_list** &nbsp; &nbsp; &nbsp;  clip_dir文件夹下，sub_images中的所有txt文件

功能：generate cam pose


### 1.9 ninth step
**./point_cloud_registration/scripts/data_gen.py**  
默认参数: --sub_map_out_dir {path} --sub_local_map_dir {path} ......

**image_dir** &nbsp; &nbsp; &nbsp;  clip_dir文件夹下，camera_full_undistortion/FW中的所有jpg文件

功能: formulate data style

