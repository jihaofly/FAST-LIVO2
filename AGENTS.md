# AGENTS.md — FAST-LIVO2 建图与里程计

快速 LiDAR-惯性-视觉里程计 / 建图（FAST-LIVO2），本机为 `jihaofly/FAST-LIVO2`（fork 自 hku-mars），分支 `main`。修改前请阅读。

## 定位

接收 Livox 雷达 + IMU（+ 相机）数据，输出里程计与点云，并负责 **PCD 点云保存**（X2 异步写盘定制）。

## 关键文件

| 文件 | 说明 |
|------|------|
| `src/LIVMapper.cpp` | 核心建图逻辑；`savePCD()`（fork 分发）+ `writePCDFiles()`（写盘） |
| `include/LIVMapper.h` | 类声明（含 `save_session_id_`、`writePCDFiles`） |
| `config/MID360_Fly.yaml` | 飞行定位（img_en=1；pcd_save 由 launch 参数覆盖为 false） |
| `config/MID360_Fly_Lidar.yaml` | 高质量建图（img_en=0 → 只写 intensity 单 PCD） |
| `launch/mapping_MID360.launch` | 建图 launch（`pcd_save` / `rviz` / `config_name` 参数） |

## X2 PCD 异步写盘（本仓库核心定制）

- 主进程收到 SIGINT → `savePCD()` fork 子进程 `pcd_saver` 后台写盘 → 主进程**秒退**
- 子进程 `setsid()` 脱离进程组 + `prctl(PR_SET_NAME,"pcd_saver")`，孤儿后台写，不占串口/端口
- 输出：`Log/pcd/<启动时间戳>.pcd`（`save_session_id_` = 启动时刻 `%Y%m%d_%H%M%S`），不覆盖历史
- fork 失败退化为同步写盘兜底
- **外部匹配主进程用 `-x fastlivo_mappi`**（comm 15 字符截断，勿用 `-f fastlivo_mapping` 以免误伤 pcd_saver）

## 修改后必须

```bash
source /opt/ros/noetic/setup.bash
catkin_make --pkg fast_livo    # 编译验证
```

## 注意

- 改动后推送 `origin main`，并在顶层 `catkin_ws` 更新 `src/FAST-LIVO2` 子模块指针
- 完整 X2 设计见顶层 `docs/RADAR_PCD_X2.md`
