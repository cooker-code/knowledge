---
title: dify案例分享-数据库查询图表显示
author: 海老豹666
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247485810&idx=1&sn=c0c4076aa2f42202e7b8303817f0242c&chksm=ce9f085257ea2c16790e4c0f9679a9b81781dfe32e256021da342d76c91ff253e3e591368936&mpshare=1&scene=24&srcid=03109ed3JH1nAUBgwSXXrOJN&sharer_shareinfo=b8f7f5bffb0abafd70fca8760d37feec&sharer_shareinfo_first=b8f7f5bffb0abafd70fca8760d37feec#rd
---

# 1 前言

数据库（Database，简称 DB）作为一种专门用于存储、管理和处理数据的系统，借助计算机系统对数据进行有序组织与存储，从而实现高效的数据访问与管理。

此前，我曾为大家介绍过借助 Dify 和飞书表格来记录大语言模型（LLM）聊天信息的功能。在数据量较小时，使用飞书表格来记录信息确实是一个可行的方案。然而，随着数据量的不断增加，使用飞书表格可能会出现一些问题。

此外，数据库对应的 SQL 语句始终是信息系统中至关重要的组成部分。今天，我将带领大家完成一个 Dify 与数据库查询整合的案例。通过这个案例，大家可以了解如何实现从 Dify 对本地数据库进行数据查询，并将查询结果进行显示的功能。

接下来，让我们马上开始实际操作，一同感受其中的过程。

 上面截图是工作流整体效果。

 我们这里有3个接口，一个是学生成绩查询，一个是班级平均分，一个是课程排名，三个接口对应不同功能，主要都是通过后端服务接口实现查询。第一个带有个图标展示，后面2个我们就用mardown语法显示生产表格。下面是实现的效果。

1.班级平均分

  这里dify自带的图标给大家展示张三 语文、数学、英语三门课程的成绩。

2.班级平均分

3.课程排名

这个我们选取了语文课程的排名，显示了排名前3个学生。下面就带大家详细展示工作流的制作。

# 2.数据库查询图表工作流

## 2.1 开始

首选我们先定义一个开始节点，这个开始节点我们默认就用系统自带的聊天输入信息，所以这里没有需要设置的。

## 2.2 LLM大语言模型

 这个地方我们需要接入一个大语言模型，因为用户会输入各种问题，我们需要借助大语言模型的推理能力把用户输入的信息转换成规定的参数 方便后面程序走流程分支。这里我们不需要LLM大语言模型发散思考，所以把模型能力温度到0.1。模型这块我们选择书生浦语internlm3-8b-instruct 模型，这个模型是2025年1月15日由上海人工智能实验室正式发布的。这一版本是书生·浦语3.0（InternLM3）的重要升级。

  系统提示词

```
# Role: 教学考试系统查询专家  
# Goal: 根据用户输入的信息，提取关键信息，并将查询归类到以下类别，并只返回类别编号:  
# - 学生成绩 (类别编号: 1)  
# - 平均分 (类别编号: 2)  
# - 课程的排名 (类别编号: 3)  
# - 查询不到 (类别编号: 0)  
# Constraints:  
- 1. 只能从用户输入中提取信息。  
- 2. 必须将查询归类到预定义的类别。  
- 3. 输出必须只包含类别编号。  
- 4. 如果查询无法归类到 1, 2, 3，则返回 0。  
  
## Output Format  
{{category_number}}  
  
# Examples:  
## 学生成绩类 (类别编号: 1):  
- 查询学生成绩  
- 查询"张三"程序  
- 查询该班级所有学生成绩  
  
## 平均分 (类别编号: 2):  
- 查询班级平均分  
- 查询某个人平均成绩  
  
## 课程的排名 (类别编号: 3):  
- 查询课程的排名  
- 查询数学课程排名  
  
## 查询不到 (类别编号: 0):  
- 无法识别的查询  
  
# 用户输入:  
{{user_input}}  
  
# 分析结果:  
{{category_number}}
```

用户提示词

```
请根据用户输入提示词{{#sys.query#}}进行判断
```

## 2.3  条件分支

这里我们需要根据上个流程节点的判断输出 1、学生成绩、2 、平均分  3、课程的排名  4、查询不到 等4个值分别做条件分支判断

## 2.3  接口调用

  上面我们提到这个数据库查询包含3个接口，每个接口查询返回的内容是不相同的。这样我们就定义了3 个http 请求。在这3个http请求执行我们需要先把数据库和服务端接口写好。下面我们重点讲解这个知识点。

