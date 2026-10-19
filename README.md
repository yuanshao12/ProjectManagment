# 全流程项目节点管理系统（PMS）

企业项目全生命周期节点管控系统。覆盖 **8 大阶段 / 24 标准节点**（商机储备 → 方案深化 → 招投标 → 合同立项 → 实施建设 → 交付验收 → 运营维护 → 结项复盘），每个节点绑定负责人、计划/实际时间与交付物，节点串行流转，逾期自动预警并推送站内消息。

> 配套文档：[项目框架说明.md](项目框架说明.md)（前后端框架详解）、[代码说明文档.md](代码说明文档.md)（逐文件作用说明）、[执行记录.md](执行记录.md)（开发与验证记录）

## 技术栈

| 端 | 技术 |
|---|---|
| 前端 | Vue 3.5（script setup）+ Vite 5 + Element Plus 2.8 + Pinia 2 + ECharts 5 + dayjs + axios |
| 后端 | Java 17+ + Spring Boot 3.2.5 + MyBatis-Plus 3.5.7 + Sa-Token 1.38 + Hutool + Lombok |
| 数据库 | MySQL 8（库名 `pms`） |

## 功能一览

- **工作台**：项目统计卡片、阶段分布、近 12 月趋势、部门分布、待办/动态/最近项目
- **项目台账**：多条件筛选分页、新建项目（自动按模板初始化 24 节点）、编辑、作废、数据权限（admin 全部 / manager 本部门 / 其他仅本人）
- **项目详情**：项目信息、24 节点表、纯 CSS 甘特图、节点操作（开始/完成/延期/更换负责人）、跟进记录、附件上传下载
- **业务记录**：7 类记录的增删改查，与项目/节点联动
- **消息待办**：站内消息分页、未读角标、单条/全部已读、跳转项目
- **报表中心**：汇总 + 人员/部门/季度多维度统计图表
- **归档查询**：已结项项目与附件数检索
- **个人中心**：我的项目、待办节点、修改密码
- **系统设置**（admin）：用户/部门/角色/字典/操作日志/节点模板工期配置
- **定时任务**：每日 08:00 自动标记逾期节点、3 天内到期提醒（站内消息）

## 目录结构

```
XMguanli/
├── frontend/               # 前端（Vue3 + Vite，端口 5173）
│   ├── src/api/            # axios 封装 + 全量接口（Mock 双通道）
│   ├── src/mock/           # 演示数据与 Mock 处理器（离线可跑）
│   ├── src/views/          # 登录 + 9 个业务页面
│   └── vite.config.js      # /api 代理 → localhost:8080
├── backend/                # 后端（SpringBoot，端口 8080）
│   ├── src/main/java/com/company/pms/   # controller/service/mapper/entity/...
│   ├── sql/pms_schema.sql  # 建库建表 + 种子数据（一键导入）
│   └── src/main/resources/application.yml
├── 项目框架说明.md          # 框架与架构文档
├── 代码说明文档.md          # 每个代码文件的作用说明
└── 执行记录.md              # 开发进度与验证记录
```

## 快速开始

### 环境要求

- JDK 17+、Maven 3.8+
- Node.js 18+、npm 9+
- MySQL 8

### 1. 初始化数据库

```powershell
# 已内置 root/123456 账号假设；若不同请先改 backend/src/main/resources/application.yml
mysql -u root -p < backend/sql/pms_schema.sql
```

脚本会自动建库 `pms`、建 11 张表并写入种子数据（角色/部门/用户/字典/24 节点模板/3 个演示项目）。

### 2. 启动后端（端口 8080）

```powershell
cd backend
mvn spring-boot:run
```

### 3. 启动前端（端口 5173）

```powershell
cd frontend
npm install          # 首次运行
npm run dev          # 默认 Mock 模式，无需后端即可预览
```

联调真实后端：

```powershell
$env:VITE_USE_MOCK='false'; npm run dev
```

浏览器访问 http://localhost:5173 。

### 演示账号（密码统一 `123456`）

| 账号 | 角色 | 数据权限 |
|---|---|---|
| `admin` | 系统管理员 | 全部项目 + 系统设置 |
| `zhangwei` | 销售经理（manager） | 本部门项目 |
| `lina` / `zhoujie` | 销售 / 商务 | 仅本人相关 |
| `chengang` / `liuyang` | 项目经理 / 实施工程师 | 仅本人相关 |
| 其余（wangqiang/zhaomin/sunli/qianjin/wufang） | 方案/招投标/运维/交付/档案 | 仅本人相关 |

## 常用验证命令

```powershell
# 前端生产构建（9 页面编译校验）
cd frontend && npm run build

# 后端编译
cd backend && mvn -q -DskipTests clean compile
```

## 配置说明

| 配置 | 位置 | 说明 |
|---|---|---|
| Mock 开关 | `frontend/.env` 或环境变量 `VITE_USE_MOCK` | 默认 Mock；设为 `false` 走后端 |
| 代理目标 | `frontend/vite.config.js` | `/api` → `http://localhost:8080` |
| 数据库连接 | `backend/src/main/resources/application.yml` | 默认 `root/123456`，按需修改；URL 必须用 `characterEncoding=UTF-8` |
| 上传目录 | yml `pms.upload-path` | 默认 `uploads/`（相对后端运行目录），上传上限 50MB |
| Sa-Token | yml `sa-token` | token 名 `Authorization`、前缀 `Bearer`、有效期 7 天 |

## 接口约定

- 统一返回 `{ code, msg, data }`，`code === 200` 成功；分页 `data = { list, total, pageNum, pageSize }`
- 除登录外均需请求头 `Authorization: Bearer <token>`；未登录返回 401
- 全部约 35 个接口的路径/入参契约见 [项目框架说明.md](项目框架说明.md) 与 [代码说明文档.md](代码说明文档.md)

## 验证状态（2026-10-19）

- 后端 `mvn clean compile` ✅　前端 `npm run build` ✅　SQL 导入 MySQL 8.0.11 ✅
- 前后端真实联调全链路通过：登录/鉴权/数据权限/节点串行守卫/消息已读 ✅，浏览器端到端 9 页面 + 详情页渲染正常、控制台无报错 ✅
