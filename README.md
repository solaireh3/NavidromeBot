# NavidromeBot

基于 Telegram 的 [Navidrome](https://www.navidrome.org/) 账号管理机器人，支持注册码分发、用户管理、签到保号、数据库备份等功能，通过 Docker 一键部署。

---

## 功能特性

- **账号管理** — 创建、删除 Navidrome 账号，查看用户信息与白名单
- **注册码系统** — 生成专属邀请码，支持定向分发给指定用户
- **开放注册** — 管理员可按名额开放/关闭公开注册入口
- **线路管理** — 新增、修改、删除音乐服务线路
- **签到保号** — 用户定期签到以保持账号活跃，超期自动清理
- **强制入群** — 注册用户须加入指定 Telegram 群组
- **消息广播** — 向全体用户或指定群组推送图文消息
- **数据库备份/恢复** — 定时备份 MongoDB，支持一键恢复并同步 Navidrome
- **任务控制** — 运行时开关定时备份、签到保号等后台任务
- **管理面板** — Bot 内嵌图形化管理界面，无需命令行操作

---

## 效果预览

| 主面板 | 管理面板 | 任务控制 |
|:---:|:---:|:---:|
| ![主面板](imgs/start_panel.jpg) | ![管理面板](imgs/admin_panel.jpg) | ![任务控制](imgs/task_panel.jpg) |

---

## 快速部署

### 方式一：同时启动 MongoDB + Bot（推荐新手）

适用于尚未安装 MongoDB 的场景，`docker-compose.yml` 已内置 MongoDB 服务。

**1. 创建目录并下载所需文件**

```bash
mkdir -p navidrome_bot/Navidrome/db_backup navidrome_bot/Navidrome/logs
cd navidrome_bot
curl -o docker-compose.yml https://raw.githubusercontent.com/dddddluo/NavidromeBot/main/docker-compose.yml
curl -o Navidrome/config.json https://raw.githubusercontent.com/dddddluo/NavidromeBot/main/Navidrome/config.json
```

**2. 编辑配置文件**

参考下方 [配置说明](#配置说明) 填写 `Navidrome/config.json`。

> **注意：** 同时修改 `docker-compose.yml` 中 MongoDB 的用户名（`MONGO_INITDB_ROOT_USERNAME`）和密码（`MONGO_INITDB_ROOT_PASSWORD`），并确保与 `config.json` 中的 `DB_URL` 保持一致。如无需从外部访问 MongoDB，建议删除 `ports` 映射或改为 `127.0.0.1:27017:27017`。

**3. 启动服务**
```bash
docker-compose up -d
```

---

### 方式二：已有 MongoDB，仅启动 Bot

**1. 创建目录并准备配置文件**

```bash
mkdir -p navidrome_bot/Navidrome/db_backup navidrome_bot/Navidrome/logs
cd navidrome_bot
```

下载配置文件模板并按 [配置说明](#配置说明) 填写，将 `DB_URL` 指向已有的 MongoDB 实例：

```bash
curl -o Navidrome/config.json https://raw.githubusercontent.com/dddddluo/NavidromeBot/main/Navidrome/config.json
```

**2. 拉取镜像并启动**
```bash
docker pull dddddluo/navidrome_bot:latest

docker run -d \
  --name navidrome_bot \
  --restart always \
  -v ./Navidrome/config.json:/app/Navidrome/config.json \
  -v ./Navidrome/db_backup:/app/Navidrome/db_backup \
  -v ./Navidrome/logo.jpg:/app/Navidrome/logo.jpg \
  -v ./Navidrome/logs:/app/Navidrome/logs \
  dddddluo/navidrome_bot:latest
```

---

## 配置说明

编辑 `Navidrome/config.json`，参考以下说明填写各项。

```jsonc
{
  // ===== Telegram Bot =====
  "TELEGRAM_BOT_TOKEN": "从 @BotFather 获取的 Bot Token",
  "TELEGRAM_BOT_NAME": "bot_username（不含@）",
  "OWNER": 123456789,                  // 接收备份文件的管理员 TG ID
  "ADMIN_ID": [123456789, 987654321],  // 管理员 TG ID 列表
  "ALLOWED_GROUP_IDS": [-1001234567890], // 强制加入的群组 ID 列表
  "GROUP_INVITE_LINK": "https://t.me/your_group", // 群组邀请链接
  "LOG_GROUP_ID": -1001234567890,      // 日志推送群组 ID（可选）

  // ===== MongoDB =====
  "DB_NAME": "bot",
  "DB_URL": "mongodb://user:password@127.0.0.1:27017/bot?authSource=admin",
  "DB_BACKUP_DIR": "/app/Navidrome/db_backup",
  "DB_BACKUP_RETENTION_DAYS": 7,       // 备份文件保留天数
  "BACKUP_DB_ENABLE": true,            // 是否开启定时自动备份

  // ===== Navidrome API =====
  "API_BASE_URL": "http://your-navidrome-host:4533", // 结尾不要加 /
  "NA_ADMIN_USERNAME": "admin",
  "NA_ADMIN_PASSWORD": "your_password",

  // ===== 签到保号 =====
  "TIME_USER_ENABLE": true,            // 是否开启签到保号
  "TIME_USER": "7d",                   // 保号周期，支持 s/m/h/d（秒/分/时/天）

  // ===== 外观 =====
  "START_PIC": ""  // Bot 欢迎图片，可填 HTTP 链接或本地路径，留空使用内置默认图片
}
```

---

## 管理员命令

| 命令 | 说明 |
|------|------|
| `/start` | 打开主面板 |
| `/help` | 查看帮助信息 |
| `/new_code [数量]` | 生成注册码，默认 1 个；回复用户消息可定向生成 |
| `/list_code` | 查看所有注册码 |
| `/del_user <tgid>` | 删除指定用户的 Navidrome 账号（可回复消息操作） |
| `/new_line <名称> <地址>` | 新增或修改线路 |
| `/del_line <名称>` | 删除线路 |
| `/mm` | 查看用户信息 |
| `/na_token` | 手动刷新 Navidrome Token |

---

## 管理面板功能

通过 `/start` → **管理面板** 进入，支持以下操作：

- 👥 查看用户 / 🎖 查看白名单
- ❗️ 删除未签到用户 / ⛔️ 删除离群用户
- 🔖 查看注册码
- 📚 备份数据库 / 💾 恢复数据库
- ✅ 开放注册 / ❎ 关闭注册
- 📢 消息广播 / 🗑️ 删除广播
- ⚙️ 任务控制（开关定时任务）

---

## 升级

### 使用 docker-compose

```bash
docker pull dddddluo/navidrome_bot:latest
docker-compose down && docker-compose up -d
```

如只想重启 bot 容器而不影响 MongoDB：

```bash
docker-compose up -d --no-deps navidrome_bot
```

### 使用 docker

```bash
docker pull dddddluo/navidrome_bot:latest
docker stop navidrome_bot && docker rm navidrome_bot
docker run -d \
  --name navidrome_bot \
  --restart always \
  -v ./Navidrome/config.json:/app/Navidrome/config.json \
  -v ./Navidrome/db_backup:/app/Navidrome/db_backup \
  -v ./Navidrome/logo.jpg:/app/Navidrome/logo.jpg \
  -v ./Navidrome/logs:/app/Navidrome/logs \
  dddddluo/navidrome_bot:latest
```

> 升级不会影响已有数据，`config.json`、数据库备份及日志均通过卷挂载持久化在宿主机。

---

[!["Buy Me A Coffee"](https://github.com/user-attachments/assets/3a24f81e-ebc1-4244-9a01-26aecb466214)](https://ko-fi.com/dddddluo)