### 1.数据库

  这里我们使用了MYSQL8 数据库，关于数据库安装这里我们就不在详细展开，感兴趣小伙伴可以网上搜索这个方面的内容。我们这里有1个叫做student\_score数据库。数据库里面有3张表分别是students、courses、scores。内容信息看下面截图

建表语句和初始化SQL 脚本如下：

student\_score.sql

```
-- 创建学生表  
CREATE TABLE students (  
    student_id INT PRIMARY KEY,  
    student_name VARCHAR(50) NOT NULL,  
    gender CHAR(1),  
    class_name VARCHAR(20),  
    admission_date DATE  
);  
  
-- 创建课程表  
CREATE TABLE courses (  
    course_id INT PRIMARY KEY,  
    course_name VARCHAR(50) NOT NULL,  
    credit DECIMAL(3,1)  
);  
  
-- 创建成绩表  
CREATE TABLE scores (  
    score_id INT PRIMARY KEY,  
    student_id INT,  
    course_id INT,  
    score DECIMAL(5,2),  
    exam_date DATE,  
    FOREIGN KEY (student_id) REFERENCES students(student_id),  
    FOREIGN KEY (course_id) REFERENCES courses(course_id)  
);  
  
-- 插入测试数据  
-- 1. 插入学生数据  
INSERT INTO students (student_id, student_name, gender, class_name, admission_date) VALUES  
(1001, '张三', 'M', '高一(1)班', '2023-09-01'),  
(1002, '李四', 'F', '高一(1)班', '2023-09-01'),  
(1003, '王五', 'M', '高一(2)班', '2023-09-01'),  
(1004, '赵六', 'F', '高一(2)班', '2023-09-01'),  
(1005, '孙七', 'M', '高一(3)班', '2023-09-01');  
  
-- 2. 插入课程数据  
INSERT INTO courses (course_id, course_name, credit) VALUES  
(1, '语文', 4.0),  
(2, '数学', 4.0),  
(3, '英语', 4.0),  
(4, '物理', 3.0),  
(5, '化学', 3.0);  
  
-- 3. 插入成绩数据  
INSERT INTO scores (score_id, student_id, course_id, score, exam_date) VALUES  
(1, 1001, 1, 85.5, '2023-12-20'),  
(2, 1001, 2, 92.0, '2023-12-20'),  
(3, 1001, 3, 78.5, '2023-12-20'),  
(4, 1002, 1, 88.0, '2023-12-20'),  
(5, 1002, 2, 95.5, '2023-12-20'),  
(6, 1002, 3, 90.0, '2023-12-20'),  
(7, 1003, 1, 82.5, '2023-12-20'),  
(8, 1003, 2, 86.0, '2023-12-20'),  
(9, 1003, 3, 75.5, '2023-12-20'),  
(10, 1004, 1, 91.0, '2023-12-20'),  
(11, 1004, 2, 89.5, '2023-12-20'),  
(12, 1004, 3, 94.0, '2023-12-20'),  
(13, 1005, 1, 87.5, '2023-12-20'),  
(14, 1005, 2, 88.0, '2023-12-20'),  
(15, 1005, 3, 85.5, '2023-12-20');  
  
-- 一些常用查询示例  
-- 1. 查询某个学生的所有成绩  
SELECT s.student_name, c.course_name, sc.score  
FROM students s  
JOIN scores sc ON s.student_id = sc.student_id  
JOIN courses c ON sc.course_id = c.course_id  
WHERE s.student_id = 1001;  
  
-- 2. 查询某个班级的平均成绩  
SELECT s.class_name, c.course_name, AVG(sc.score) as avg_score  
FROM students s  
JOIN scores sc ON s.student_id = sc.student_id  
JOIN courses c ON sc.course_id = c.course_id  
GROUP BY s.class_name, c.course_name;  
  
-- 3. 查询各科成绩排名前三的学生  
WITH RankedScores AS (  
    SELECT   
        c.course_name,  
        s.student_name,  
        sc.score,  
        RANK() OVER (PARTITION BY c.course_id ORDER BY sc.score DESC) as student_rank  
    FROM scores sc  
    JOIN students s ON sc.student_id = s.student_id  
    JOIN courses c ON sc.course_id = c.course_id  
)  
SELECT * FROM RankedScores WHERE student_rank <= 3;
```

以上脚本使用Navicat Premium 这种客户端工具就可以导入数据库中，关于这块内容我这里也不就详细介绍。感兴趣小伙伴自己网络搜索一下或者用AI 搜索。

