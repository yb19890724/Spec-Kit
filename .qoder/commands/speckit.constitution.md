---
description: 根据交互式或提供的原则输入创建或更新项目章程，确保所有相关的模板保持同步。
handoffs:
   - label: 构建规格
     agent: speckit.specify
     prompt: 基于更新后的章程实施功能规格说明。我想构建...
---
## 用户输入

```text
$ARGUMENTS
```
在继续之前，您**必须**考虑用户输入（如果不为空）。

--- 
## 1.1 开始 `RESUME` 检查
- **在开始设置前，通过状态文件确定项目的状态。**

1.  **读取状态文件：**检查 `conductor/setup_state.json` 是否存在。
    - 若不存在，则为新项目设置。直接跳转至步骤 1.2。
    - 若存在，则读取其内容。

2.  **基于状态的恢复：**
    - 令JSON文件中`last_successful_step`的值为`step`。
    - 根据`step`的值跳转至**下一逻辑章节**：

    - 若`step`为“2.1_product_guide”，则提示“恢复设置：产品指南（`product.md`）已完成。接下来我们将创建产品规范。”并转至**第2.2节**。
    - 若`step`为“2.2_product_guidelines”，则提示“恢复设置：产品指南与规范已完成。接下来将定义技术栈”，并进入**第2.3节**。
    - 若`step` 为 “2.3_tech_stack”，则提示：“恢复设置：产品指南、规范及技术栈已定义。接下来将选择代码风格指南。” 并进入 **第 2.4 节**。
    - 若`step` 为 “2.4_code_styleguides”，则提示："恢复设置： 所有指南与技术栈已配置完毕。接下来将定义项目工作流。"并跳转至**第2.5节**。
    - 若`step`为“2.5_workflow”，则提示：“恢复设置：初始项目框架已完成。接下来将生成首个任务轨迹。”并跳转至**第二阶段(3.0)**。
    - 若 `step` 为 “3.3_initial_track_generated”：
        - 提示：“项目已初始化。可通过 `/conductor:newTrack` 创建新轨道，或使用 `/conductor:implement` 开始实现现有轨道。”
        - 终止 `setup` 流程。
    - 若 `step` 参数无效，则提示错误并终止。


---

## 1.2 初始化前概述
1.  **提供高层次概述：**
    -   向用户展示初始化流程概述：
        > "欢迎使用Conductor。我将引导您完成以下项目设置步骤：
        > 1. **项目识别:** 分析当前目录以判断是新建项目还是现有项目。
        > 2. **产品定义:** 协作定义产品愿景、设计规范及技术栈。
        > 3. **配置阶段:** 选择适用的代码规范指南并定制开发工作流。
        > 4. **任务生成:** 定义初始 **任务**（如功能开发或缺陷修复等高阶工作单元），并自动生成详细开发计划。
        >
        > 让我们开始吧！"

---

## 2.0 第一阶段：简化项目设置
**操作流程：遵循此步骤顺序，引导用户完成交互式设置。**

### 2.0 项目启动
1.  **检测项目成熟度:**
    -   **项目分类:** 根据以下指标判断项目属于“棕地项目”（现有项目）或“绿地项目”（新项目）:
    -   **现有项目指标:**
        -   检查是否存在版本控制目录：`.git`、`.svn` 或 `.hg`。
        -   若存在`.git`目录，执行`git status --porcelain`。若输出非空，则归类为“棕地”（现有项目）。
        -   检查依赖清单文件：`package.json`、`pom.xml`、`requirements.txt`、`go.mod`、`composer.json`。
        -   检查源代码目录：`src/`、`app/`、`lib/` 是否包含代码文件。
        -   若满足任意上述条件（版本控制目录、Git仓库、依赖清单或源代码目录），则归类为**现有项目**。
    -   **新项目:**
        -   仅当未发现任何“现有项目指标”且当前目录为空或仅包含通用文档（例如单个`README.md`文件），且不含功能性代码或依赖项时，方可归类为**新项目**


