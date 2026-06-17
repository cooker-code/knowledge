---
title: pywebio 数据表格Ⅱ
author: Streamlit Web
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU2NzQxOTMzOA==&mid=2247484827&idx=1&sn=f65818346b7799dcaba338ec7c0737c3&chksm=fd14c11c993df410214ee57dd441ea4f92cf92f97f3ff84592682384581cb37390670929766e&mpshare=1&scene=24&srcid=0704CYJ1prz44lpUJuGpDpc3&sharer_shareinfo=bebbd379bbd5a4f2ead846821e97adf4&sharer_shareinfo_first=bebbd379bbd5a4f2ead846821e97adf4#rd
---

Streamlit 构建和共享数据应用的更快方式

* **Streamlit 快速将数据脚本转换为可共享的 Web 应用程序**
* **现代化AI与科学应用定制化**Web 开发利器****
* **现代化数据分析必备良器**

**码码是一种沉浸式娱乐 (ง •\_•)ง**

1  好久不见

Hello 各位 Streamlit 伙伴，最近工作有些忙碌，日常事务堆叠，再加 Power BI 重操旧业，更新节奏不得不慢了下来。感谢大家一直以来的关注，待闲暇些定尽快整理新内容与大家分享。

此帖应 ST 伙伴之约，将之前 已整理完的 《Pywebio 数据表格》 源码分享出来，希望对大家有帮助！

