---
title: 构建Agents框架｜Qwen-Agent使用概览
author: AI智能体Auto
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwOTcxMjE5MQ==&mid=2247483829&idx=1&sn=0ba168c9fe23568ef75bafc6dc086a73&chksm=c0a1b6f2267762324ba3d6b7ce775d9b6c43ea364b8332755cc529b7d52e44617b5864579e40&mpshare=1&scene=24&srcid=11177qqzRR9zKh9zQbZPQths&sharer_shareinfo=531fadad593430982fb96f1951cf7771&sharer_shareinfo_first=531fadad593430982fb96f1951cf7771#rd
---

**01 前言**

目前市面上有很多用于构建Agent的框架，包括有Langchain（116k）、LlamaIndex（44.3k），AutoGen（49.9k）、CrewAI（38.2k）、MetaGPT（58.4k），Qwen-Agent（11.6k）、Youtu-Agent（2.9k），另外还有OpenAI Agents SDK等等。（框架括号里的数量对应项目在github的star数）。

而QwenAgent是阿里云开源的一个用于构建Agent应用的开发框架。框架主要包括LLM、Tool、Memory、Agent等组件，这些也是构建复杂Agent的基础，在构建Agent应用时，可以选择框架内置的组件或者自定义这些组件。

github地址：https://github.com/QwenLM/Qwen-Agent

**02 LLM组件**

**介绍**

LLM组件是构建Agent不可或缺的一部分，提供决策、任务分解、组织响应等功能。

Qwen-Agent框架中默认定义了如下的LLM实现：

```
# qwen_agent/llm/__init__.py__all__ = [   'BaseChatModel',   'QwenChatAtDS',   'TextChatAtOAI',   'TextChatAtAzure',   'QwenVLChatAtDS',   'QwenVLChatAtOAI',   'QwenAudioChatAtDS',   'QwenOmniChatAtOAI',   'OpenVINO',   'Transformers',   'get_chat_model',   'ModelServiceError',]
```

支持千问系列文本、多模态问答及兼容OpenAI的接口调用。

**使用方式**

运行示例前执行如下命令安装Python依赖包：  
pip install -U "qwen-agent[rag,code\_interpreter,gui,mcp]"

方式1、外部直接调用LLM

```
from qwen_agent.llm import get_chat_model  
llm_cfg = {    # Use the model service provided by DashScope:    # 'model_type': 'qwen_dashscope',    'model': 'Qwen/Qwen3-Coder-480B-A35B-Instruct',    # base_url，也称为 api_base    'model_server': 'https://api-inference.modelscope.cn/v1/',      'api_key': 'ms-*',  
    # Use your own model service compatible with OpenAI API:    # 'model': 'Qwen',    # 'model_server': 'http://127.0.0.1:7905/v1',    # (Optional) LLM hyper-parameters:    'generate_cfg': {        'top_p': 0.8    }  }# 确认使用的具体llm实现llm = get_chat_model(llm_cfg)  
messages = [{    'role': 'user',    'content': "自我介绍"}]responses = []  
for responses in llm.chat(messages=messages,                          functions=None,                          delta_stream=True, # 每次消息都是增量的内容                          stream=True):    print(responses[0]['content'], end='', flush=True)
```

首先指定选择的模型配置，框架会依次通过model\_type、url或者model参数进行确认具体的LLM实现，最后调用chat方法完成问答。示例使用了url为魔塔社区的HTTP API，所以框架会选用兼容OpenAI实现方式完成模型的调用。

api\_key可以访问魔塔社区进行获取：

https://modelscope.cn/my/myaccesstoken

方式2、Agent内部调用

