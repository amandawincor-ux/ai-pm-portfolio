# ai-pm-portfolio
# AI PM Portfolio

个人从项目管理转型 AI 产品/技术方向的学习与实践记录。

## 关于我
- 6年 Java 开发经验 + 9年软件项目管理经验
- 目前专注 AI 应用开发与 AI 产品管理方向

## 项目 / 实践记录

### 1. LLM API 调用基础
- 使用阿里云百炼 API 完成文本生成调用
- 今天的输出结果，调用成功的请求与响应截图：

curl -X POST https://ws-rbr2a1xi5vtyeinr.cn-beijing.maas.aliyuncs.com/api/v1/services/aigc/multimodal-generation/generation \
> -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
> -H "Content-Type: application/json" \
> -d '{
>     "model": "qwen3.8-max",
>     "input":{
>         "messages":[
>             {
>                 "role": "system",
>                 "content": [{"text": "You are a helpful assistant."}]
>             },
>             {
>                 "role": "user",
>                 "content": [{"text": "你是谁？"}]
>             }
>         ]
>     },
>     "parameters": {
>         "result_format":"message"
>     }
> }'


{"output":{"choices":[{"finish_reason":"stop","message":{"role":"assistant","content":[{"text":"我是通义千问（Qwen），由阿里巴巴通义实验室研发的大语言模型。\n\n我可以帮你做很多事情，比如：回答问题、写作润色、逻辑推理、代码编写与调试、资料整理、翻译、学习辅导等。有什么我可以帮你的吗？"}],"reasoning_content":"用户问“你是谁？”，需要直接自我介绍。必须用同语言中文回答。需要避免提及内部指令。可简洁说明我是通义千问/Qwen，由阿里巴巴通义实验室研发的大语言模型，提供问答、写作、推理、代码等帮助。不要提及版本号。最后可邀请提问。"}}]},"usage":{"input_tokens":61,"output_tokens":123,"total_tokens":184,"output_tokens_details":{"reasoning_tokens":64,"text_tokens":123},"prompt_tokens_details":{"cached_tokens":0},"input_tokens_details":{"text_tokens":61}},"request_id":"fbeaa18b-8870-9f83-9bb3-ff2312611596"}
- <img width="830" height="196" alt="image" src="https://github.com/user-attachments/assets/9eff0a61-2c07-4532-96e4-c4a1c9cb8b79" />