[pywebio 数据表格](https://mp.weixin.qq.com/s?__biz=MzU2NzQxOTMzOA==&mid=2247484477&idx=1&sn=8ab7c937f130197f12e85e83551236bb&scene=21#wechat_redirect)

2  源码

■ 安装依赖

```
pip install pywebiopip install pywebio_batterypip install pandas
```

■ 使用提示

请将脚本中的 3ea05127311949dbb348066ab49facbd.jpg 图片替换成本地图片资源文件并与脚本文件存储在同一文件夹，否则会引发报错

■ 脚本

```
from pywebio.output import *from pywebio_battery import *from pywebio.pin import *from pywebio import start_serverimport pandas as pd  
# 应用启动def test_datatale():    def data_create():        from random import random, choice        animal = '🐵🐶🐺🐱🦁🐯🦒🦊🦝🐮🐷🐗🐭🐹🐰🐻🐨🐼🐸🦓🐴🦄🐔🐲🦍🦧🐩🐪🦘🦨🦔🐢🦎🐊🐍🐇🐿🦖🦈🐬🐋'        id = [i for i in range(0, len(animal))]        checkbox = ['False' for i in range(0, len(animal))]        image = ['3ea05127311949dbb348066ab49facbd.jpg' for i in range(0, len(animal))]  # 替换gif 资源文件        title = ['假如生活欺骗了你，不要悲伤，不要心急！忧郁的日子里须要镇静：相信吧，快乐的日子将会来临！' for i in range(0, len(animal))]        lazy_index = [round(random(), ndigits=2) for i in range(0, len(animal))]        clever_index = [round(random(), ndigits=2) for i in range(0, len(animal))]        loyalty_index = [round(random(), ndigits=2) for i in range(0, len(animal))]        survival_index = [round(random(), ndigits=2) for i in range(0, len(animal))]        state = [choice(['启用', '关闭', '暂停']) for i in range(0, len(animal))]        edit = ['编辑' for i in range(0, len(animal))]        details = ['详情' for i in range(0, len(animal))]        delete = ['删除' for i in range(0, len(animal))]        data = {            'id': id,            'checkbox': checkbox,            'head_sculpture': list(animal),             'image': image,             'title': title,            'lazy_index': lazy_index,             'loyalty_index': loyalty_index,            'survival_index': survival_index,            'state': state,            'edit': edit,            'details': details,            'delete': delete,        }        data = pd.DataFrame(data=data)        data.index = data['id']        return data    def row_all(sr, sign):        def pin_save(button_save):            data = {                'id': pin.input_id,                'head_sculpture': pin.input_head_sculpture,                'checkbox': pin.input_checkbox,                'image': pin.input_image,                'title': pin.input_title,                'lazy_index': pin.input_lazy_index,                'loyalty_index': pin.input_loyalty_index,                'survival_index': pin.input_survival_index,                'state': pin.input_state,            }            if button_save == '保存':                datatable_update(instance_id='test_table1', row_id=pin.input_id, data=data)            elif button_save == '取消':                pass            close_popup()        def pin_insert(button_insert):            data = {                'id': pin.input_id,                'head_sculpture': pin.input_head_sculpture,                'checkbox': pin.input_checkbox,                'image': pin.input_image,                'title': pin.input_title,                'lazy_index': pin.input_lazy_index,                'loyalty_index': pin.input_loyalty_index,                'survival_index': pin.input_survival_index,                'state': pin.input_state,            }            if button_insert == '保存':                datatable_insert(instance_id='test_table1', row_id=sr[0], records=data)            elif button_insert == '取消':                pass            close_popup()  
        def pin_image(input_image):            if not input_image is None and 'https://' in input_image:                with use_scope(name='scope_image', clear=True):                    put_row(                        content=[                            put_textarea(name='input_image', rows=4, value=input_image, label='图像'),                            put_image(src=input_image, height='120px', title='图像')                        ]                    )  
        match sign:            case 'display':                # 图片读取                import os                path = os.getcwd()                img = open(path + '/' + sr.at["image"], 'rb').read()  
                # 弹出框                popup(                    title='详细内容',                     content=[                        put_column(                            content=[                                put_row(                                    content=[                                        put_markdown(f'''**id:** {sr.at["id"]}'''),  #                                         put_markdown(f'''**头像:** {sr.at["head_sculpture"]}'''),  #                                         put_markdown(f'''**选择:** {sr.at["checkbox"]}'''),  #                                     ]                                ),                                # None,                                put_column(                                    content=[                                        put_markdown('''**图像:** '''),  #                                         put_image(src=img, height='120px', title='图像'),                                    ],                                    size='25px'                                ),                                # None,                                put_column(                                    content=[                                        put_markdown('''**标题:** '''),  #                                         put_markdown(sr.at["title"]),  #                                     ],                                    size='25px'                                ),                                # None,                                put_row(                                    content=[                                        put_markdown(f'''**lazy_index:** {sr.at["lazy_index"]}'''),  #                                         put_markdown(f'''**loyalty_index:** {sr.at["loyalty_index"]}'''),  #                                         put_markdown(f'''**survival_index:** {sr.at["survival_index"]}'''),  #                                    ]                                ),                                # None,                                put_markdown(f'''**状态:** {sr.at["state"]}'''),  #                             ],                            size='35px 160px 90px 35px'                        ),                        put_collapse(                            title='更多内容',                             content=                            [                                put_image(src='https://www.jnnews.tv/a/10001/202312/3a09d267fdbf8fba8f7e033c69fdbed8.jpeg', width='600px'),                                put_markdown(                                    '''                                        快科技12月14日消息，中央气象台发布寒潮、暴雪、冰冻黄色预警，北京、河北等地有大到暴雪。在此期间，寒冷地区的新能源汽车又迎来新一轮大考。  
                                        12月13日，今冬以来最强寒潮已经启程。一直到16日，这轮强寒潮将自西向东、自北向南横扫我国大部地区，部分地区降温可达14℃以上。  
                                        寒潮影响期间，北方部分地区最低气温将接近或跌破同期极值；南方则会上演由暖到冷的大逆转，部分地区最高气温降幅可达20℃左右。  
                                        随着寒潮暴雪的来临，已经有不少新能源车主开始抱怨电池续航问题，这个问题在北方更明显一些，低温会导致电池的放电速率加快，同时电池的储电能力也会大幅降低。  
                                        这使得新能源汽车在寒冷冬季的续航能力大打折扣，这也让不少车主后悔购买此类车，不过与寒冬相比，电动车车主们更惧怕什么？“充电涨价！而且是悄悄地涨价！”  
                                        很多车主发现，多地的公共充电桩电费开始上调。有特斯拉车主直言，有些地区的特斯拉充电桩在优惠充电时段（21:00到次日9:00）的电价，从原本的1.08元/度，已经涨到了1.29元/度，“本来冬天电力就衰减，再加上充电费上涨，真养了个‘爹’了”。  
                                        另有网约车司机算账，称即使精打细算去充优惠时段的电，每个月还是会比以往多出400-500元的支出。                                    '''                                ),                             ],                            open=False)                    ],                    size='large',                    implicit_close=False,                    closable=True                    )            case 'edit':                # 图片读取                import os                path = os.getcwd()                img = open(path + '/' + sr.at["image"], 'rb').read()  
                # 弹出框                popup(                    title='编辑信息',                     content=[                        put_column(                            content=[                                put_row(                                    content=[                                        put_input(name='input_id', type='number', label='id', value=str(sr.at["id"]), readonly=True),  #                                         None,                                        put_input(name='input_head_sculpture', type='text', label='头像', value=sr.at["head_sculpture"]),                                        None,                                        put_checkbox(name='input_checkbox', options=['True', 'False'], label='选择', inline=True, value=sr.at["checkbox"])                                    ]                                ),                                # None,                                put_scope(                                    name='scope_image',                                     content=[                                        put_row(                                            content=[                                                put_textarea(name='input_image', rows=4, value=sr.at["image"], label='图像'),                                                put_image(src=img, height='120px', title='图像'),                                            ]                                        ),                                    ],                                ),                                # put_row(                                #     content=[                                #         put_textarea(name='input_image', rows=4, value=sr.at["image"], label='图像'),                                #         put_image(src=img, height='120px', title='图像'),                                #     ]                                # ),                                # None,                                put_textarea(name='input_title', label='标题', rows=3, value=sr.at["title"]),                                # None,                                put_row(                                    content=[                                        put_input(name='input_lazy_index', type='float', label='lazy_index', value=sr.at["lazy_index"]),                                        None,                                        put_input(name='input_loyalty_index', type='float', label='loyalty_index', value=sr.at["loyalty_index"]),                                        None,                                        put_input(name='input_survival_index', type='float', label='survival_index', value=sr.at["survival_index"]),                                    ]                                ),                                # None,                                put_radio(name='input_state', options=['启用', '关闭'], label='状态', value=sr.at["state"], inline=True)                            ],                            size='70px 160px 120px 70px'                        ),                        put_collapse(                            title='更多内容',                             content=                            [                                put_row(                                    content=[                                        put_textarea(name='input_image1', rows=5, value='https://www.jnnews.tv/a/10001/202312/3a09d267fdbf8fba8f7e033c69fdbed8.jpeg', label='图像'),                                        put_image(src='https://www.jnnews.tv/a/10001/202312/3a09d267fdbf8fba8f7e033c69fdbed8.jpeg', width='100%', title='图像'),                                    ]                                ),                                put_textarea(name='input_title1', label='内容', rows=9, value=                                    '''                                        快科技12月14日消息，中央气象台发布寒潮、暴雪、冰冻黄色预警，北京、河北等地有大到暴雪。在此期间，寒冷地区的新能源汽车又迎来新一轮大考。  
                                        12月13日，今冬以来最强寒潮已经启程。一直到16日，这轮强寒潮将自西向东、自北向南横扫我国大部地区，部分地区降温可达14℃以上。  
                                        寒潮影响期间，北方部分地区最低气温将接近或跌破同期极值；南方则会上演由暖到冷的大逆转，部分地区最高气温降幅可达20℃左右。  
                                        随着寒潮暴雪的来临，已经有不少新能源车主开始抱怨电池续航问题，这个问题在北方更明显一些，低温会导致电池的放电速率加快，同时电池的储电能力也会大幅降低。  
                                        这使得新能源汽车在寒冷冬季的续航能力大打折扣，这也让不少车主后悔购买此类车，不过与寒冬相比，电动车车主们更惧怕什么？“充电涨价！而且是悄悄地涨价！”  
                                        很多车主发现，多地的公共充电桩电费开始上调。有特斯拉车主直言，有些地区的特斯拉充电桩在优惠充电时段（21:00到次日9:00）的电价，从原本的1.08元/度，已经涨到了1.29元/度，“本来冬天电力就衰减，再加上充电费上涨，真养了个‘爹’了”。  
                                        另有网约车司机算账，称即使精打细算去充优惠时段的电，每个月还是会比以往多出400-500元的支出。                                    '''                                ),                             ],                            open=False                        ),                        put_actions(name='button_save', buttons=['保存', '取消']).style('float:right;')                    ],                    size='large',                    implicit_close=False,                    closable=True                    )                pin_on_change(name='input_image', onchange=pin_image, clear=True, init_run=False)                pin_on_change(name='button_save', onchange=pin_save, clear=True, init_run=False)            case 'remove':                datatable_remove(instance_id='test_table1', row_ids=sr)            case 'insert':                # 图片读取                # import os                # path = os.getcwd()                # img = open(path + '/python_pywebio/Income_calculator/App/image/' + sr.at["image"], 'rb').read()  
                # 弹出框                popup(                    title='编辑信息',                     content=[                        put_column(                            content=[                                put_row(                                    content=[                                        put_input(name='input_id', type='number', label='id', value=str(sr[1]), readonly=True),  #                                         None,                                        put_input(name='input_head_sculpture', type='text', label='头像', value=None),                                        None,                                        put_checkbox(name='input_checkbox', options=['True', 'False'], label='选择', inline=True, value=None)                                    ]                                ),                                # None,                                put_scope(                                    name='scope_image',                                     content=[                                        put_row(                                            content=[                                                put_textarea(name='input_image', rows=4, value=None, label='图像'),                                                put_image(src=None, height='120px', title='图像')                                            ]                                        ),                                    ],                                ),                                # None,                                put_textarea(name='input_title', label='标题', rows=3, value=None),                                # None,                                put_row(                                    content=[                                        put_input(name='input_lazy_index', type='float', label='lazy_index', value=None),                                        None,                                        put_input(name='input_loyalty_index', type='float', label='loyalty_index', value=None),                                        None,                                        put_input(name='input_survival_index', type='float', label='survival_index', value=None),                                    ]                                ),                                # None,                                put_radio(name='input_state', options=['启用', '关闭'], label='状态', value=None, inline=True)                            ],                            size='70px 160px 120px 70px'                        ),                        put_collapse(                            title='更多内容',                             content=                            [                                put_row(                                    content=[                                        put_textarea(name='input_image1', rows=5, value='', label='图像'),                                        put_image(src=None, width='100%', title='图像'),                                    ]                                ),                                put_textarea(name='input_title1', label='内容', rows=9, value=                                    '''  
                                    '''                                ),                             ],                            open=False                        ),                        put_actions(name='button_insert', buttons=['保存', '取消']).style('float:right;')                    ],                    size='large',                    implicit_close=False,                    closable=True                    )                pin_on_change(name='input_image', onchange=pin_image, clear=True, init_run=False)                pin_on_change(name='button_insert', onchange=pin_insert, clear=True, init_run=False)  
  
  
    df = data_create()    # print(df.iloc[11])    data = df.to_dict(orient='records')    # print(data)    put_datatable(        records=data,        id_field='id',        instance_id='test_table1',        actions=[            ('详情', lambda row_id: row_all(df.iloc[int(row_id)], 'display')),  #             ('编辑', lambda row_id: row_all(df.iloc[int(row_id)], 'edit')),  #             ('插入', lambda row_id: row_all((row_id, len(data)), 'insert')),            ('删除', lambda row_id: row_all(row_id, 'remove'))        ],        # height='auto'        theme='alpine',         column_args={'id': {'flex': 100}},        grid_args=dict(sortChanged=JSFunction("event", "console.log(event.source)"))    )  
def main():    test_datatale()  
start_server(    main,    port=8510,    debug=True,  # 代码更新重启应用    cdn=False    )
```

**>>下期预告**

《CSS 原生样式设计》

《Streamlit Flow 流程设计》

**《**nivo Line 折线图**》**

《Ai实例应用》

**《\***st功能合集**》**

《AgGrid 数据表格Ⅳ》

《AgGrid 表格实例Ⅴ》

《MUI 数据表格示例Ⅱ》

《MUI 卡片》

《自定义主题样式》

《超级类数据表格》

《AI集成聊天功能》

《用户浏览器关闭检测》

💩 : 最近工作有点忙，所以有些内容更新不及时，小伙伴们有问题可以直接问我或留言，不一定即时回复，但一定会回复 (ง •\_•)ง

**—— END ——**

■ 公众号：Streamlit Web

■ B站：六客叨叨 | ■ 知乎：六客叨叨

爬出低代码的天坑，拥抱脚本，关注我👇

**(ง •\_•)ง**

---

**书山有路勤为径 学海无涯苦作舟**