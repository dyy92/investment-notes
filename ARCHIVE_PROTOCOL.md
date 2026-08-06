# 自动化归档协议（ARCHIVE_PROTOCOL）

供各定时自动化任务在生成主报告后调用，把产物归档到【私有】Git 仓库并同步离线备份。

## 仓库分工（重要）
- **公开仓** `investment-notes`（C:/Users/cp9wo/investment-notes）：仅放非敏感模板/示例/本协议指令本身。
- **私有仓** `investment-notes-private`（C:/Users/cp9wo/investment-notes-private）：存放敏感内容——8 个自动化产出的宽基/黄金/CPI 研究、"客户沟通版"话术，以及主理人对话产出的投资分析文档。

私有仓根目录：C:/Users/cp9wo/investment-notes-private
私有仓离线备份：C:/Users/cp9wo/investment-notes-private-backup.bundle 与 D:/百度云/investment-notes-private-backup.bundle

## 前置安全约束（最高优先级）
1. 归档目标必须是私有仓 `investment-notes-private`。push 前 sanity-check：remote `origin` 的 SSH 地址须包含 `investment-notes-private`；
   若误指向公开仓 `investment-notes`，立即中止推送，并在报告末尾告警：
   "⚠️ 归档目标疑似公开仓，已中止推送以防泄露"，仅完成本地落盘 + 百度网盘 bundle。
2. 若 push 因网络不通 / 鉴权失效等失败：不重试、不抛错、不中断主报告，仅在报告末尾追加告警行
   （如"⚠️ GitHub 私有仓归档未成功：<原因>，本地与百度网盘备份不受影响，下次运行再试"）。
3. 归档是"附加动作"，绝不影响主报告的交付与质量。

## 归档步骤（尽力而为）
1. 定位本次任务生成的所有产物文件（具体文件名由调用方在自动化 prompt 中指明）。
2. 复制到私有仓对应子目录：
   - 研究报告类（简报/晨报/月报/周报/分析）→ research/
   - 持仓组合类 → portfolio/
   - 观察清单类 → watchlist/
   （子目录不存在则新建）
3. 在私有仓目录执行（用 Windows 路径，Git Bash 亦可识别）：
   git -C "C:/Users/cp9wo/investment-notes-private" add -A
   git -C "C:/Users/cp9wo/investment-notes-private" commit -m "自动化归档：<任务名> <YYYY-MM-DD>"
   git -C "C:/Users/cp9wo/investment-notes-private" push origin main
   （若 add -A 后无实际变更，commit 报"nothing to commit"属正常，跳过即可，不视为错误。）
4. 重新生成离线备份包并同步百度网盘：
   git -C "C:/Users/cp9wo/investment-notes-private" bundle create investment-notes-private-backup.bundle --all
   将生成的 investment-notes-private-backup.bundle 移动到 C:/Users/cp9wo/investment-notes-private-backup.bundle
   复制一份到 D:/百度云/investment-notes-private-backup.bundle（百度网盘 PC 客户端自动上传）
5. 完成。一切正常则无需在主报告额外声明；有告警按上文在报告末尾追加。

## 设计原则
- GitHub 只是多一份镜像；本地仓库 + 百度网盘 bundle 才是真正兜底。
- 任一步失败都只告警、不阻断，保证主报告永远先交付。
- 敏感内容只进私有仓；公开仓永不承载研究 / 客户话术。