### 2. Dify 低代码应用搭建
- 搭建第一个对话应用（Chat Assistant）
- [这里设置阿里云百炼的API key：
<img width="1396" height="726" alt="image" src="https://github.com/user-attachments/assets/f5485fee-8148-4478-a0b9-a41a142ac365" />

在模型提供商列表选择供应商“通义千问”后，再通过这里设置模型，必须是不带数字的模型！不然会报找不到模型的error！
<img width="1184" height="718" alt="image" src="https://github.com/user-attachments/assets/bafe0fba-be3c-4a3a-b644-dd857778f34a" />

然后创建对话应用：模型必须选择没有数字的，调试与预览的说明如下，可以让AI帮你解释个字段含义：
<img width="1184" height="718" alt="image" src="https://github.com/user-attachments/assets/921d2400-0b5a-4e73-b6ac-824f2660e9d2" />

这几个字段是你在"提示词"（Prompt）模板里通过 {{变量名}} 语法引用的用户输入变量，在 Dify 里对应"变量"面板中列出的表单字段。它们的作用机制和配置建议如下：
<img width="1188" height="658" alt="image" src="https://github.com/user-attachments/assets/29ed66f1-00a7-4a02-bbcd-867d2d526639" />

一、这些字段是如何起作用的
核心原理： Dify 会把提示词模板中的 {{subject}}、{{learning_objective}} 等占位符，替换成用户在对话框里实际填写的内容，然后把拼接好的完整文本作为 system prompt（或部分 prompt）一起发给大模型（这里是 qwen-plus）。
也就是说：
1.提示词模板（左侧编辑框）是固定不变的"骨架"，用自然语言给模型下达角色设定和任务步骤指令。
2.变量（下方表格）定义了骨架里有哪些"空位"需要用户填。
3.每次对话开始时，如果这些变量被设置为"必填"（对应右侧的开关"可选"），用户界面上就会弹出对应的输入框，让用户填写。
4.用户填写后，Dify 后台自动把 {{subject}} 替换成用户填的实际文字，生成最终prompt，再发给qwen-plus。
{{subject}} → 让模型判断学科领域（数学/生物/历史等），用来校准术语和讲解深度
{{learning_objective}} → 让模型理解用户想达成的学习目标（比如"用因式分解解一元二次方程"）
（被截断未显示，但可推测）current_level、specific_question_or_task、contextual_background 应该分别对应用户当前水平、具体问题/任务、背景信息，同样会在后续步骤指令中被引用来指导模型如何调整回答方式。
补充几点实操建议
1.必填 vs 选填的取舍：为了降低用户使用门槛（打字太多容易劝退），建议只把 specific_question_or_task 设为必填，其余都设为可选——因为很多用户其实说清楚问题就够了，模型可以根据对话追问补充学科、水平等信息，这样体验更自然。
2.在提示词里加"缺省处理逻辑"：比如可以在 instruction 里补一句，"如果 {{current_level}} 或 {{subject}} 未提供，先根据 {{specific_question_or_task}} 自行判断，或礼貌地反问用户以确认。"这样即使用户没填某些选填字段，模型也不会卡壳。
3."字段名称"其实就是表单标签：注意左表格里的"字段名称"列，这才是用户在聊天界面实际看到的输入框标签文字（比如你可以把 subject 显示为"你想学的学科"），跟"变量 KEY"（代码里引用的 {{subject}}）是两回事，可以分开设置更友好的展示名。
4.如果这个 Agent 未来要接知识库（截图里下方的"上下文"区域），可以考虑导入教材/题库作为检索上下文，让模型回答更贴近具体课程内容，而不仅依赖这几个变量提供的信息。
发布运行后
<img width="1088" height="542" alt="image" src="https://github.com/user-attachments/assets/89c30f68-03dc-4964-9366-4c0e0a1a2d97" />

就拥有了一个可以对话的AI agent！
<img width="1214" height="668" alt="image" src="https://github.com/user-attachments/assets/c8ce8e66-9918-49bf-aaa9-89835c4b665a" />


Prompt 日志如下：
<img width="1396" height="720" alt="image" src="https://github.com/user-attachments/assets/e3757709-1b10-4167-866d-7d0e84aefcf2" />


### 3. RAG 知识库与向量检索
- 使用 Dify 知识库完成语义检索测试
- <img width="1396" height="786" alt="image" src="https://github.com/user-attachments/assets/709f9876-2226-4019-b27b-ecff4c2ca69c" />
<img width="1394" height="684" alt="image" src="https://github.com/user-attachments/assets/fdb1940e-8978-4e1d-b933-fc2fe8d19d8a" />

<img width="1396" height="672" alt="image" src="https://github.com/user-attachments/assets/99bf0a79-fdd6-48f7-8468-63b9adce453f" />
点击“发布”后，点击execute workflow，然后在terminal运行
curl -X POST "http://localhost:5678/webhook/text-summary"   -H "Content-Type: application/json"   -d '{"text":"今天学习了 n8n 的基本概念：workflow、节点、触发器；还学会了用 LLM 对一段中文新闻做自动摘要。"}'
<img width="1396" height="198" alt="image" src="https://github.com/user-attachments/assets/c13ae8a0-1478-4e46-a7c7-f5077fa37d1f" />
![Uploading image.png…]()


## 技术栈
- LLM: 通义千问 (Qwen)
- 低代码平台: Dify
- 向量检索: Dify 内置向量库 / ChromaDB

## 架构图
（待补充）

## 录屏演示
（待补充）