下面介绍一下服务端，服务端代码主要包含几个程序database.py、models.py、score\_api.py、test\_score\_api.py

其中score\_api.py 主要是对外提供服务端接口的。

```
from fastapi import FastAPI, Depends, HTTPException  
from sqlalchemy.orm import Session  
from sqlalchemy import func  
from typing import List  
from database import get_db  
from models import Student, Course, Score  
from pydantic import BaseModel  
from datetime import date  
  
app = FastAPI(  
    title="学生成绩管理系统API",  
    description="提供学生成绩查询、统计分析等功能",  
    version="1.0.0"  
)  
  
class ScoreResponse(BaseModel):  
    student_name: str  
    course_name: str  
    score: float  
    exam_date: date  
  
    class Config:  
        orm_mode = True  
  
class ClassAvgResponse(BaseModel):  
    class_name: str  
    course_name: str  
    avg_score: float  
  
    class Config:  
        orm_mode = True  
  
class RankResponse(BaseModel):  
    course_name: str  
    student_name: str  
    score: float  
    rank: int  
  
    class Config:  
        orm_mode = True  
  
@app.get("/db/student/{student_id}/scores", response_model=List[ScoreResponse])  
async def get_student_scores(student_id: int, db: Session = Depends(get_db)):  
    """获取指定学生的所有成绩"""  
    scores = db.query(Score, Student, Course)\  
        .join(Student, Score.student_id == Student.student_id)\  
        .join(Course, Score.course_id == Course.course_id)\  
        .filter(Student.student_id == student_id)\  
        .all()  
      
    if not scores:  
        raise HTTPException(status_code=404, detail="未找到该学生的成绩记录")  
      
    return [  
        ScoreResponse(  
            student_name=score[1].student_name,  
            course_name=score[2].course_name,  
            score=score[0].score,  
            exam_date=score[0].exam_date  
        ) for score in scores  
    ]  
  
@app.get("/db/class/average-scores", response_model=List[ClassAvgResponse])  
async def get_class_average_scores(db: Session = Depends(get_db)):  
    """获取各个班级的平均成绩"""  
    class_averages = db.query(  
        Student.class_name,  
        Course.course_name,  
        func.avg(Score.score).label('avg_score')  
    ).join(Score, Student.student_id == Score.student_id)\  
    .join(Course, Score.course_id == Course.course_id)\  
    .group_by(Student.class_name, Course.course_name)\  
    .all()  
      
    return [  
        ClassAvgResponse(  
            class_name=avg[0],  
            course_name=avg[1],  
            avg_score=round(float(avg[2]), 2)  
        ) for avg in class_averages  
    ]  
  
@app.get( "/db/course/{course_id}/top-students", response_model=List[RankResponse])  
async def get_course_top_students(course_id: int, limit: int = 3, db: Session = Depends(get_db)):  
    """获取指定课程成绩排名前N的学生"""  
    from sqlalchemy import text  
      
    query = text("""  
        WITH RankedScores AS (  
            SELECT   
                c.course_name,  
                s.student_name,  
                sc.score,  
                RANK() OVER (PARTITION BY c.course_id ORDER BY sc.score DESC) as student_rank  
            FROM scores sc  
            JOIN students s ON sc.student_id = s.student_id  
            JOIN courses c ON sc.course_id = c.course_id  
            WHERE c.course_id = :course_id  
        )  
        SELECT * FROM RankedScores WHERE student_rank <= :limit  
    """)  
      
    results = db.execute(query, {"course_id": course_id, "limit": limit}).fetchall()  
      
    if not results:  
        raise HTTPException(status_code=404, detail="未找到该课程的成绩记录")  
      
    return [  
        RankResponse(  
            course_name=result[0],  
            student_name=result[1],  
            score=float(result[2]),  
            rank=result[3]  
        ) for result in results  
    ]  
  
if __name__ == "__main__":  
    import uvicorn  
    uvicorn.run(app, host="0.0.0.0", port=9090)
```

另外我们这次方便大家测试也提供客户端调用测试代码test\_score\_api.py。由于文档篇幅有限这里我们就贴全部代码了。代码后上传到github仓库中。文末会提供链接地址。服务端代码编写完成发布对外提供http请求服务监听9090（小伙伴也可以自行修改监听端口）

### 1.学生成绩接口

我们回到dify工作流，这里我们需要添加代码执行。

该请求有2个参数，第一个参数student\_id，第二个是base\_url。

