# TikTok Shop US 利润分析 Skill

这是一个面向 TikTok Shop 美国站的可审计利润核算技能。它将结算、ERP、仓库、广告、采购和售后数据统一到订单与 SKU 两个粒度，输出店铺 P&L、SKU 利润、样品成本和跨系统对账结果。

## 使用方式

将 `tiktok-shop-us-profit` 目录安装为 Codex skill，或把该目录放入技能目录。上传报表后，可直接提出：

> 分析 2026 年 7 月 TikTok Shop US 的已知经营利润；按结算日期核算，广告账户消耗为 $8,116.41，并生成 SKU 排名、样品成本与 ERP/TK 对账。

## 文件结构

- [tiktok-shop-us-profit/SKILL.md](tiktok-shop-us-profit/SKILL.md)：可执行入口、工作流、核算防错规则。
- [TikTok_Shop_US_Full_Profit_Analysis_Skill(2).md](TikTok_Shop_US_Full_Profit_Analysis_Skill(2).md)：已确认的业务口径、SKU 成本与历史规则。
- [tiktok-shop-us-profit/templates/report-template.md](tiktok-shop-us-profit/templates/report-template.md)：标准交付报告表格。

## 核心原则

- 利润按实际销售 SKU、已确认成本逐单/逐 SKU 核算。
- TikTok Order ID 是跨系统唯一主键；先订单级对账，再看明细差异。
- 缺失成本不按 0 处理；估算与未纳入项目必须明示。
- Amazon 仓库渠道不进入 TikTok 利润；已停售 SKU 不混入当前经营决策。