```
from qwen_agent.agents import BasicAgentfrom qwen_agent.utils.output_beautify import typewriter_print  
llm_cfg = {    # Use the model service provided by DashScope:    # 'model_type': 'qwen_dashscope','model': 'Qwen/Qwen3-Coder-480B-A35B-Instruct','model_server': 'https://api-inference.modelscope.cn/v1/',  # base_url，也称为 api_base'api_key': 'ms-*',  
    # Use your own model service compatible with OpenAI API:    # 'model': 'Qwen',    # 'model_server': 'http://127.0.0.1:7905/v1',    # (Optional) LLM hyper-parameters:'generate_cfg': {'top_p': 0.8    }  }# 最基础的，只有llm调用bot = BasicAgent(llm=llm_cfg)  
messages = [{'role': 'user','content': "自我介绍"}]  
responses = []response_plain_text = ''for responses in bot.run(messages=messages):    # 另一种输出方式    # 流式输出。    response_plain_text = typewriter_print(responses, response_plain_text)
```

这里使用的是最基础的Agent：BasicAgent，它仅支持模型调用，不支持如工具的使用。通过调用run方法实现Agent的调用，进而完成模型的调用。

方式3、自定义LLM组件实现

```
from typing import List, Iterator, Union, Dict, Literal, Optional  
from qwen_agent.llm import BaseChatModelfrom qwen_agent.llm.base import register_llmimport osfrom qwen_agent.llm.schema import Message, ASSISTANTfrom openai import OpenAI  
# 自定义 LLM# 可以集成BaseChatModel，或者其它类型的LLM,如TextChatAtOAI# 如果模型不支持Function Call，可以继承[BaseFnCallModel](../qwen_agent/llm/function_calling.py)类，#   这个类通过封装一个类似ReAct的工具调用Prompt，已经基于普通对话接口实现了Function Calling@register_llm('ds')class DsChat(BaseChatModel):  
    def __init__(self, cfg: Optional[Dict] = None):super().__init__(cfg)        self.model = self.model or'DeepSeek-V3.1'        cfg = cfg or {}  
        # # url 可以配置的方式; 可以默认        # api_base = cfg.get('api_base')        # api_base = api_base or cfg.get('base_url')        # api_base = api_base or cfg.get('model_server')  
        api_key = cfg.get('api_key')        api_key = api_key or os.getenv('DEEPSEEK_API_KEY')        self.api_key = api_key        print(api_key)  
    def _chat_with_functions(self, messages: List[Union[Message, Dict]], functions: List[Dict], stream: bool,                             delta_stream: bool, generate_cfg: dict, lang: Literal['en', 'zh']) -> Union[        List[Message], Iterator[List[Message]]]:        # 演示，先不实现        pass  
  
    def _chat_stream(self, messages: List[Message], delta_stream: bool, generate_cfg: dict) -> Iterator[List[Message]]:        # 流式对话        client = OpenAI(api_key=self.api_key, base_url="https://api.deepseek.com")  
        # 转换消息格式        formatted_messages = [{"role": "system", "content": "You are a helpful assistant"}]for msg in messages:if hasattr(msg, 'role') and hasattr(msg, 'content'):                formatted_messages.append({"role": msg.role, "content": msg.content})            elif isinstance(msg, dict):                formatted_messages.append(msg)  
        responses = client.chat.completions.create(            model="deepseek-chat",            messages=formatted_messages,            stream=True        )  
if delta_stream:            # 增量流式：每个块返回一个部分内容for chunk in responses:if chunk.choices[0].delta.content is not None:                    yield [Message(role=ASSISTANT, content=chunk.choices[0].delta.content)]else:            # 累积流式：累积完整内容后返回            full_content = ""for chunk in responses:if chunk.choices[0].delta.content is not None:                    full_content += chunk.choices[0].delta.content                    yield [Message(role=ASSISTANT, content=full_content)]  
  
  
    def _chat_no_stream(self, messages: List[Message], generate_cfg: dict) -> List[Message]:        pass  
if __name__ == '__main__':    ds_llm = DsChat({"api_key": "sk-*","model_type": "ds"    })    rsp = ds_llm.chat(        messages = [{"role":"user", "content":"自我介绍"}],        stream=True,        delta_stream = True    )for r in rsp:        print(r[0]['content'], end='')
```