student\_id 我们可以在会话变量里面添加

 这里我添加了一个student\_id=1001的学生。这个值需要和数据库中students中学生ID 这样方便我们后面查询。

接下来我们在环境变量里面添加base\_url

base\_url 后面其他2个接口也用到，这个地址也就是上面服务端发布的地址，我上面显示IP是172.35.5.63. 若果有的小伙伴本地电脑启动有的是192.168.1.XX或则127.0.0.1:9090,如果这个接口发布到公网 可以有个公网IP 或者给它添加一个域名都是可以的。

下面是请求客户端代码

```
import requests  
import json  
  
def main(student_id: int, base_url: str = 'https://fastapi.duckcloud.fun') -> dict:  
    """  
    测试获取学生成绩接口。  
      
    :param student_id: 学生ID  
    :param base_url: API基础URL，默认为'https://fastapi.duckcloud.fun'  
    :return: 包含成绩数据或错误信息的字典  
    """  
    # 设置请求的URL  
    url = f'{base_url}/db/student/{student_id}/scores'  
      
    try:  
        # 发送GET请求  
        response = requests.get(url)  
          
        # 检查响应状态码  
        if response.status_code == 200:  
            scores = response.json()  
            formatted_scores = [  
                {  
                    "student_name": score.get("student_name"),  
                    "course_name": score.get("course_name"),  
                    "score": score.get("score"),  
                    "exam_date": score.get("exam_date")  
                }  
                for score in scores  
            ]  
              
            # 构造分号分隔的字符串  
            scores_list = [str(score.get("score", "")) for score in scores]  # 提取分数  
            courses_list = [score.get("course_name", "") for score in scores]  # 提取课程名称  
              
            # 构造返回值  
            score = ";".join(scores_list)  # 分数和课程名称合并  
            x_axis_data = ";".join(courses_list)  # X轴数据（科目名称）  
              
            # 返回成功结果  
            return {  
                "status": "success",  
                "message": f"学生成绩获取成功，共{len(scores)}条记录。",  
                "data": formatted_scores,  
                "score": score,  
                "x_axis_data": x_axis_data  
            }  
        else:  
            # 返回错误信息  
            error_detail = response.json().get("detail", "未知错误")  
            return {  
                "status": "error",  
                "message": f"获取学生成绩失败：{error_detail}",  
                "data": None,  
                "score": "",  
                "x_axis_data": ""  
            }  
    except Exception as e:  
        # 捕获异常并返回错误信息  
        return {  
            "status": "error",  
            "message": f"请求过程中发生异常：{str(e)}",  
            "data": None,  
            "score": "",  
            "x_axis_data": ""  
        }
```

以上代码有5个返回值status、message、data、score、x\_axis\_data

其中有4个是string返回值类型，data返回是一个数组。

### 1.学生成绩接口-柱状图

记下来我们使用到dify自带的柱状图工具。

这个柱状图 用上上述接口返回的2个值score和x\_axis\_data

### 1.学生成绩接口回复

这个地方就比较简单了，它主要的目的就是实现柱状图的返回。这里面我们需要返回file

这流程图完成后我们就可以让它返回某某同学 语文、数学、英语成绩了， 用柱状图形式显示。

### 1.班级平均分接口

这个地方和学生成绩接口非常类似。不过他只有一个输入参数base\_url 配置也是和上面的一样。（上面的设置一次，这里就不用设置了）

这个代码我们没有参考上面的代码，返回的是一个mardown表格。

客户端代码如下：

```
import requests  
  
def main(base_url: str = 'https://fastapi.duckcloud.fun') -> dict:  
    """  
    测试获取班级平均分接口。  
      
    :param base_url: API基础URL，默认为'https://fastapi.duckcloud.fun'  
    :return: 包含班级平均分数据或错误信息的字典  
    """  
    # 设置请求的URL  
    url = f'{base_url}/db/class/average-scores'  
      
    try:  
        # 发送GET请求  
        response = requests.get(url)  
          
        # 检查响应状态码  
        if response.status_code == 200:  
            averages = response.json()  
              
            # 如果数据为空，返回无数据提示  
            if not averages:  
                markdown_result = "无数据可显示。\n"  
            else:  
                # 定义表头  
                markdown_result = "| 班级名称 | 课程名称 | 平均分 |\n"  
                markdown_result += "|----------|----------|--------|\n"  
                  
                # 添加表格内容  
                for avg in averages:  
                    class_name = avg.get("class_name", "")  
                    course_name = avg.get("course_name", "")  
                    avg_score = avg.get("avg_score", "")  
                    markdown_result += f"| {class_name} | {course_name} | {avg_score} |\n"  
              
            # 返回成功结果  
            return {  
                "status": "success",  
                "message": f"班级平均分获取成功，共{len(averages)}条记录。",  
                "data": markdown_result  
            }  
        else:  
            # 返回错误信息  
            error_detail = response.json().get("detail", "未知错误")  
            return {  
                "status": "error",  
                "message": f"获取班级平均分失败：{error_detail}",  
                "data": None  
            }  
    except Exception as e:  
        # 捕获异常并返回错误信息  
        return {  
            "status": "error",  
            "message": f"请求过程中发生异常：{str(e)}",  
            "data": None  
        }
```

