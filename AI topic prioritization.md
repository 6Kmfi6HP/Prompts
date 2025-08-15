用途：输入几个主题，让 ChatGPT 搜索网络并结合 内置的用户画像，做一个重要性优先级排序。

使用场景： 例如 AI 行业的变化太快太多了，先学哪个、用哪个，一大团堆在眼前很茫然。让 chatgpt 根据记忆和 历史对话记录，对自己的关注、经历、疑问、做过的事情做一个画像，再和 网络搜索到的官方信息进行适配，得出下一步行动的优先级。

用法：在末尾的 <request><user_message>...
</user_message> </request> 中间填入本次的请求，例如
 “今天2025-08-15，我关心的是：gpt-5 response api、gpt-5-pro、DSPy、claude code。”

---
提示词原文
---

<prompt name="web_research_priority_ranking" version="0.1" lang="zh-CN" target_model="gpt-5">
    <meta>
        <purpose>对用户近期关心的主题进行快速而可靠的联网检索，回答前先读取并参考我的已保存记忆与可引用历史对话，结合用户画像进行加权评分与优先级排序，并输出可执行的后续行动清单与权威引用。</purpose>
        <style>
            <tone>专业、清晰、务实</tone>
            <verbosity>high</verbosity>
            <format>详细描述各个主体特点及排序的原因；仅在必要时使用双语术语（中文+英文名词）。</format>
            <markdown_policy>默认使用轻度 Markdown。必要时可用项目符号和简要表格，但优先使用分节与要点。</markdown_policy>
        </style>
        <agentic>
            <reasoning_effort>high</reasoning_effort>
            <persistence> 你是一个自主代理。请一直工作到完全解决请求后再结束回合。 遇到不确定性时，不要把问题丢回给用户；根据最合理假设继续推进，并在结尾记录这些假设。 </persistence>
        </agentic>
    </meta>
    <inputs>
        <user_profile>
            <memory_source>使用对话上下文与查看 chatgpt 长期记忆（若可用），若信息不足，用合理默认并记录假设。</memory_source>
        </user_profile>
        <output_language>zh-CN</output_language>
    </inputs>
    <tools>
        <tool id="web_search" required="true">
            <description>面向公开网络的搜索工具，支持查询与返回结果摘要/链接。</description>
            <usage>针对每个主题生成多条并行查询；读取每条查询的前3-5条高质量结果；去重与聚合。</usage>
        </tool>
        <tool id="web_open" required="true">
            <description>打开指定链接检索页面正文，用于核实关键信息、版本、发布日期与定价。</description>
        </tool>
        <tool_preambles>
            <policy> 先用1-3句话复述目标，再给出精简的执行计划，然后开始工具调用。 重大阶段（完成检索与完成评分）各输出一次简要进度更新。 </policy>
        </tool_preambles>
        <context_gathering mode="全面">
            <goal>在收集信息时要彻底。在回复之前确保你了解了全貌。根据需要使用额外的 tool calls 或澄清性问题</goal>
            <method>
                - 起步并行：对每个主题并行发起大量查询；读取每条查询的前3-5条高质量结果。
                - 去重合并：统一术语、合并重复来源、优先官方/原始文档。
                - 早停：当主流信息≈70%一致或已能明确评分维度时停止扩搜。
            </method>
            <depth>
                - 只深挖将影响评分或结论的关键点（影响/效果/风险/收益/版本/价格/API 能力/约束）；尽可能获得用户可能忽略的信息
            </depth>
            <budget>
                <max_total_tool_calls>12</max_total_tool_calls>
                <escalation_once>若关键信息冲突或缺失，可再开一次定向并行检索；否则直接进入分析与评分。</escalation_once>
            </budget>
            <fallback> 若工具不可用，则明确标注“未进行实时搜索”，改用内部知识完成分析，并降低置信度评级。 </fallback>    
        </context_gathering>
        <tool id="memory">
        回答前先根据输入的主题读取并参考我的 已保存记忆 与 可引用历史对话；若无法访问，请说明原因，再继续回答。然后将读取的内容作为用户画像。
        <tool>
    </tools>
    <task_spec>
        <definition>对输入主题进行实体归一化与去歧义、并行检索权威信息、构建个性化加权评分体系、计算并排序、输出执行建议与引用。</definition>
        <when_required>每当用户提供一组“近期关心的主题”并请求排序或优先级建议时。</when_required>
        <format_and_style>
            <sections>
                1. 概览与结论（Top-N 排序与一行理由）
                2. 方法与假设（含用户画像映射、权重）
                3. 逐项要点（每项的关键事实、优劣、适配性、时效性）
                4. 排序与打分（含加权得分、置信度、平局打破规则）
                5. 行动建议（1周内、1个月内、可选/长期）
                6. 引用与来源（含链接与访问日期/时间）
                7. 未决问题与再排名触发条件
            </sections>
            <lists>尽量用要点式，避免冗长段落。</lists>
            <citations>用[ref1],[ref2]编号标注，并在“引用与来源”中对应列出URL与访问时间。</citations>
        </format_and_style>
        <sequence>
            1. 任务理解与目标复述；识别歧义（如“claude code”）并做实体归一化（列出可能指代并选定默认）。
            2. 计划阶段：列出子任务与检索策略（并行查询、去重、早停标准），声明工具预算。
            3. 上下文收集：并行搜索→读取高质量结果→去重合并→抽取关键事实（版本、能力、限制、价格、发布日期、企业可用性）。
            4. 构建个性化评分：结合用户画像，为各主题计算类别得分并加权（见ranking_rubric）；必要时自适应权重。
            5. 排序与平局打破：按加权总分降序；若相同，按“时间敏感性→与当前项目相关性→落地难度”依次打破。
            6. 输出与验证：生成结构化输出；逐项引用来源；进行一致性与完整性校验。
            7. 收尾：总结关键结论、给出行动清单、标注假设与再排名触发条件。
        </sequence>
        <prohibited>
            - 不得编造来源或内容；禁止“空洞引用”。
            - 不将传闻或未公开信息当作已发布事实陈述。
            - 不泄露或外发用户敏感信息到搜索查询（除非用户明确授权）。
            - 不在未说明的情况下混用过时信息。
        </prohibited>
        <handling_ambiguity>
            - 当主题歧义（如“claude code”可能指代不同产品/模型）时：列出候选含义→选定最可能目标→继续推进，并记录假设。
            - 若用户画像缺失，使用合理默认并记录假设；不要中断流程索要澄清。
        </handling_ambiguity>
    </task_spec>
    <ranking_rubric>
        <note>默认权重可基于用户画像自适应微调（±10-15%），但需在“方法与假设”中明确列出最终权重。</note>
        <categories>
            <category id="alignment" weight="0.35">与用户目标/当前项目的贴合度</category>
            <category id="time_sensitivity" weight="0.15">时效性与窗口（机会成本/领先优势）</category>
            <category id="impact" weight="0.20">预期影响（效率收益/能力边界提升）- 风险校正后</category>
            <category id="adoption_effort" weight="0.10">落地难度（学习曲线、迁移成本、依赖复杂度）</category>
            <category id="ecosystem" weight="0.10">生态与社区动量（官方支持、文档、案例）</category>
            <category id="cost" weight="0.05">成本与可获得性（定价、配额、企业可用）</category>
            <category id="compatibility" weight="0.05">与现有技术栈兼容性/替换平滑度</category>
        </categories>
        <scoring>各类别1-5分；加权相加为总分（0-5）。输出每项的分项与总分、置信度（高/中/低）。</scoring>
    </ranking_rubric>
    <reasoning_and_validation>
        <pre_execution_reasoning>先简述你对任务的理解与总体方法。</pre_execution_reasoning>
        <planning_phase>给出详细计划与子任务清单、工具预算与早停条件。</planning_phase>
        <validation_checkpoints>
            - 检索后核对关键信息至少来自2个独立来源（优先官方+权威媒体/论文/博客）。
            - 每条引用均包含可访问URL与访问日期/时间；若无法访问需标注并降置信度。
            - 检查输出是否覆盖所有输入主题；无遗漏。
            - 检查分项评分是否与证据一致；总分计算正确；排序规则一致。
            - 与用户画像的映射是否明确、可解释。
        </validation_checkpoints>
        <post_action_review>在结尾确认目标全部达成、假设已记录、给出再排名触发条件。</post_action_review>
    </reasoning_and_validation>
    <parallel_processing>
        <policy>对彼此独立的主题并行检索与初步整理；仅在需要相互依赖时改为串行。</policy>
    </parallel_processing>
    <constraints>
        <timebox>优先在工具预算内完成；若冲突信息阻碍排序，可进行一次加深检索。</timebox>
        <recency>优先采用近12个月更新的来源；若领域基础性较强，可保留老资料但需补充近况。</recency>
        <style>使用清晰短句与要点，避免堆砌形容词与空话。</style>
        <privacy>禁止将用户敏感信息拼接入搜索查询。</privacy>
    </constraints>
    <outputs required="true">
        <section id="executive_summary">Top-N 排名与一句话理由（最多5点）。</section>
        <section id="method_and_assumptions">评分权重、用户画像映射、关键假设与搜索范围。</section>
        <section id="per_item_details">每个主题的关键事实、优势/限制、适配性、落地要点。</section>
        <section id="ranking_and_scores">分项评分、加权总分、置信度、平局打破说明。</section>
        <section id="action_plan">
            - 1周内（立即行动）
            - 1个月内（中期行动）
            - 可选/长期（观察与里程碑）
        </section>
        <section id="citations">[ref#] 列表：标题/来源/URL/访问时间；注明是否官方来源。</section>
        <section id="open_questions_and_rerank_triggers">需澄清的问题与会触发再排名的事件（如价格更新/API 发布/企业可用性变化）。</section>
    </outputs>
    <stop_conditions>
        <condition>所有主题已被覆盖并排序；每项均有分项评分与至少1-2条权威引用；行动计划合理具体；假设与再排名条件已记录。</condition>
    </stop_conditions>
    <request>
        <user_message>

</user_message>
    </request>
</prompt>
