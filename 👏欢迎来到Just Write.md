JustWrite 是由@**西郊次生林**（*微信公众号|少数派|知乎同名*）定制的一款 **Obsidian** 写作库，专为**文章**以及**小说作者**准备。

具体使用方法详见 [JustWrite使用教程](JustWrite使用教程.md)。
# 🖋️ 写作中心
```dataviewjs
let ftMd = dv.pages("").file.sort(t => t.cday)[0]
let total = parseInt([new Date() - ftMd.ctime] / (60*60*24*1000))
let totalDays = " 您已使用 *JustWrite* "+total+" 天，"
let nofold = '!"misc/templates"'
let allFile = dv.pages(nofold).file
let totalTag = allFile.etags.distinct().length+" 个标签"

dv.paragraph(
	totalDays+"一共 "+totalTag+"。"
)
```
🎎**我的计划**
- ✨ [写作计划](写作计划.md)

> [!GRID] 发布中心
> **🚀 一键发布**
> - 🐱‍🐉 [公众号](https://mp.weixin.qq.com/cgi-bin/loginpage)
> - 🎃 [知乎](https://www.zhihu.com/)
> - 📰 [少数派](https://sspai.com/)
# 📊数据板
```dataviewjs
// 配置：在此处设置要统计的目标文件夹路径（注意大小写敏感）
const targetFolder = "项目";

// 获取目标文件夹及其所有子文件夹中的文件
const allFiles = dv.pages(`"${targetFolder}"`)
    .file
    // 精确过滤逻辑
    .filter(file => {
        const isMarkdownFile = file.path.endsWith(".md"); // 确认是MD文件
        const isIndexFile = file.name === "Index"; // 兼容大小写
        const isTargetFolderFile = file.path.startsWith(targetFolder + "/"); // 确保在目标文件夹内
        
        return isMarkdownFile && !isIndexFile && isTargetFolderFile;
    });

// 统计已发布文章（published 字段非空）
const publishedArticles = allFiles
    .map(file => {
        const frontmatter = dv.page(file.path).file?.frontmatter;
        const publishData = frontmatter?.published;
        
        // 标准化发布平台数据格式
        let platforms = [];
        if (Array.isArray(publishData)) { // YAML数组格式
            platforms = publishData.filter(p => p.trim());
        } else if (typeof publishData === "string") { // 逗号分隔字符串
            platforms = publishData.split(/,\s*/).filter(p => p.trim());
        } else if (publishData) { // 单个平台
            platforms = [publishData.toString()];
        }
        
        return { file, platforms };
    })
    .filter(article => article.platforms.length > 0); // 只保留已发布文章

// 统计文章数量
const articleCount = allFiles.length;

// 输出结果
dv.paragraph(`📂 当前，**${targetFolder}** 中共写作有 **${articleCount}** 篇文章，其中 **${publishedArticles.length}** 篇已发布。`);
```
```dataviewjs
// 配置：在此处设置要统计的目标文件夹路径（注意大小写敏感）
const targetFolder = "项目";

// 创建存储文件夹数据的对象
let folderData = {};

// 获取目标文件夹及其所有子文件夹中的文件
const files = dv.pages(`"${targetFolder}"`);

// 异步处理所有文件
await Promise.all(files.map(async (file) => {
    // 排除文件夹本身
    if (!file.file.link.path.includes("/")) return;
    
    // 获取文件相对路径并分割层级
    const relativePath = file.file.path.replace(targetFolder + "/", "");
    const pathSegments = relativePath.split("/");
    
    // 当文件位于子文件夹时（路径深度>1）
    if (pathSegments.length > 1) {
        // 获取完整子文件夹路径
        const folderPath = pathSegments.slice(0, -1).join("/");
        
        // 读取文件内容（自动跳过空文件）
        const content = await dv.io.load(file.file.path);
        
        // 清理内容和统计字符
        const cleanContent = content
            .replace(/^---[\s\S]*?---/, "")  // 移除YAML frontmatter
            .replace(/(\r\n|\n|\r)/gm, "")  // 移除换行符
            .replace(/\s+/g, "");           // 移除所有空白字符
        
        // 统计严格字符数（包括中日韩文字、英文字母、数字、标点）
        const charCount = [...cleanContent].length;

        // 累加到对应文件夹
        folderData[folderPath] = (folderData[folderPath] || 0) + charCount;
    }
}));

// 将对象转换为可排序的数组
const sortedFolders = Object.entries(folderData)
    .sort((a, b) => b[1] - a[1])
    .map(([folder, count]) => [folder, count]);

// 输出结果表格
dv.table(
    ["项目", "字符数统计"],
    sortedFolders
);
```