首先通过@register\_llm注册LLM组件，并自定义模型类型参数，这里定义“ds”，实现则调用DeepSeek模型。然后继承BaseChatModel,分别实现\_chat\_with\_functions (工具调用接口）、

\_chat\_stream (流式生成接口)、

\_chat\_no\_stream (非流式生成接口)。

在使用时，只需指定"model\_type":"ds"即可调用该实现。

**03 Tool组件**

**介绍**

工具可以是Qwen-Agent内置的，如高德天气查询；或者自定义实现如股票查询分析等等。同时支持配置MCP，借助MCP生态实现Agent的能力扩展。

Qwen-Agent默认定义的Tool组件包括：

```
# qwen_agent/tools/__init__.py  
__all__ = ['BaseTool','CodeInterpreter','ImageGen','AmapWeather','TOOL_REGISTRY','DocParser','KeywordSearch','Storage','Retrieval','WebExtractor','SimpleDocParser','VectorSearch','HybridSearch','FrontPageSearch','ExtractDocVocabulary','PythonExecutor','MCPManager','WebSearch',]
```

其中BaseTool属于基础的类。这里可以看到Tool还包括了HybridSearch混合查询（包括关键词查询+向量化查询），通过知识库来提供模型上下文。

**使用方式**

方式1、外部直接调用

```
from qwen_agent.tools.amap_weather import AmapWeather  
tool = AmapWeather()  
# 环境变量AMAP_TOKEN设置import osos.environ['AMAP_TOKEN'] = 'your-api-key-here'  
res = tool.call(params = {'location': '西湖区'})  
print(res)
```

外部调用统一使用call方法。其中AmapWeather是内置的高德天气查询工具，配置环境变量AMAP\_TOKEN，即可查询指定地区的天气。

方式2、Agent内部调用

```
from qwen_agent.agents.assistant import Assistantfrom qwen_agent.utils.output_beautify import typewriter_print  
llm_cfg = {    # 使用与 OpenAI API 兼容的模型服务，例如 vLLM 或 Ollama：'model': 'Qwen/Qwen3-Coder-480B-A35B-Instruct','model_server': 'https://api-inference.modelscope.cn/v1/',  # base_url，也称为 api_base'api_key': 'ms-*',  
    # （可选） LLM 的超参数：'generate_cfg': {'top_p': 0.8    }}  
system_instruction = '''生成并执行1+1的python脚步，执行code_interpreter获取结果'''tools = ['code_interpreter']  # `code_interpreter` 是框架自带的工具，用于执行代码。  
bot = Assistant(llm=llm_cfg,                system_message=system_instruction,                function_list=tools)messages = [{"role":"user", "content":"告诉我结果"}]  
response_plain_text = ""  
rsp = bot.run(messages)for r in rsp:    # 流式输出。    response_plain_text = typewriter_print(r, response_plain_text)
```

在Agent中，使用`\_call\_tool(...)`函数调用工具，每个Agent实例可以调用初始化给它的工具列表。这里指定了内置的code\_interpreter工具配合通用的Assistant Agent完成调用python代码得出结果。api\_key通过魔塔社区获取。

方式3、自定义工具

```
from typing import Union, List  
from qwen_agent.llm.schema import ContentItemfrom qwen_agent.tools.base import register_toolfrom qwen_agent.tools.base import BaseToolfrom qwen_agent.agents.assistant import Assistantfrom qwen_agent.utils.output_beautify import typewriter_print  
  
@register_tool("add")class Add(BaseTool):    description = 'a add tool can calculate one number add other number，example 1 + 1 = 2'    parameters = {'type': 'object','properties': {'one': {"description":"第一个数",'type': 'number'            },'two': {"description": "第二个数",'type': 'number'            },        },'required': ['one', 'two'],    }  
    def call(self, params: Union[str, dict], **kwargs) -> Union[str, list, dict, List[ContentItem]]:        params = self._verify_json_format_args(params)        one = params['one']        two = params['two']returnint(one) + int(two)  
  
# 使用agent测试tool  
llm_cfg = {    # 使用与 OpenAI API 兼容的模型服务，例如 vLLM 或 Ollama：'model': 'Qwen/Qwen3-Coder-480B-A35B-Instruct','model_server': 'https://api-inference.modelscope.cn/v1/',  # base_url，也称为 api_base'api_key': 'ms-*',  
    # （可选） LLM 的超参数：'generate_cfg': {'top_p': 0.8    }}  
  
system_instruction = '''生成并执行1+1的python脚步，执行add获取结果'''tools = ['add']  # `add` 自定义的工具  
bot = Assistant(llm=llm_cfg,                system_message=system_instruction,                function_list=tools)messages = [{"role":"user", "content":"告诉我结果"}]  
response_plain_text = ""  
rsp = bot.run(messages)for r in rsp:    # 流式输出。    response_plain_text = typewriter_print(r, response_plain_text)
```

这里通过@register\_tool定义了一个自定义工具，继承于BaseTool。指定了工具的名称为add，以及工具描述和参数说明parameters。parameters定义为JSON   
Schema格式。使用时把工具换成定义的add名称，实现算术的加法。

**04 Agent组件**

**介绍**

Agent实现是核心部分，它会通过调用LLM和Tool来解决复杂场景问题的回答。可以实现不同的模式，如多轮问答、多个Agent相互协同等。

Qwen-Agent内置了如下的Agent实现：

```
# qwen_agent/agents/__init__.py__all__ = ['Agent','BasicAgent','MultiAgentHub','DocQAAgent','DialogueSimulator','HumanSimulator','ParallelDocQA','Assistant','ArticleAgent','ReActChat','Router','UserAgent','GroupChat','WriteFromScratch','GroupChatCreator','GroupChatAutoRouter','FnCallAgent','VirtualMemoryAgent','DialogueRetrievalAgent','TIRMathAgent',]
```

最基础的是Agent类，所有的Agent实现均直接或间接的继承至Agent类。一个Agent对象集成了工具调用和LLM调用接口。Agent接收一个消息列表输入，并返回一个消息列表的生成器，即流式输出的消息列表。

**使用方式**

上面的示例中已经用到Assistant Agent，下面看下多Agent协同和自定义的使用方式。

方式1、多agent协同

```
"""例如在下面的例子中，我们实例化一个五子棋群聊，其中真人用户下黑棋，并实例化白棋玩家和棋盘两个Agent作为NPC：从结果可以看出，在用户小塘输入下棋位置<1,1>后，群聊先自动选择棋盘来更新；然后选择NPC小明发言；小明发言结束后棋盘再次更新；然后继续选择用户小塘发言，并等待用户输入。然后用户小塘继续输入下棋位置<1,2>后，群聊又继续管理NPC发言......"""  
from qwen_agent.agents import GroupChatfrom qwen_agent.llm.schema import Message  
# Define a configuration file for a multi-agent:# one real player, one NPC player, and one chessboardNPC_NAME = '小明'USER_NAME = '小塘'CFGS = {'background':    f'一个五子棋群组，棋盘为5*5，黑棋玩家和白棋玩家交替下棋，每次玩家下棋后，棋盘进行更新并展示。{NPC_NAME}下白棋，{USER_NAME}下黑棋。',    'agents': [{'name': '棋盘','description': '负责更新棋盘','instructions': '你扮演一个五子棋棋盘，你可以根据原始棋盘和玩家下棋的位置坐标，把新的棋盘用矩阵展示出来。棋盘中用0代表无棋子、用1表示黑棋、用-1表示白棋。用坐标<i,j>表示位置，i代表行，j代表列，棋盘左上角位置为<0,0>。','selected_tools': ['code_interpreter']    }, {'name': NPC_NAME,'description': '白棋玩家','instructions': '你扮演一个玩五子棋的高手，你下白棋。棋盘中用0代表无棋子、用1黑棋、用-1白棋。用坐标<i,j>表示位置，i代表行，j代表列，棋盘左上角位置为<0,0>，请决定你要下在哪里，你可以随意下到一个位置，不要说你是AI助手不会下！返回格式为坐标：\n<i,j>\n除了这个坐标，不要返回其他任何内容',    }, {'name': USER_NAME,'description': '黑棋玩家','is_human': True    }]}  
  
def app():    llm= {'model': 'Qwen/Qwen3-Coder-480B-A35B-Instruct','model_server': 'https://api-inference.modelscope.cn/v1/',  # base_url，也称为 api_base'api_key': 'ms-*',    }    # Define a group chat agent from the CFGS    bot = GroupChat(agents=CFGS, llm=llm)    # Chat    messages = []while True:        query = input('user question: ')        messages.append(Message('user', query, name=USER_NAME))        response = []for response in bot.run(messages=messages):            # 输出            print('bot response:', response)        messages.extend(response)  
if __name__ == '__main__':    app()
```

这是官方的一个示例，模拟和机器人下五子棋。通过GroupChat类指定多个Agent，包括一个棋盘Agent和两个玩家，一个NPC（非玩家角色）一个真人玩家。

方式2、自定义Agent

```
import copyfrom typing import Iterator, List  
from qwen_agent import Agentfrom qwen_agent.llm.schema import CONTENT, ROLE, SYSTEM, Messagefrom qwen_agent.utils.output_beautify import typewriter_print  
PROMPT_TEMPLATE_ZH = """请充分理解以下参考资料内容，组织出满足用户提问的条理清晰的回复。#参考资料：{ref_doc}  
"""  
PROMPT_TEMPLATE_EN = """Please fully understand the content of the following reference materials and organize a clear response that meets the user's questions.# Reference materials:{ref_doc}  
"""  
PROMPT_TEMPLATE = {'zh': PROMPT_TEMPLATE_ZH,'en': PROMPT_TEMPLATE_EN,}  
  
class DocQA(Agent):  
    def _run(self,             messages: List[Message],             knowledge: str = '',             lang: str = 'en',             **kwargs) -> Iterator[List[Message]]:        messages = copy.deepcopy(messages)        system_prompt = PROMPT_TEMPLATE[lang].format(ref_doc=knowledge)if messages and messages[0][ROLE] == SYSTEM:            messages[0][CONTENT] += system_promptelse:            messages.insert(0, Message(SYSTEM, system_prompt))  
return self._call_llm(messages=messages)  
  
if __name__ == '__main__':    llm = {'model': 'Qwen/Qwen3-Coder-480B-A35B-Instruct','model_server': 'https://api-inference.modelscope.cn/v1/',  # base_url，也称为 api_base'api_key': 'ms-*',    }    # Define a group chat agent from the CFGS    bot = DocQA(llm=llm)    # Chat    message_list = []    print_text = ""while True:        # 测试输入：你的网名是？        query = input('user question: ')        message_list.append(Message('user', query, name="xudj"))        response = []for response in bot.run(messages=message_list, knowledge="你的网名叫小明"):            # 输出            print_text = typewriter_print(response, print_text)        message_list.extend(response)
```

这里通过继承Agent实现一个自定义的Agent，可以通过在调用Agent的run传入knowledge内容来补充系统提示词上下文内容。如示例，knowledge赋值你的网名叫小明，在提问时问：你的网名是？Agent便会返回小明。

**05 最后**

有了上面的学习和使用，再来看看整个Qwen-Agent的框架设计和实现，会发现其使用了几个核心的设计模式：

* 1、策略模式：

  不管是LLM、Tool、Agent，都有一个顶层基础的类，其余实现均继承该基础类。这样在使用不同实现时，通过参数便可以灵活选择。
* 2、模板方法：

  以LLM组件BaseChatModel顶层类举例，定义了chat方法，该方法内部会选择性调用流式对话或非流式对话方法，而这两个方法被@abstractmethod修饰成抽象方法，将由具体的实现类来实现逻辑。
* 3、工厂方法：

  自定义LLM实现时，通过@register\_llm('name')注解修饰，是一种装饰器实现，这样项目编译的时候，会把自定义的LLM实现类加入到全局的LLM\_REGISTRY中，结合策略以便通过参数查找并使用到对应自定义实现。

最后，Qwen-Agent框架虽然以开源形式提供了源码，不过官方并未配备详尽的使用文档。此外，源码中部分关键参数的说明也存在缺失，可能对使用者造成些许障碍，好在其结构相对简洁。

1

**END**

1