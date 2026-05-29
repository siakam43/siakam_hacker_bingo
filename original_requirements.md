# 原始需求

## 整体说明
1. 我需要开发一个单纯由markdown文档组成的skill, 名字叫siakam-hacker-bingo
2. 核心功能：逐个分析目标项目中的每一个.c文件和.h文件，挖掘安全问题并记录下来。

## 流程与架构
### 0 工具输入
1. SKILL使用方法：/siakam-hacker-bingo  project_dir 
2. 参数project_dir为待分析项目路径，如果用户没有指定该参数，默认为当前文件夹。
3. 在project_dir中，如果存在.siakamignore文件，需要将该文件中指定的子目录或文件排除掉，不进行分析。.siakamignore文件语法和.gitignore语法一样。
4. SKILL的输出路径为project_dir中的.siakam_out/BINGO文件夹



### 步骤1：建立任务列表与任务分配
1. 扫描project_dir中的全部源码文件（.c + .h），排除.siakamignore文件中指定跳过的文件。
2. 根据项目的文件夹结构，合理分配子任务，保证每个子任务不会分析过多文件，保证不遗漏且不重复分析任何源码文件: 建议每个最小子文件夹作为一个子任务。如果一个父文件夹同时包含源码和子文件夹，可以单独作为一个子任务，只分析父文件夹中的源码，子文件夹由其它子任务负责。
3. 在.siakam_out/BINGO文件夹中建立任务列表文件，记录每一个子任务负责的文件列表，跟踪每一个文件的分析状态。
4. 主agent根据任务列表顺序，启动sub-agent进行每个任务目标的安全分析。
5. 任务数量会很多，一定要求LLM不要偷懒，逐个子任务认真分析，没有效率要求，但有质量要求。
6. SKILL需要设置一个并发变量（方便人工调整），决定同时有几个subagent在并行分析。
7. 当全部subagent完成分析，将全部结果文件汇总到一个文件中。保存在.siakam_out/BINGO文件夹中
8. 任务量大，SIKLL需要有中断恢复机制

### 步骤2：安全分析
1. 每个subagent会有一个源码列表，必须逐个文件依次认真分析，不要跳过任何代码。同样没有效率要求，但有质量要求。任务过多时不要逐渐降低分析质量。
2. 分析的业务全部是c语言代码。目标包括：linux内核驱动模块，手机主核启动链（uefi,bl31,bl2,xloader等），手机小核固件（isp，sensorhub，gpu等） 
3. 不关心数据库，web，隐私泄露，内存泄漏，ddos等方面的安全问题。不关心性能问题和稳定性问题
4. 重点关注的安全问题类别
- 外部输入未校验: Missing validation on all external inputs (interface parameter, file, IPC, shared memory); Shared memory data read without integrity/safety checks; Trust boundary violations (trusting data from untrusted sources); Missing bounds checking on data from external sources; Type confusion from unvalidated input; Deserialization of untrusted data without validation; Missing validation on inter-process communication (IPC) data; Environment variable injection from external sources
- 内存安全：Buffer overflows (stack-based, heap-based); Out-of-bounds read/write (array index vulnerabilities); Use-after-free (UAF) vulnerabilities; Double free vulnerabilities; Null pointer dereference; Uninitialized memory access; Integer overflow/underflow leading to memory corruption; Format string vulnerabilities
- 系统安全:Authentication bypass logic; Privilege escalation paths; Remote code execution; dynamic code execution
- 密码安全:Hardcoded API keys, passwords, or tokens; Weak cryptographic algorithms or implementations; Improper key storage or management; Cryptographic randomness issues; Certificate validation bypasses
- 多线程竞争场景下的内存安全问题
5. 如果发现安全问题，每个subagent单独记录一个结果文件。每个安全问题需要记录代码范围，漏洞类型，漏洞描述。

## 要求
1. 需要使用代码仓探索的常用工具，SKILL应提前申请权限。
2. 本SKILL全部功能由LLM完成分析，在Agent(claude/opencode)调用skill时，遵循skill的说明知道LLM工作。本模块禁止开发使用任何脚本和代码！ 
3. LLM应尽最大努力，精准分析每一个任务，做出最合理的判断。不要牺牲质量来提升效率。
5. 整个skill的提示词都用英文。
6. 漏洞分析原则：Better to miss some theoretical issues than flood the report with false positives. Each finding should be something a security engineer would confidently raise in a PR review.