2.  **根据成熟度执行工作流:**
   -   **若为现有项目:**
       -   通知检测到现有项目.
       -   若`git status --porcelain`命令（作为存量项目指标执行）显示存在未提交的变更，则告知用户："警告： 您的Git仓库存在未提交修改。请在继续前提交或暂存修改，因Conductor将进行修改操作。"
       -    **启动存量项目初始化协议：**
    
       -   **1.0 预分析确认：**
           1. **请求授权：** 告知用户已检测到存量（现有）项目。
           2. **征求许可：** 请求进行只读扫描以分析项目，提供以下选项并采用如下结构：
                > A) Yes  
                  B) No
                > 
                > 请回复 A 或 B。 
           3. **处理拒绝：** 若许可被拒绝，终止流程并等待用户进一步指示。
           4. **确认操作：** 获得确认后进入下一步。

       -   **2.0 代码分析：**
           1.  **操作提示：** 告知用户即将执行代码分析。
           2.  **优先处理README：** 若存在`README.md`文件，优先进行分析。
           3.  **全面扫描：** 扩展分析范围至其他相关文件以全面理解
    
       -   **2.1 文件大小与相关性筛选：**
           1.  **尊重忽略文件：** 在扫描任何文件前，必须检查是否存在 `.geminiignore` 和 `.gitignore` 文件。若存在其中任一或两者，必须使用其组合模式将文件和目录排除在分析之外。若两者存在冲突，`.geminiignore`中的模式应优先于`.gitignore`。这是规避`node_modules`等冗余无关文件的核心机制。
           2.  **高效列举相关文件：** 列举待分析文件时，必须使用遵循忽略规则的命令。例如可使用 `git ls-files --exclude-standard -co | xargs -n 1 dirname | sort -u`，该命令能列出所有相关目录（Git 追踪目录及未忽略的其他文件），而无需逐个列出文件。若未使用 Git，则必须构建能读取忽略文件并剔除对应路径的 `find` 命令。
           3.  **手动忽略方案：** 仅当同时不存在 `.geminiignore` 和 `.gitignore` 时，才应手动忽略常见目录。示例命令：`ls -lR -I ‘node_modules’ -I ‘.m2’ -I ‘build’ -I ‘dist’ -I ‘bin’ -I ‘target’ -I ‘.git’ -I ‘.idea’ -I ‘.vscode’`。

       -   **2.2 提取并推断项目背景：**
           1.  **严格文件访问：** 切勿索取更多文件。分析仅基于提供的文件片段和目录结构。
           2.  **提取技术栈：** 分析清单文件内容以识别：
               -   编程语言
               -   框架（前端与后端）
               -   数据库驱动程序
           3.  **推断架构类型：** 根据文件树骨架（前两层）推断架构类型（如单仓库、微服务、MVC）。
           4.  **推断项目目标：** 严格依据提供的`README.md`标题或`package.json`描述，用一句话概括项目目标。
       -   **完成现有项目初始化后，请转至2.1节中的生成产品指南部分。**
   -   **若为新项目：**
       -   宣布将启动新项目。
       -   继续执行本文件中的下一步操作。

3.  **初始化 Git 仓库（适用于全新项目）:**
   -   若不存在 `.git` 目录，则执行 `git init` 并向用户报告已初始化新的 Git 仓库。

4.  **咨询项目目标（针对全新项目）:**
   -   **向用户提出以下问题，并在等待其回复后再进行下一步操作:** "您想构建什么？?"
   -   **重要提示：在用户作出响应之前，严禁执行任何工具调用.**
   -   **收到用户的回复后:**
       -   执行 `mkdir -p conductor`.
       -   **初始化状态文件:** 创建 `conductor` 目录后，必须立即创建 `conductor/setup_state.json` 文件，其内容必须完全如下所示：
            `{"last_successful_step": ""}`
       -   将用户的回复写入位于`conductor/product.md`文件中的标题为`# 初始概念`的段落下。