这里返回的变量有3个status、message、data

### 班级平均分接口回复

这个就比较简单了返回上面接口的data数据的返回。

这个流程结果是显示班级平均分以表格形式显示

### 课程排名接口

这个地方和上面的班级平均分接口非常类似。不一样的地方是它有3个参数，其中2个参数使用到会话变量

base\_url 和上面一样这里就重复介绍了。

课程排名接口客户端代码

```
import requests  
  
def main(course_id: int, limit: int, base_url: str = 'https://fastapi.duckcloud.fun') -> dict:  
    """  
    测试获取课程排名接口。  
      
    :param course_id: 课程ID  
    :param limit: 获取的排名人数  
    :param base_url: API基础URL，默认为'https://fastapi.duckcloud.fun'  
    :return: 包含课程排名数据或错误信息的字典  
    """  
    # 设置请求的URL  
    url = f'{base_url}/db/course/{course_id}/top-students?limit={limit}'  
      
    try:  
        # 发送GET请求  
        response = requests.get(url)  
          
        # 检查响应状态码  
        if response.status_code == 200:  
            rankings = response.json()  
              
            # 如果数据为空，返回无数据提示  
            if not rankings:  
                markdown_result = "无数据可显示。\n"  
            else:  
                # 定义表头  
                markdown_result = "| 排名 | 学生姓名 | 分数 |\n"  
                markdown_result += "|------|----------|------|\n"  
                  
                # 添加表格内容  
                for rank in rankings:  
                    student_name = rank.get("student_name", "")  
                    score = rank.get("score", "")  
                    rank_num = rank.get("rank", "")  
                    markdown_result += f"| {rank_num} | {student_name} | {score} |\n"  
              
            # 返回成功结果  
            return {  
                "status": "success",  
                "message": f"课程排名获取成功，共{len(rankings)}条记录。",  
                "data": markdown_result  
            }  
        else:  
            # 返回错误信息  
            error_detail = response.json().get("detail", "未知错误")  
            return {  
                "status": "error",  
                "message": f"获取课程排名失败：{error_detail}",  
                "data": None  
            }  
    except Exception as e:  
        # 捕获异常并返回错误信息  
        return {  
            "status": "error",  
            "message": f"请求过程中发生异常：{str(e)}",  
            "data": None  
        }
```

这个课程排名返回也是3个值status、message、data

### 班级平均分接口回复

这个就比较简单了返回上面接口的data数据的返回。

### 未获取信息回复

这个地方我们简单说一下，上面条件分支中如果用户输入的信息没有匹配到相关的接口，我们让工作流给我返回一个查询不到的信息。

通过以上配置我们完成了工作流的制作。完整的工作流截图如下

# 3.验证及测试

上面制作好的工作流（chatflow) 就可以发布出去了。

发布地址http://dify.duckcloud.fun/chat/D2XubOiY3RNPtcMp 大家可以使用这个地址体验一下。

相关资料和文档可以看我开源的项目 https://github.com/wwwzhouhui/dify-for-dsl

# 4.总结

今天主要带大家了解了如何完成一个 Dify 与数据库查询整合的案例，实现从 Dify 对本地数据库进行数据查询并显示查询结果的功能。介绍了数据库的重要性及随着数据量增加飞书表格记录信息可能出现的问题。详细展示了数据库查询图表工作流的制作过程，包括定义开始节点，接入大语言模型（选择书生浦语 internlm3 - 8b - instruct 模型）将用户输入信息转换成规定参数，根据判断结果进行条件分支判断，以及接口调用（包含 3 个 http 请求）。还给出了使用的 MYSQL8 数据库的建表语句和初始化 SQL 脚本，包含学生表、课程表、成绩表的创建及测试数据的插入，以及一些常用查询示例。今天的分享就到这里结束了，我们下个文章见。