# Steam 状态监控插件V3

## 访问统计

本插件是专为AstrBot设计的插件，用于定时轮询 Steam Web API，监控指定玩家的在线/离线/游戏状态变更，并在状态变化时推送通知。支持多 SteamID 监控，自动记录游玩日志，支持群聊分组，数据持久化，支持丰富指令。

## 功能特性
- 支持定时轮询多个 SteamID 的状态，分群管理，每个群聊可独立配置监控玩家
- 检测玩家上线、下线、开始/切换/退出游戏等状态变更，自动推送游戏启动/关闭提醒
- 成就变动自动推送提醒
- **头像框渲染**：开始游戏/结束游戏/list/rank 均支持 Steam 头像框，本地优先缓存 7 天
- **游戏时长排行榜**：支持  / ，按数字天数查询，凌晨 4:00 天分界
- 游戏时长排行榜：支持  本群排行和  所有群排行，可按数字天数查询
- 智能轮询 + 固定轮询双模式可切换，默认为1-30分钟查询一次状态，取决于steam的上次在线时间
- 持久化记录玩家游玩日志，重启bot后状态不会丢失
- **批量查询优化**：采用 Steam 官方批量接口（单次最多 100 个 ID），大幅降低 API 调用次数，从根本上避免触发 Steam 限流（HTTP 429 / x-eresult: 84）
- **多种 ID 输入格式**：`addid` 现支持 SteamID64、个人资料链接、自定义 vanity URL、`s.team` 短链、8 位好友码等多种格式
- **通知开关精细化**：可独立控制游戏结束通知、成就推送、以及图片/文本推送方式
- **网络代理支持**：可配置 http / https / socks5 代理，改善网络环境下的数据获取稳定性
- **字体自动管理**：自动检测并加载插件 `fonts` 目录下的 NotoSansHans 系列字体，渲染更稳定
- **性能优化**：节流写盘、单点异常隔离、批量预拉取，避免拖慢 AstrBot 主进程与 WebUI

## 默认轮询间隔说明（智能轮询模式）
| 玩家最近在线时间      | 轮询间隔 |
|----------------------|---------|
| 游戏中               | 1分钟   |
| 12分钟内             | 3分钟   |
| 12分钟~3小时         | 5分钟   |
| 3小时~24小时         | 10分钟  |
| 24~48小时            | 20分钟  |
| 超过48小时           | 30分钟  |

## 快速上手
1. 在AstrBot网页后台的配置中配置 Steam_Web_API_Key：[点击获取](https://steamcommunity.com/dev/apikey)
2. 在AstrBot网页后台的配置中配置 SGDB_API_KEY（用于获取封面图，可选）：[点击获取](https://www.steamgriddb.com/profile/preferences/api)
3. 在需要进行提醒的群聊输入指令添加要监控的玩家（以下格式均支持）：
   - `/steam addid 7656119xxxxxxxxx`（SteamID64）
   - `/steam addid https://steamcommunity.com/profiles/7656119xxxxxxxxx`（个人资料链接）
   - `/steam addid https://steamcommunity.com/id/customname`（自定义 vanity URL）
   - `/steam addid https://s.team/p/7656119xxxxxxxxx`（s.team 短链）
   - `/steam addid 123456789`（8 位好友码）
4. 启动轮询：
   `/steam on`  启动本群 Steam 状态监控，后续状态变更会自动推送。

## 配置项说明
| 配置项 | 说明 | 默认值 |
|-------|------|-------|
| `steam_api_key` | Steam Web API Key | — |
| `sgdb_api_key` | SteamGridDB API Key（用于封面图） | — |
| `fixed_poll_interval` | 固定轮询间隔（秒），为 0 时使用智能轮询 | 0 |
| `smart_poll_intervals` | 智能轮询各状态间隔（分钟，逗号分隔） | `1,3,5,10,20,30` |
| `retry_times` | Steam API 请求重试次数 | 3 |
| `max_group_size` | 单群最大监控人数 | 20 |
| `detailed_poll_log` | 详细轮询日志开关 | true |
| `enable_achievement_poll` | 成就轮询推送开关 | true |
| `enable_game_end_notify` | 游戏结束通知开关 | true |
| `notify_send_image` | 通知发送图片开关 | true |
| `notify_send_text` | 通知发送文本开关 | true |
| `enable_proxy` | 启用网络代理 | false |
| `proxy_url` | 代理链接（如 `http://127.0.0.1:7890`） | 空 |

> 带「修改后重启AstrBot生效」标注的配置项需重启后生效。

## 注意事项
- 获取速度与是否成功获取 Steam 数据取决于网络环境。建议通过加速或代理（现已内置代理配置项）来保证稳定的查询状态。
- 如果出现未知的轮询错误可以使用 `/steam clear_allids` 来清除所有群聊的轮询 id。
- 修改插件参数后，如果出现重复通知的情况，请不要重载插件，而是重启 AstrBot。
- 如果出现未知的无法提醒，但轮询显示正常的情况，请使用 `/steam on/off` 进行修复。
- 监控人数较多时，建议适当调高 `max_group_size` 并保持智能轮询，以兼顾时效与 Steam 限流。

## 指令列表
- `/steam on` 启动本群Steam状态监控
- `/steam off` 停止本群Steam状态监控
- `/steam list` 列出本群所有玩家当前状态
- `/steam alllist` 列出所有群聊分组及玩家状态
- `/steam config` 查看当前插件配置
- `/steam set [参数] [值]` 设置配置参数（如 `/steam set poll_interval_sec 30`）
- `/steam addid [SteamID/链接/好友码]` 添加玩家到本群监控列表（支持多种格式）
- `/steam delid [SteamID]` 从本群监控列表删除SteamID
- `/steam push_group [SteamID]` 添加id到联动推送的副群（轮询一次通知多个群聊）
- `/steam delpush_group [SteamID]` 删除id联动推送的副群
- `/steam openbox [SteamID]` 查看指定SteamID的全部详细信息
- `/steam rank [天数]` 查看本群游戏时长排行榜（默认今日，可指定天数）
- `/steam allrank [天数]` 查看所有群游戏时长排行榜（默认今日，可指定天数）
- `/steam rank_on` 开启每日排行榜自动推送
- `/steam rank_off` 关闭每日排行榜自动推送
- `/steam rs` 清除所有状态并初始化
- `/steam achievement_on` 开启本群Steam成就推送
- `/steam achievement_off` 关闭本群Steam成就推送
- `/steam test_achievement_render [steamid] [gameid] [数量]` 测试成就图片渲染
- `/steam test_game_start_render [steamid] [gameid]` 测试开始游戏图片渲染
- `/steam清除缓存` 清除所有头像、封面图等图片缓存
- `/steam help` 显示所有指令帮助

## 依赖
- Python 3.7+
- httpx
- Pillow
- AstrBot 框架