5.  **继续:** 立即进入下一部分.

### 2.1 生成产品指南（交互式）
1.  **本节介绍：** 告知用户，接下来将协助其创建 `product.md` 文件。
2.  **按顺序提问：** 每次只问一个问题。等待并处理用户的回答后，再提出下一个问题。持续进行这种交互过程，直至收集到足够的信息。
    -   **限制：** 请将您的咨询问题控制在最多5个。
    -   **建议：** 针对每个问题，根据已有的常见模式或上下文生成3个高质量的建议答案。
    -   **示例主题：** 目标用户、目标、功能等
    *   **一般准则：**
        *   **1. 分类问题类型:** 在设计任何问题之前，必须先将其目的归类为“补充型”或“排他型”。 
            *   使用**增量法**进行头脑风暴和范围定义（例如用户、目标、功能、项目指南）。这些问题允许多个答案。
            *   使用**独家选择**来处理基础性、单一性的承诺（例如选择主要技术、特定工作流规则）。此类问题需要唯一答案。
             
        *   **2. 问题设计：** 根据分类要求，必须遵循以下规范：
            *   **若为补充型：** 设计开放式问题以鼓励多点回答。必须在问题后直接添加选项列表及精确表述“（可多选）”。
            *   **若为排他性选择题：** 设计引导用户做出单一明确决策的直接问题。严禁添加“(选择所有适用选项)”字样。

        *   **3. 交互流程：**
            *   **关键要求：** 必须按顺序逐个提问。切勿在单轮中提出多个问题。每个问题后需等待用户回应。
            *   每个多选题的最后两个选项必须为“输入自定义答案”和“自动生成并审核`product.md`文件”。
            *   推进前通过总结确认理解无误。  
            - **格式：** 必须以垂直列表形式呈现，每个选项独占一行。
            - **结构：**
                > A) [选项A]  
                  B) [选项B]  
                  C) [选项C]  
                  D) [输入您自己的答案]  
                  E) [自动生成并审核 product.md]
                >
    -   **针对现有项目：** 根据代码分析提出项目现有内容相关问题。
    -   **自动生成逻辑：** 若用户选择选项E，立即停止本节提问。根据先前回答和项目背景，运用最佳判断推断剩余细节，生成完整的`product.md`内容，写入文件后转入下一节。

3.  **起草文档：** 对话结束后（或选择选项E时），生成`product.md`的内容。若选择选项E，请根据先前回答和项目背景，运用专业判断推断剩余细节。建议对收集到的细节进行扩展，以创建全面的文档。
    -   **关键提示：** 生成文档的唯一依据是**用户选择的答案**。必须完全忽略您提出的问题及任何未选中的`A/B/C`选项。
        -   **操作要求：** 将用户选定的答案整合为文档中结构规范的章节。建议对用户选择进行扩展，生成完整且精炼的输出内容。最终文件中严禁包含对话选项（A、B、C、D、E）。
4.  **用户确认循环：** 将起草内容提交给用户审核，并启动确认循环。      
    > “我已起草产品指南。请审阅以下内容：”
    >
    > ```markdown
    > [此处为起草版 product.md 内容]
    > ```
    >
    > "您希望下一步做什么？  
     A) **批准：** 文件内容正确，可继续推进。  
     B) **建议修改：** 请告知具体修改内容。
    >
    > 您可随时通过内置选项“使用外部编辑器修改”（若存在）编辑生成的文件，或在本步骤后使用您偏好的外部编辑器进行修改。   
     请回复A或B"。
    - **循环处理：** 根据用户回复，若需修改则应用变更并重新呈现文档，若获批准则终止循环。
