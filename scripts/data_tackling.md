##~/YOPO/YOPO/test_yopo_ros.py文件中
##添加直接得出路径长度、飞行时间和坐标的代码

	#new_add time and len record
        self.start_time = time.time()
        self.start_pos = np.array([self.odom.pose.pose.position.x,self.odom.pose.pose.position.y,self.odom.pose.pose.position.z])
        self.path_len = 0.0
        self.last_pos = self.start_pos
        print(f"Start pos: ({self.start_pos[0]:.3f},{self.start_pos[1]:.3f}, {self.start_pos[2]:.3f})")

        # 新增：累加路径长度
        if hasattr(self, 'last_pos'):
            self.path_len += np.linalg.norm(pos - self.last_pos)
            self.last_pos = pos

        if np.linalg.norm(pos - self.goal) < 5 and not self.arrive:
            print("Arrive!")
            
            # 新增：打印飞行时间和路径长度
            if hasattr(self, 'start_time'):
                flight_time = time.time() - self.start_time
                print(f"Flight time: {flight_time:.2f} s")
                print(f"Path length: {self.path_len:.2f} m")
            
            self.arrive = True
