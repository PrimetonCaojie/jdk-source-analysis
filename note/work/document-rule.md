## 规范文档

为保障文章输出质量，在编辑文档前请先阅读以下链接内容

### 1、文档命名规范

**分析文档** 命名规则为 类名_分析.md（示例：ReentrantLock_分析.md）

**文档配图** 命名规则尽量贴合

**代码分支** github账号名称_月份_branch（示例：mf_mar_branch）

### 2、文档归类规范

**分析文档** 存放路径与类所在package一致

**文档配图** 在jdk-source-analysis/note/doc中创建与类所在package相同层级的文件夹存放（示例：*java/util/concurrent/locks/ReentrantReadWriteLock/读写锁状态划分方式.jpg*）

### 3、代码提交规范

尽量提交有意义的提交注释内容(清楚完整描述所做的文档修改内容)

### X、其它

有歧义类的文章(非源码但重要，例如synchronized、volatile)：存放路径为jdk-source-analysis/note/analysis