5.  **写入文件：** 经批准后，将生成的内容追加到现有的 `conductor/product.md` 文件中，保留 `# 初始概念` 部分。
6.  **提交状态：** 文件创建成功后，必须立即向 `conductor/setup_state.json` 写入以下精确内容：
    `{"last_successful_step": "2.1_product_guide"}`

### 2.2 生成产品指南（交互式）
1.  **介绍本节：** 告知用户您将协助其创建 `product-guidelines.md` 文件。
2.  **依次提问：** 每次只提出一个问题。在询问下一个问题前，请等待并处理用户的回答。持续进行此交互流程，直至收集到足够信息。
    -   **限制:** 您的提问数量上限为5个。
    -   **建议:** 针对每个问题，基于现有常见模式或上下文生成3个高质量建议答案。为每个答案提供简要说明，并突出强调最推荐的选项。
    -   **示例主题:** 品牌信息传递、视觉识别系统等。
    *   **通用准则：**
        *   **1. 分类问题类型：** 在设计任何问题前，必须先将其目的归类为“增量型”或“排他型”。
            *   采用**增量型**问题用于头脑风暴和范围界定（如用户、目标、功能、项目准则）。此类问题允许多选答案。
            *   采用**排他性选择**用于基础性、单一承诺事项（例如选择核心技术、特定工作流规则）。此类问题要求唯一答案。
        *   **2. 问题设计：** 根据分类要求，必须遵循以下规范：
            *   **建议：** 提供选项时，应为每个选项附上简要说明，并突出强调最推荐的选项。
            *   **若为补充型：** 设计开放式问题以鼓励多点选择。必须在问题后直接添加选项列表及精确表述“(可多选)”。
            *   **若为排他型：** 设计引导用户作出单一明确决策的直接问题。严禁添加“(可多选)”字样。
        *   **3. 交互流程：**
            *   **关键要求：** 必须按顺序逐个提问。单轮内不得同时提出多个问题。每次提问后需等待用户回应。
            *   每个多选题的最后两个选项必须为“输入自定义答案”和`product-guidelines.md`。
            *   继续操作前请通过总结确认理解。
        - **格式：** 您必须以垂直列表形式呈现这些选项，每个选项独占一行。    
        - **结构：**
        > A) [选项A]  
          B) [选项B]  
          C) [选项C]  
          D) [输入您自己的答案]  
          E) [自动生成并审查 product-guidelines.md]
        >
    -   **自动生成逻辑：** 若用户选择选项E，则立即停止本部分提问，转入下一步进行文档起草。
3.  **起草文档：** 对话结束后（或选择选项E），生成`product-guidelines.md`的内容。若选择选项E，请根据先前回答和项目背景，运用专业判断推断剩余细节。建议对收集到的细节进行扩展，以创建全面的文档。
    **关键提示：** 生成文档的唯一依据是**用户选择的答案**。必须完全忽略您提出的问题及任何未选中的`A/B/C`选项。
    -   **操作要求：** 将用户选定的答案整合为文档中结构规范的章节。建议对用户选择进行扩展，生成完整且精炼的输出内容。最终文件中严禁包含对话选项（A、B、C、D、E）。
4.  **用户确认循环：** 将起草内容呈现给用户进行审核，并启动确认循环。
    > "我已起草技术栈文档，请审阅以下内容："
    >
    > ```markdown
    > [此处为tech-stack.md内容草稿]
    > ```
    >
    > "接下来您想做什么？
    > A) **批准：** 文件正确，我们可以继续。
    > B) **建议修改：** 请告知需要修改的内容。
    >
    > 您始终可以使用内置的“使用外部编辑器修改”选项（若存在）来编辑生成的文件，或在此步骤之后使用您喜欢的外部编辑器进行编辑。
    > 请用A或B作答。"
    - **循环：** 根据用户响应，要么应用更改并重新呈现文档，要么在批准时终止循环。
5.  **写入文件：** 经批准后，将生成的内容写入 `conductor/tech-stack.md` 文件。
6.  **提交状态：** 文件创建成功后，您必须立即向 `conductor/setup_state.json` 写入以下精确内容：
    `{"last_successful_step": "2.3_tech_stack"}`
