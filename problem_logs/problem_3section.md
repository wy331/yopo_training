# 未理解问题记录

## 1. state_transform.py 的具体变换逻辑

**现象**：
`normalize_obs`、`prepare_input`、`pred_to_endstate`、`state_body2world` 这些函数在 `yopo_network.py`、`yopo_trainer.py`、`test_yopo_ros.py` 中被反复调用，但 `state_transform.py` 文件未提供，看不到具体实现。不清楚它们如何影响网络输入输出的数值范围。

**原因**：
- 只能看到调用处，例如：
  ```python
  obs = self.state_transform.normalize_obs(obs)
  obs = self.state_transform.prepare_input(obs)
  endstate = self.state_transform.pred_to_endstate(endstate_pred)
  ```
- 不知道 `normalize_obs` 用的是均值方差还是最大最小归一化；
- 不知道 `prepare_input` 是否做了 reshape、维度重排或转置；
- 不知道 `pred_to_endstate` 是否乘了 `vel_max`、`acc_max` 或 `goal_length`；
- 不知道 `state_body2world` 的旋转矩阵方向是 `R_wb` 还是 `R_bw`。

**解决方向**：
- 找到 `policy/state_transform.py` 源码，逐函数阅读；
- 对照 `traj_opt.yaml` 中的 `vel_max_train`、`acc_max_train`、`goal_length`，确认归一化尺度；
- 用一个小样例手动跑一遍 `normalize_obs` 和 `pred_to_endstate`，打印输入输出数值范围；
- 对照 `yopo_trainer.py` 中 `state_body2world` 的调用，确认旋转方向。

## 2. YopoBackbone 和 YopoHead 的网络结构

**现象**：
`yopo_network.py` 中引用了 `YopoBackbone` 和 `YopoHead`，但 `policy/models/backbone.py` 和 `policy/models/head.py` 未提供。不知道卷积层、全连接层具体怎么设计，也不清楚为什么 `output_dim = 10`。

**原因**：
- 只能看到：
  ```python
  self.image_backbone = YopoBackbone(hidden_state)
  self.yopo_head = YopoHead(hidden_state + observation_dim, output_dim)
  endstate = torch.tanh(output[:, :9])
  score = torch.nn.functional.softplus(output[:, 9])
  ```
- 不知道 backbone 用的是什么 CNN（ResNet？自定义卷积？），输出特征维度是多少；
- 不知道 head 是几层全连接、每层多少神经元；
- `output_dim = 10` 只能从 `output[:, :9]` 和 `output[:, 9]` 反推是 9 维末端状态 + 1 维 score，但具体设计动机不清楚。

**解决方向**：
- 找到 `policy/models/backbone.py` 和 `policy/models/head.py`；
- 画出每一层的输入输出维度；
- 确认 `hidden_state = 64` 在 backbone 和 head 中如何传递；
- 对照论文或原仓库，确认 `output_dim = 10` 是否就是“9 维 PVA + 1 维 score”的设计。

## 3. ESDF 地图的构建与查询细节

**现象**：
`SafetyLoss.get_sdf_from_ply` 如何从点云构建 ESDF、`get_batch_sdf` 的裁剪逻辑如何保证不同地图在 batch 中正确对齐，这两点只能看到代码，无法完全确认其正确性和边界情况。

**原因**：
- `get_sdf_from_ply` 中：
  ```python
  dist_to_obstacle = distance_transform_edt(free_mask) * self.voxel_size
  dist_inside_obstacle = distance_transform_edt(obstacle_mask) * self.voxel_size
  dist_to_obstacle[obstacle_mask] = -dist_inside_obstacle[obstacle_mask]
  ```
  只能看出障碍物内为负、外为正，但不清楚 `map_expand_min/max` 对边界的影响。
- `get_batch_sdf` 中：
  ```python
  min_indices = centers - max_spans // 2 - 5
  max_indices = centers + max_spans // 2 + 5
  ...
  new_min_indices = min_indices.clamp(min=0)
  underflow_amount = new_min_indices - min_indices
  ```
  裁剪、clamp、underflow/overflow 调整的逻辑较复杂，不清楚多地图 batch 时是否会出现错位。
- `grid_sample` 使用 `align_corners=True`，归一化到 `[-1, 1]`，不清楚体素坐标与物理坐标的对应是否严格一致。

**解决方向**：
- 用一张小地图手动构造 ESDF，打印 `min_bounds`、`max_bounds`、`sdf_shapes`；
- 取一个已知点，手动算 `grid` 和 `grid_point`，验证 `grid_sample` 查询结果是否等于该点真实距离；
- 构造 batch 包含两张不同地图，检查 `get_batch_sdf` 返回的 `local_origin`、`local_shape` 是否与各自地图对齐；
- 对照 `open3d` 点云坐标和 `voxel_size = 0.2`，确认索引与物理坐标的换算无误。

## 总结

| # | 未理解问题 | 卡点 | 下一步 |
|---|-----------|------|--------|
| 1 | `state_transform.py` 变换逻辑 | 文件未提供，数值范围不清楚 | 阅读源码，手动跑样例验证 |
| 2 | `YopoBackbone` / `YopoHead` 结构 | 文件未提供，层设计不清楚 | 阅读源码，画出维度变化 |
| 3 | ESDF 构建与查询细节 | 裁剪逻辑复杂，边界情况不清楚 | 手动构造小地图验证 `grid_sample` |
