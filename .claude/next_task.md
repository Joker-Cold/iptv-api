# 港澳台 IPTV 更新 — 运维参考

## 当前状态（2026-03-29 已完成首次运行）

Docker 容器 `iptv-api` 已成功启动并完成首次测速。容器会每天自动更新一次。

### 首次运行结果
- 120 个接口参与测速，严格通过测速的有 6 个台湾频道
- 补偿机制保留了更多频道（香港、澳门、台湾均有结果）
- 主要瓶颈：国内网络访问港澳台源延迟高（15s+），大部分源未通过严格测速

## 项目配置概览

### 频道模板 `config/user_demo.txt`
根据 `F:\iptv_git\COLD_OK.m3u8` 中实际存在的频道创建，分 5 个分类：
- 📺香港频道 (33个)：TVB 系列、ViuTV、HOY、凤凰、港台电视、NowTV 等
- 🏀香港体育 (16个)：Now Sports、SPOTV、beIN SPORTS、ELTA 等
- 🎬香港影视 (9个)：HBO、AXN、Animax、AT-X、BBC Earth 等
- 🎰澳门频道 (2个)：澳视澳门、澳门资讯
- 🌸台湾频道 (19个)：TVBS 系列、八大、三立、中天、纬来、大爱等

### 用户配置 `config/user_config.ini`
针对港澳台源优化：
- 降低分辨率/速率门槛（海外源质量参差不齐）
- 每频道保留 8 个接口（源不稳定，多留备用）
- 放宽测速超时至 15s
- 开启补偿机制（避免结果为空）

### 本地数据源 `config/local/COLD_OK.m3u8`
已将 `F:\iptv_git\COLD_OK.m3u8` 复制到 `config/local/` 目录。

### 频道别名 `config/alias.txt`
在末尾添加了港澳台频道的正则别名，匹配 COLD_OK.m3u8 中的各种命名格式（繁体、英文、带[直播]后缀等）。

## Docker 运行环境

### 容器启动命令
```bash
docker run -d \
  -p 80:8080 \
  -v f:/iptv_git/iptv_api/config:/iptv-api/config \
  -v f:/iptv_git/iptv_api/output:/iptv-api/output \
  -e PUBLIC_DOMAIN=127.0.0.1 \
  -e PUBLIC_PORT=80 \
  --name iptv-api \
  guovern/iptv-api:latest
```

### Docker 镜像加速
`C:\Users\Lenovo\.docker\daemon.json` 已配置多个国内镜像源（DaoCloud、百度、南大、上交、中科大等）。拉取镜像时如仍失败，需开启 VPN/代理。

### 访问地址
- M3U 播放列表：http://127.0.0.1/m3u
- TXT 格式：http://127.0.0.1/txt
- 结果文件：`output/user_result.txt`

## 日常管理命令

| 操作 | 命令 |
|------|------|
| 查看容器状态 | `docker ps` |
| 查看日志（最近 20 行） | `docker logs iptv-api --tail 20` |
| 手动触发更新（重启容器） | `docker restart iptv-api` |
| 停止容器 | `docker stop iptv-api` |
| 启动已停止的容器 | `docker start iptv-api` |
| 删除容器（需先停止） | `docker rm -f iptv-api` |
| 更新镜像版本 | `docker pull guovern/iptv-api:latest && docker rm -f iptv-api`，然后重新执行启动命令 |

## 自动更新机制

- 容器内置定时任务，**每天自动更新一次**
- 只要 Docker Desktop 保持运行，无需手动操作
- 电脑重启后 Docker Desktop 会自动启动，容器也会自动恢复运行
- 修改 `config/` 下的配置文件后，`docker restart iptv-api` 即可生效（volume 挂载，无需重建）

## 可能的优化方向

1. **提高有效频道数**：在 `user_config.ini` 中进一步放宽速率门槛（如 `min_speed` 降到 0.05）或增加超时时间
2. **更新本地数据源**：如获取到新的 m3u8 文件，替换 `config/local/COLD_OK.m3u8` 后重启容器
3. **添加订阅源**：在 `config/subscribe.txt` 中添加在线订阅地址，可获取更多源（当前为空，仅使用本地源）
4. **网络优化**：如有代理/VPN，可在 Docker Desktop → Settings → Resources → Proxies 中配置，让容器通过代理访问港澳台源，大幅提升测速通过率