7.  **继续：** 写入状态文件后，立即进入下一部分。

### 2.3 生成技术栈（交互式）
1.  **Introduce the Section:** Announce that you will now help define the technology stacks.
2.  **Ask Questions Sequentially:** Ask one question at a time. Wait for and process the user's response before asking the next question. Continue this interactive process until you have gathered enough information.
    -   **CONSTRAINT:** Limit your inquiry to a maximum of 5 questions.
    -   **SUGGESTIONS:** For each question, generate 3 high-quality suggested answers based on common patterns or context you already have.
    -   **Example Topics:** programming languages, frameworks, databases, etc
    *   **General Guidelines:**
        *   **1. Classify Question Type:** Before formulating any question, you MUST first classify its purpose as either "Additive" or "Exclusive Choice".
            *   Use **Additive** for brainstorming and defining scope (e.g., users, goals, features, project guidelines). These questions allow for multiple answers.
            *   Use **Exclusive Choice** for foundational, singular commitments (e.g., selecting a primary technology, a specific workflow rule). These questions require a single answer.

        *   **2. Formulate the Question:** Based on the classification, you MUST adhere to the following:
            *   **Suggestions:** When presenting options, you should provide a brief rationale for each and highlight the one you recommend most strongly.
            *   **If Additive:** Formulate an open-ended question that encourages multiple points. You MUST then present a list of options and add the exact phrase "(Select all that apply)" directly after the question.
            *   **If Exclusive Choice:** Formulate a direct question that guides the user to a single, clear decision. You MUST NOT add "(Select all that apply)".

        *   **3. Interaction Flow:**
            *   **CRITICAL:** You MUST ask questions sequentially (one by one). Do not ask multiple questions in a single turn. Wait for the user's response after each question.
            *   The last two options for every multiple-choice question MUST be "Type your own answer" and "Autogenerate and review tech-stack.md".
            *   Confirm your understanding by summarizing before moving on.
        - **Format:** You MUST present these as a vertical list, with each option on its own line.
        - **Structure:**
          A) [Option A]
          B) [Option B]
          C) [Option C]
          D) [Type your own answer]
          E) [Autogenerate and review tech-stack.md]
    -   **FOR EXISTING PROJECTS (BROWNFIELD):**
        -   **CRITICAL WARNING:** Your goal is to document the project's *existing* tech stack, not to propose changes.
        -   **State the Inferred Stack:** Based on the code analysis, you MUST state the technology stack that you have inferred. Do not present any other options.
        -   **Request Confirmation:** After stating the detected stack, you MUST ask the user for a simple confirmation to proceed with options like:
        A) Yes, this is correct.
        B) No, I need to provide the correct tech stack.
        -   **Handle Disagreement:** If the user disputes the suggestion, acknowledge their input and allow them to provide the correct technology stack manually as a last resort.
    -   **AUTO-GENERATE LOGIC:** If the user selects option E, immediately stop asking questions for this section. Use your best judgment to infer the remaining details based on previous answers and project context, generate the full `tech-stack.md` content, write it to the file, and proceed to the next section.
3.  **Draft the Document:** Once the dialogue is complete (or option E is selected), generate the content for `tech-stack.md`. If option E was chosen, use your best judgment to infer the remaining details based on previous answers and project context. You are encouraged to expand on the gathered details to create a comprehensive document.
    -   **CRITICAL:** The source of truth for generation is **only the user's selected answer(s)**. You MUST completely ignore the questions you asked and any of the unselected `A/B/C` options you presented.
    -   **Action:** Take the user's chosen answer and synthesize it into a well-formed section for the document. You are encouraged to expand on the user's choice to create a comprehensive and polished output. DO NOT include the conversational options (A, B, C, D, E) in the final file.
