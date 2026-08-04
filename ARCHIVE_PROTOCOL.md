# 自动化归档协议（ARCHIVE_PROTOCOL）

供各定时自动化任务在生成主报告后调用，把产物归档到本地 Git 仓库并同步离线备份。

仓库根目录：C:/Users/cp9wo/investment-notes
离线备份：C:/Users/cp9wo/investment-notes-backup.bundle 与 D:/百度云/investment-notes-backup.bundle

## 前置安全约束（最高优先级）
1. 本仓库须为 Private。若执行时判断仓库仍为公开，立即跳过 push，仅在报告末尾写：
   "⚠️ 仓库仍为公开，已跳过 GitHub 推送，请先在 GitHub 设为 Private"；
   本地落盘与百度网盘 bundle 照常完成（不 push）。
2. 若 push 因网络不通 / 鉴权失效等失败：不重试、不抛错、不中断主报告，
   仅在报告末尾追加告警行（如"⚠️ GitHub 归档未成功：<原因>，本地与百度网盘备份不受影响，下次运行再试"）。
3. 归档是"附加动作"，绝不影响主报告的交付与质量。

## 归档步骤（尽力而为）
1. 定位本次任务生成的所有产物文件（具体文件名由调用方在自动化 prompt 中指明）。
2. 复制到仓库对应子目录：
   - 研究报告类（简报/晨报/月报/周报/分析）→ research/
   - 持仓组合类 → portfolio/
   - 观察清单类 → watchlist/
   （子目录不存在则新建）
3. 在仓库目录执行（用 Windows 路径，Git Bash 亦可识别）：
   git -C "C:/Users/cp9wo/investment-notes" add -A
   git -C "C:/Users/cp9wo/investment-notes" commit -m "自动化归档：<任务名> <YYYY-MM-DD>"
   git -C "C:/Users/cp9wo/investment-notes" push origin main
   （若 add -A 后无实际变更，commit 会报"nothing to commit"，属正常，跳过即可，不视为错误。）
4. 重新生成离线备份包并同步百度网盘：
   git -C "C:/Users/cp9wo/investment-notes" bundle create investment-notes-backup.bundle --all
   将生成的 investment-notes-backup.bundle 移动到 C:/Users/cp9wo/investment-notes-backup.bundle
   复制一份到 D:/百度云/investment-notes-backup.bundle（百度网盘 PC 客户端自动上传）
5. 完成。一切正常则无需在主报告额外声明；有告警按上文在报告末尾追加。

## 设计原则
- GitHub 只是多一份镜像；本地仓库 + 百度网盘 bundle 才是真正兜底。
- 任一步失败都只告警、不阻断，保证主报告永远先交付。
