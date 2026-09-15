# 问题记录

## 1. PyTorch 不支持 RTX 5060（sm_120）
**现象**：报错 sm_120 is not compatible with current PyTorch installation。  
**原因**：原环境 Python 3.8 + PyTorch 2.4.1+cu118 不支持 RTX 5060。  
**解决**：新建 Python 3.10 环境 `yopo_gpu`，安装 `torch==2.8.0+cu128`。

## 2. nvcc 不支持 compute_120
**现象**：`nvcc fatal: Unsupported gpu architecture 'compute_120'`。  
**原因**：系统 CUDA 是 11.8，不认识 compute_120。  
**解决**：安装 CUDA 12.8 Toolkit，更新 PATH 和 LD_LIBRARY_PATH，重新编译 Simulator。

## 3. 控制器死等 /mavros/state，无人机不动
**现象**：YOPO 正常发指令，但无人机位置不变，`/so3_cmd` 只有悬停力。  
**原因**：仿真中没有 `/mavros/state` 发布者，控制器卡在等待 armed + OFFBOARD。  
**解决**：运行 `fake_mavros.py`，持续发布 `/mavros/state`，设置 `armed=True`、`mode='OFFBOARD'`。

## 总结
| # | 问题 | 原因 | 解决 |
|---|------|------|------|
| 1 | PyTorch 不支持 sm_120 | Python 3.8 装不上新版 PyTorch | 换 Python 3.10 + torch 2.8.0+cu128 |
| 2 | nvcc 不认识 compute_120 | CUDA 11.8 太旧 | 装 CUDA 12.8 并重新编译 |
| 3 | 控制器不转发指令 | 缺 /mavros/state | 伪造 mavros 状态 |