4.  **User Confirmation Loop:** Present the drafted content to the user for review and begin the confirmation loop.
    > "I've drafted the tech stack document. Please review the following:"
    >
    > ```markdown
    > [Drafted tech-stack.md content here]
    > ```
    >
    > "What would you like to do next?
    > A) **Approve:** The document is correct and we can proceed.
    > B) **Suggest Changes:** Tell me what to modify.
    >
    > You can always edit the generated file with the Gemini CLI built-in option "Modify with external editor" (if present), or with your favorite external editor after this step.
    > Please respond with A or B."
    - **Loop:** Based on user response, either apply changes and re-present the document, or break the loop on approval.
6.  **Write File:** Once approved, write the generated content to the `conductor/tech-stack.md` file.
7.  **Commit State:** Upon successful creation of the file, you MUST immediately write to `conductor/setup_state.json` with the exact content:
    `{"last_successful_step": "2.3_tech_stack"}`
8.  **Continue:** After writing the state file, immediately proceed to the next section.

### 2.4 Select Guides (Interactive)
1.  **Initiate Dialogue:** Announce that the initial scaffolding is complete and you now need the user's input to select the project's guides from the locally available templates.
2.  **Select Code Style Guides:**
    -   List the available style guides by running `ls ~/.gemini/extensions/conductor/templates/code_styleguides/`.
    -   For new projects (greenfield):
        -   **Recommendation:** Based on the Tech Stack defined in the previous step, recommend the most appropriate style guide(s) and explain why.
        -   Ask the user how they would like to proceed:
            A) Include the recommended style guides.
            B) Edit the selected set.
        -   If the user chooses to edit (Option B):
            -   Present the list of all available guides to the user as a **numbered list**.
            -   Ask the user which guide(s) they would like to copy.
    -   For existing projects (brownfield):
        -   **Announce Selection:** Inform the user: "Based on the inferred tech stack, I will copy the following code style guides: <list of inferred guides>."
        -   **Ask for Customization:** Ask the user: "Would you like to proceed using only the suggested code style guides?"
            - Ask the user for a simple confirmation to proceed with options like:
              A) Yes, I want to proceed with the suggested code style guides.
              B) No, I want to add more code style guides.
    -   **Action:** Construct and execute a command to create the directory and copy all selected files. For example: `mkdir -p conductor/code_styleguides && cp ~/.gemini/extensions/conductor/templates/code_styleguides/python.md ~/.gemini/extensions/conductor/templates/code_styleguides/javascript.md conductor/code_styleguides/`
    -   **Commit State:** Upon successful completion of the copy command, you MUST immediately write to `conductor/setup_state.json` with the exact content:
        `{"last_successful_step": "2.4_code_styleguides"}`

### 2.5 Select Workflow (Interactive)
1.  **Copy Initial Workflow:**
    -   Copy `~/.gemini/extensions/conductor/templates/workflow.md` to `conductor/workflow.md`.
2.  **Customize Workflow:**
    -   Ask the user: "Do you want to use the default workflow or customize it?"
        The default workflow includes:
        - 80% code test coverage
        - Commit changes after every task
        - Use Git Notes for task summaries
        -   A) Default
        -   B) Customize
    -   If the user chooses to **customize** (Option B):
        -   **Question 1:** "The default required test code coverage is >80% (Recommended). Do you want to change this percentage?"
            -   A) No (Keep 80% required coverage)
            -   B) Yes (Type the new percentage)
        -   **Question 2:** "Do you want to commit changes after each task or after each phase (group of tasks)?"
            -   A) After each task (Recommended)
            -   B) After each phase
        -   **Question 3:** "Do you want to use git notes or the commit message to record the task summary?"
            -   A) Git Notes (Recommended)
            -   B) Commit Message
        -   **Action:** Update `conductor/workflow.md` based on the user's responses.
        -   **Commit State:** After the `workflow.md` file is successfully written or updated, you MUST immediately write to `conductor/setup_state.json` with the exact content:
            `{"last_successful_step": "2.5_workflow"}`

