# 计算机维修社团 · 在线测评系统

两个页面 + 一个配置文件，部署到 GitHub Pages 即可公网使用：

| 文件 | 用途 |
|------|------|
| `index.html` | 学生答题页：班级/姓名/手机号 + 28 道单选，交卷自动评分并上传成绩 |
| `admin.html` | 教师后台：查看全部答卷、统计（人数/平均分/最高分）、按班级筛选、导出 CSV |
| `config.js` | 公共配置：云存储 Bin ID 和学生端写入密钥 |

学生成绩保存在 [JSONBin.io](https://jsonbin.io)（免费云存储），教师后台通过它读取数据。

---

## 部署教程（约 15 分钟）

### 第 1 步：注册 JSONBin 并获取 Master Key

1. 打开 https://jsonbin.io 用邮箱注册并登录（免费）。
2. 登录后进入 **API Keys** 页面，复制 **X-Master-Key**（形如 `$2a$10$xxxx...`）。
   这是你的管理密钥，**只用于教师后台，不要发给学生**。

### 第 2 步：创建数据仓库（Bin）并配置学生端

1. 打开 `admin.html`（本地双击即可），填入 Master Key，点 **"创建新数据仓库"**。
2. 页面会显示生成的 **Bin ID**，记下来。
3. 回到 jsonbin.io 的 **API Keys** 页面，点 **Create Access Key**：
   - 权限只勾选 **Read** 和 **Update**（不要勾 Delete/Create）；
   - 绑定到刚才创建的 Bin（可选项，更安全）；
   - 复制生成的 Access Key。
4. 用记事本打开 `config.js`，填入：
   ```js
   var CONFIG = {
     BIN_ID: "你的BinID",
     ACCESS_KEY: "$2a$10$你的AccessKey"
   };
   ```
   保存。学生端即配置完成。

> 为什么这样设计：学生端只持有低权限 Access Key（只能读/更新这一个 Bin，删不掉数据）；
> Master Key 只在教师后台使用，且建议教师在本地打开 admin.html 使用，密钥不经过公网。

### 第 3 步：发布到 GitHub Pages

1. 注册并登录 https://github.com。
2. 右上角 **+** → **New repository**，名称如 `club-quiz`，选 **Public**，点 **Create repository**。
3. 在仓库页点 **uploading an existing file**，把本文件夹中的
   `index.html`、`admin.html`、`config.js` 三个文件一起拖进去，点 **Commit changes**。
4. 进入仓库 **Settings** → 左侧 **Pages**：
   - Source 选 **Deploy from a branch**；
   - Branch 选 **main**、目录 **/ (root)**，点 **Save**。
5. 等待 1~2 分钟，刷新页面会显示发布地址：
   ```
   https://你的用户名.github.io/club-quiz/
   ```

### 第 4 步：开始使用

- **学生答题链接**（发到班级群）：
  `https://你的用户名.github.io/club-quiz/`
- **教师后台**（自己收藏，勿外传）：
  `https://你的用户名.github.io/club-quiz/admin.html`
  打开后输入 Master Key 和 Bin ID 即可查看成绩。
  更安全的做法：把 `admin.html` 下载到本地电脑直接双击打开使用。

---

## 常见问题

- **交卷提示"成绩提交失败"**：检查 config.js 里 BIN_ID / ACCESS_KEY 是否填对、Access Key 是否勾了 Read + Update 权限；改完需重新上传 config.js 并等 Pages 更新。
- **学生端能否防作弊**：静态页面无法完全防作弊，建议在教室环境集中组织测评。
- **修改题目**：题库在 index.html 的 `QUESTIONS` 数组中，每题 `{c:模块, t:题干, o:[四个选项], a:正确答案下标0-3}`；改完重新上传即可。
- **两个学生同时交卷会不会冲突**：JSONBin 按提交顺序写入，极小概率并发覆盖，班级规模使用基本无影响；如遇个别丢失可让学生截图补录。
- **数据安全**：Access Key 具备读取成绩的能力，若担心信息泄露，可在手机号等字段上做脱敏，或仅用教师本地版后台收集。
