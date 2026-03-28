# 玩家位置坐标与朝向相关 Protobuf 结构索引

> 本文档汇总了 `StarResonanceDpsAnalysis.Proto/proto/` 目录下与玩家当前位置坐标和朝向相关的 protobuf 结构定义。

---

## 目录

- [zproto — 运行时协议结构](#zproto--运行时协议结构)
  - [核心位置结构](#核心位置结构)
  - [玩家角色位置](#玩家角色位置)
  - [实体属性中的位置/朝向](#实体属性中的位置朝向)
  - [场景传送与进入](#场景传送与进入)
  - [路径与移动](#路径与移动)
  - [向量辅助类型](#向量辅助类型)
- [bokura — 配置表结构](#bokura--配置表结构)
- [table_basic — 基础向量类型](#table_basic--基础向量类型)

---

## zproto — 运行时协议结构

### 核心位置结构

#### `Position`（`stru_position.proto`）

最核心的位置与朝向结构，同时包含坐标和方向：

```protobuf
message Position {
  float x   = 1;  // X 坐标
  float y   = 2;  // Y 坐标（高度）
  float z   = 3;  // Z 坐标
  float dir = 4;  // 朝向角度
}
```

被广泛引用于以下结构中，是玩家/实体位置的基础类型。

---

### 玩家角色位置

#### `CharBaseInfo`（`stru_char_base_info.proto`）

玩家角色基础信息，直接内含坐标与朝向字段：

```protobuf
message CharBaseInfo {
  int64  char_id    = 1;   // 角色 ID
  string account_id = 2;   // 账号 ID
  string name       = 5;   // 角色名
  // ...
  float  x          = 10;  // X 坐标
  float  y          = 11;  // Y 坐标（高度）
  float  z          = 12;  // Z 坐标
  float  dir        = 13;  // 朝向角度
  // ...
}
```

通过 `SyncContainerData` 消息在玩家初始化时下发，是获取玩家初始位置的主要来源。

#### `SettlementPosition`（`stru_settlement_position.proto`）

结算/快照时的玩家位置与旋转：

```protobuf
message SettlementPosition {
  Position pos    = 1;  // 位置坐标（含朝向）
  Position rotate = 2;  // 旋转信息
}
```

#### `SettlementPositionParam`（`stru_settlement_position_param.proto`）

多玩家位置映射表，键为用户 ID：

```protobuf
message SettlementPositionParam {
  map<uint32, SettlementPosition> v_user_pos = 1;
}
```

#### `LastSceneData`（`stru_last_scene_data.proto`）

玩家上次所在场景的位置记录：

```protobuf
message LastSceneData {
  uint32   scene_id      = 1;  // 场景 ID
  Position pos           = 2;  // 位置坐标（含朝向）
  int32    scene_area_id = 3;  // 场景区域 ID
}
```

---

### 实体属性中的位置/朝向

#### `Entity`（`stru_entity.proto`）

游戏实体（玩家、怪物、NPC 等）的运行时数据容器。位置和朝向信息存储在 `attrs` 属性集合中：

```protobuf
message Entity {
  int64              uuid       = 1;  // 实体唯一 ID
  EEntityType        ent_type   = 2;  // 实体类型（EntChar / EntMonster 等）
  AttrCollection     attrs      = 3;  // 属性集合（含位置/朝向）
  TempAttrCollection temp_attrs = 4;  // 临时属性
  // ...
}
```

#### 位置/朝向相关的 `EAttrType` 枚举值（`enum_e_attr_type.proto`）

通过增量同步（`AoiSyncDelta`）更新实体位置时使用的属性类型：

| 枚举值 | 编号 | 说明 |
|--------|------|------|
| `AttrDir` | 50 | 实体朝向（Y 轴旋转） |
| `AttrTargetDir` | 51 | 目标朝向 |
| `AttrPos` | 52 | 实体位置坐标 |
| `AttrTargetPos` | 53 | 目标位置坐标 |
| `AttrFinalTargetDir` | 110 | 最终目标朝向 |
| `AttrFinalTargetPos` | 118 | 最终目标位置 |
| `AttrTargetPartPos` | 119 | 目标部位位置 |
| `AttrDmgTargetPos` | 120 | 伤害目标位置 |
| `AttrRotation` | 374 | 旋转 |
| `AttrRotate` | 420 | 旋转 |
| `AttrDirX` | 433 | 朝向 X 分量 |
| `AttrDirZ` | 434 | 朝向 Z 分量 |
| `AttrTargetDirX` | 435 | 目标朝向 X 分量 |
| `AttrTargetDirZ` | 436 | 目标朝向 Z 分量 |
| `AttrTargetPosIsEnd` | 569 | 目标位置是否为终点 |

---

### 场景传送与进入

#### `ScenePointInfo`（`stru_scene_point_info.proto`）

场景点位信息（含位置与相机）：

```protobuf
message ScenePointInfo {
  Position position      = 1;  // 场景点位坐标
  int32    camera_id     = 2;  // 相机 ID
  int32    scene_area_id = 3;  // 场景区域 ID
}
```

#### `PositionParam`（`stru_position_param.proto`）

位置参数（用于传送等场景）：

```protobuf
message PositionParam {
  ScenePointInfo    scene_point_info     = 2;
  ScenePosIdInfo    scene_pos_info       = 3;
  CutScenePointInfo cut_scene_point_info = 4;
}
```

#### `TransferParam`（`stru_transfer_param.proto`）

场景传送参数：

```protobuf
message TransferParam {
  int32             scene_id               = 1;   // 目标场景 ID
  EUserTransferType transfer_type          = 2;   // 传送类型
  PositionParam     position_param         = 3;   // 位置参数
  int64             change_flag            = 4;
  bool              is_server_switch       = 5;
  int32             visual_layer_config_id = 6;
  string            scene_guid             = 7;
  string            connect_guid           = 8;
  int64             sub_scene_uuid         = 9;
  bool              is_revive              = 10;
}
```

#### `EnterSceneParams`（`stru_enter_scene_params.proto`）

进入场景参数：

```protobuf
message EnterSceneParams {
  int32         change_scene_type = 1;
  int32         scene_id          = 2;
  string        scene_guid        = 3;
  TransferParam transfer_param    = 4;
}
```

---

### 路径与移动

#### `PathPointChangeParam`（`stru_path_point_change_param.proto`）

路径点变更参数（移动路径更新）：

```protobuf
message PathPointChangeParam {
  optional int32    Operation        = 1;  // 操作类型
  optional Position AddPoint         = 2;  // 新增路径点
  optional int32    RemovePointCount = 3;  // 移除点数量
}
```

---

### 向量辅助类型

#### `IntVec3`（`stru_int_vec3.proto`）

整数三维向量：

```protobuf
message IntVec3 {
  int32 x = 1;
  int32 y = 2;
  int32 z = 3;
}
```

#### `Vec4`（`stru_vec4.proto`）

四维向量（四元数旋转）：

```protobuf
message Vec4 {
  float x = 1;
  float y = 2;
  float z = 3;
  float w = 4;
}
```

---

## bokura — 配置表结构

以下 bokura 目录中的配置表包含 `position`（坐标）和 `rotation`（朝向）字段。  
这些是**静态配置数据**（实体出生点、刷新点等），而非运行时玩家位置。

| 配置表文件 | 消息类型 | 位置字段 | 朝向字段 |
|-----------|---------|---------|---------|
| `MonsterEntityTable.proto` | `MonsterEntityTableBase` | `position` (686) | `rotation` (783), `rotationx` (1044), `rotationz` (23) |
| `NpcEntityTable.proto` | `NpcEntityTableBase` | `position` (686) | `rotation` (783), `rotationx` (1044), `rotationz` (23) |
| `ZoneEntityTable.proto` | `ZoneEntityTableBase` | `position` (686) | `rotation` (783), `rotationx` (1044), `rotationz` (23) |
| `ZoneEntityGlobalTable.proto` | `ZoneEntityGlobalTableBase` | `position` (686) | `rotation` (783), `rotationx` (1044), `rotationz` (23) |
| `CollectionEntityTable.proto` | `CollectionEntityTableBase` | `position` (686) | `rotation` (783) |
| `DummyEntityTable.proto` | `DummyEntityTableBase` | `position` (686) | `rotation` (783) |
| `SceneObjectEntityTable.proto` | `SceneObjectEntityTableBase` | `position` (686) | `rotation` (783) |
| `ScenePointInfoTable.proto` | `ScenePointInfoTableBase` | `position` (686) | `rotation` (783) |
| `TrapEntityTable.proto` | `TrapEntityTableBase` | `position` (686) | `rotation` (783) |

bokura 配置表中的字段类型说明（括号中的数字为 protobuf 字段编号，编号不连续是正常现象）：
- `position`：`number_array` 类型（浮点数组，通常为 `[x, y, z]`）
- `rotation`：`float` 类型（Y 轴旋转角度）
- `rotationx`：`float` 类型（X 轴旋转角度）
- `rotationz`：`float` 类型（Z 轴旋转角度）

---

## table_basic — 基础向量类型

`table_basic.proto` 中定义的向量类型，被 bokura 配置表引用：

```protobuf
message vector2 {
  float x = 1;
  float y = 2;
}

message vector3 {
  float x = 1;
  float y = 2;
  float z = 3;
}

message vector2_array { repeated vector2 array = 1; }
message vector3_array { repeated vector3 array = 1; }
```

---

## 数据流概览

```
玩家登录
  └─► SyncContainerData → CharBaseInfo (x, y, z, dir)     ← 初始位置
  └─► SyncNearEntities  → Entity.attrs (AttrPos, AttrDir)  ← 周围实体位置

运行时更新
  └─► AoiSyncDelta → EAttrType.AttrPos / AttrDir           ← 增量位置/朝向更新
  └─► PathPointChangeParam                                  ← 移动路径更新

场景切换
  └─► EnterSceneParams → TransferParam → PositionParam      ← 传送目标位置
  └─► LastSceneData                                         ← 上次场景位置记录

结算快照
  └─► SettlementPositionParam → SettlementPosition          ← 多玩家位置快照
```