### 2.6 Finalization
1.  **Summarize Actions:** Present a summary of all actions taken during Phase 1, including:
    -   The guide files that were copied.
    -   The workflow file that was copied.
2.  **Transition to initial plan and track generation:** Announce that the initial setup is complete and you will now proceed to define the first track for the project.

---

## 3.0 INITIAL PLAN AND TRACK GENERATION
**PROTOCOL: Interactively define project requirements, propose a single track, and then automatically create the corresponding track and its phased plan.**

### 3.1 Generate Product Requirements (Interactive)(For greenfield projects only)
1.  **Transition to Requirements:** Announce that the initial project setup is complete. State that you will now begin defining the high-level product requirements by asking about topics like user stories and functional/non-functional requirements.
2.  **Analyze Context:** Read and analyze the content of `conductor/product.md` to understand the project's core concept.
3.  **Ask Questions Sequentially:** Ask one question at a time. Wait for and process the user's response before asking the next question. Continue this interactive process until you have gathered enough information.
    -   **CONSTRAINT** Limit your inquiries to a maximum of 5 questions.
    -   **SUGGESTIONS:** For each question, generate 3 high-quality suggested answers based on common patterns or context you already have.
    *   **General Guidelines:**
        *   **1. Classify Question Type:** Before formulating any question, you MUST first classify its purpose as either "Additive" or "Exclusive Choice".
            *   Use **Additive** for brainstorming and defining scope (e.g., users, goals, features, project guidelines). These questions allow for multiple answers.
            *   Use **Exclusive Choice** for foundational, singular commitments (e.g., selecting a primary technology, a specific workflow rule). These questions require a single answer.

        *   **2. Formulate the Question:** Based on the classification, you MUST adhere to the following:
            *   **If Additive:** Formulate an open-ended question that encourages multiple points. You MUST then present a list of options and add the exact phrase "(Select all that apply)" directly after the question.
            *   **If Exclusive Choice:** Formulate a direct question that guides the user to a single, clear decision. You MUST NOT add "(Select all that apply)".

        *   **3. Interaction Flow:**
            *   **CRITICAL:** You MUST ask questions sequentially (one by one). Do not ask multiple questions in a single turn. Wait for the user's response after each question.
            *   The last two options for every multiple-choice question MUST be "Type your own answer" and "Auto-generate the rest of requirements and move to the next step".
            *   Confirm your understanding by summarizing before moving on.
        - **Format:** You MUST present these as a vertical list, with each option on its own line.
        - **Structure:**
          A) [Option A]
          B) [Option B]
          C) [Option C]
          D) [Type your own answer]
          E) [Auto-generate the rest of requirements and move to the next step]
    -   **AUTO-GENERATE LOGIC:** If the user selects option E, immediately stop asking questions for this section. Use your best judgment to infer the remaining details based on previous answers and project context.
-   **CRITICAL:** When processing user responses or auto-generating content, the source of truth for generation is **only the user's selected answer(s)**. You MUST completely ignore the questions you asked and any of the unselected `A/B/C` options you presented. This gathered information will be used in subsequent steps to generate relevant documents. DO NOT include the conversational options (A, B, C, D, E) in the gathered information.
4.  **Continue:** After gathering enough information, immediately proceed to the next section.

### 3.2 Propose a Single Initial Track (Automated + Approval)
1.  **State Your Goal:** Announce that you will now propose an initial track to get the project started. Briefly explain that a "track" is a high-level unit of work (like a feature or bug fix) used to organize the project.
2.  **Generate Track Title:** Analyze the project context (`product.md`, `tech-stack.md`) and (for greenfield projects) the requirements gathered in the previous step. Generate a single track title that summarizes the entire initial track. For existing projects (brownfield): Recommend a plan focused on maintenance and targeted enhancements that reflect the project's current state.
    - Greenfield project example (usually MVP):
        ```markdown
        To create the MVP of this project, I suggest the following track:
        - Build the core functionality for the tip calculator with a basic calculator and built-in tip percentages.
        ```
    - Brownfield project example:
        ```markdown
        To create the first track of this project, I suggest the following track:
        - Create user authentication flow for user sign in.
        ```
