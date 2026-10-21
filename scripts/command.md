##启动控制器和动力学仿真：
cd Controller
source devel/setup.bash
roslaunch so3_quadrotor_simulator simulator_attitude_control.launch
##启动环境和传感器仿真：
cd Simulator
source devel/setup.bash
rosrun sensor_simulator sensor_simulator_cuda
##启动 YOPO：
cd YOPO
conda activate yopo_gpu(gpu运行）
python test_yopo_ros.py --trial=1 --epoch=50
##启动 RViz：
cd YOPO
rviz -d yopo.rviz
