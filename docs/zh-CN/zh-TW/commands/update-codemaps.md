# 更新程序代码地圖

分析程序代码库结构并更新架构文件：

1. 掃描所有原始档案的 imports、exports 和相依性
2. 以下列格式产生精简的程序代码地圖：
   - codemaps/architecture.md - 整体架构
   - codemaps/backend.md - 后端结构
   - codemaps/frontend.md - 前端结构
   - codemaps/data.md - 资料模型和结构描述

3. 计算与前一版本的差异百分比
4. 如果变更 > 30%，在更新前请求使用者批准
5. 为每个程序代码地圖新增新鮮度时间戳
6. 将報告保存到 .reports/codemap-diff.txt

使用 TypeScript/Node.js 进行分析。专注於高阶结构，而非实作细节。