3.  **User Confirmation:** Present the generated track title to the user for review and approval. If the user declines, ask the user for clarification on what track to start with.

### 3.3 Convert the Initial Track into Artifacts (Automated)
1.  **State Your Goal:** Once the track is approved, announce that you will now create the artifacts for this initial track.
2.  **Initialize Tracks File:** Create the `conductor/tracks.md` file with the initial header and the first track:
    ```markdown
    # Project Tracks

    This file tracks all major tracks for the project. Each track has its own detailed plan in its respective folder.

    ---

    - [ ] **Track: <Track Description>**
      *Link: [./conductor/tracks/<track_id>/](./conductor/tracks/<track_id>/)*
    ```
3.  **Generate Track Artifacts:**
    a. **Define Track:** The approved title is the track description.
    b. **Generate Track-Specific Spec & Plan:**
    i. Automatically generate a detailed `spec.md` for this track.
    ii. Automatically generate a `plan.md` for this track.
    - **CRITICAL:** The structure of the tasks must adhere to the principles outlined in the workflow file at `conductor/workflow.md`. For example, if the workflow specificies Test-Driven Development, each feature task must be broken down into a "Write Tests" sub-task followed by an "Implement Feature" sub-task.
    - **CRITICAL:** Include status markers `[ ]` for **EVERY** task and sub-task. The format must be:
    - Parent Task: `- [ ] Task: ...`
    - Sub-task: `    - [ ] ...`
    - **CRITICAL: Inject Phase Completion Tasks.** You MUST read the `conductor/workflow.md` file to determine if a "Phase Completion Verification and Checkpointing Protocol" is defined. If this protocol exists, then for each **Phase** that you generate in `plan.md`, you MUST append a final meta-task to that phase. The format for this meta-task is: `- [ ] Task: Conductor - User Manual Verification '<Phase Name>' (Protocol in workflow.md)`. You MUST replace `<Phase Name>` with the actual name of the phase.
    c. **Create Track Artifacts:**
    i. **Generate and Store Track ID:** Create a unique Track ID from the track description using format `shortname_YYYYMMDD` and store it. You MUST use this exact same ID for all subsequent steps for this track.
    ii. **Create Single Directory:** Using the stored Track ID, create a single new directory: `conductor/tracks/<track_id>/`.
    iii. **Create `metadata.json`:** In the new directory, create a `metadata.json` file with the correct structure and content, using the stored Track ID. An example is:
    - ```json
            {
            "track_id": "<track_id>",
            "type": "feature", // or "bug"
            "status": "new", // or in_progress, completed, cancelled
            "created_at": "YYYY-MM-DDTHH:MM:SSZ",
            "updated_at": "YYYY-MM-DDTHH:MM:SSZ",
            "description": "<Initial user description>"
            }
            ```
    Populate fields with actual values. Use the current timestamp.
    iv. **Write Spec and Plan Files:** In the exact same directory, write the generated `spec.md` and `plan.md` files.

    d. **Commit State:** After all track artifacts have been successfully written, you MUST immediately write to `conductor/setup_state.json` with the exact content:
    `{"last_successful_step": "3.3_initial_track_generated"}`

    e. **Announce Progress:** Announce that the track for "<Track Description>" has been created.

### 3.4 Final Announcement
1.  **Announce Completion:** After the track has been created, announce that the project setup and initial track generation are complete.
2.  **Save Conductor Files:** Add and commit all files with the commit message `conductor(setup): Add conductor setup files`.
3.  **Next Steps:** Inform the user that they can now begin work by running `/conductor:implement`.
    """