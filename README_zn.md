<p align="center">
  <img src="logo/北京邮电大学-logo-2048px.png" alt="北京邮电大学" height="130"/>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="logo/邮百工logo.png" alt="邮百工" height="130"/>
</p>

<h1 align="center">EngrEduBench</h1>

<p align="center"><b>工程教育能力评价标尺数据集</b></p>
<p align="center"><i>Engineering Education Competency Assessment Benchmark Dataset</i></p>

<p align="center">
  <img alt="samples" src="https://img.shields.io/badge/samples-3%2C173-blue"/>
  <img alt="target" src="https://img.shields.io/badge/target-30%2C000%2B-purple"/>
  <img alt="domains" src="https://img.shields.io/badge/domains-10-orange"/>
  <img alt="dimensions" src="https://img.shields.io/badge/capability%20dimensions-11-brightgreen"/>
  <img alt="license" src="https://img.shields.io/badge/license-CC--BY--NC--SA%204.0-lightgrey"/>
</p>

<p align="center">
  <b>简体中文</b> | <a href="README.en.md">English</a>
</p>

---

## 📖 简介

**EngrEduBench** 致力于建设业内首个**工程能力评价基准数据集**：将虚实场景中产生的学习行为轨迹转化为能力证据链，使评价从结果判定转向过程诊断，支撑高阶工程能力指标体系落地。

EngrEduBench 是一个面向工程教育与工程实践的大模型工程能力评测基准，用于系统性衡量大语言模型是否能够理解真实工程学习过程中的行为证据、识别能力表现、给出多维评分，并形成可解释的诊断评语与改进建议。基准沿用《华盛顿协议》与中国工程教育专业认证体系的 **11 项工程能力维度**（工程知识、问题分析、设计/开发解决方案、研究、使用现代工具、工程与可持续发展、职业规范、个人与团队、沟通、项目管理、终身学习），每个维度配 3 个可观测打分指标，形成 33 个细粒度评分点。

数据来源覆盖**两条互补的真实证据链**：

- **动态分支**：从 10 个工程领域（数字农业、智慧医疗、智能制造、信息通信、金融科技 …）的真实"工程师—专家—导师"多轮协作日志中提取结构化行为证据；
- **静态分支**：从北京邮电大学等高校学生科创立项 / 结项报告中抽取的工程实践记录。

两条数据流经过统一的"证据抽取 → 证据归因 → 多维打分 → 教师/督导一致性过滤"流水线，最终产出 **3,173 条**带有打分理由与汇总评语的高质量评测样本，外加 **480 条**经过人工校验的黄金基准（golden set）。

本数据集仍在持续建设中。当前开源版本提供 3,000+ 条高质量样本；后续我们将持续扩展到 **30,000+ 条**，不断优化证据覆盖、能力归因、评分一致性与黄金基准质量，并欢迎研究者、教师、学生和工程实践者共同参与数据集建设与改进。

> 本仓库**仅发布数据**；评测流水线、蒸馏脚本、微调代码不在本仓库范围内。

---

## 🗂️ 仓库结构

```
EngrEduBench/
├── README.md                          # 中文说明
├── README.en.md                       # English README
├── logo/                              # 北邮 & 邮百工 标识
│
├── 工程报告数据集/                       # ① 静态分支原始数据：学生科创报告
│   ├── 立项/                           #     40 份立项报告
│   ├── 结项/                           #     50 份结项报告
│   └── dataset_static_targeted/        #     10 份定向补采报告
│
├── 工程行为数据集/                       # ② 动态分支原始数据：跨领域多轮工程协作日志
│   ├── DigitalAgriculture_20/          #     数字农业  · 21
│   ├── DigitalEducation_20/            #     数字教育  · 20
│   ├── DigitalEntertainment_20/        #     数字娱乐  · 20
│   ├── EnvTech_10/  · EnvTech_10_2/    #     环境科技  · 10 + 10
│   ├── FinTech_20/                     #     金融科技  · 20
│   ├── InfoComm_20/                    #     信息通信  · 20
│   ├── SmartEnergy_20/                 #     智慧能源  · 20
│   ├── SmartHealthcare_20/             #     智慧医疗  · 20
│   ├── SmartManufacturing_20/          #     智能制造  · 20
│   ├── TransportLogistics_20/          #     交通物流  · 20
│   ├── 动态数据集（补）/                  #     补采样本   · 20
│   └── dataset_5/                      #     难例增补   · 5
│
├── 情境化证据数据集/                     # ③ 证据抽取结果（动/静合并）
│   ├── context_extract/                #     上下文（微场景）抽取
│   ├── evidence_extract/               #     原子证据抽取
│   │   ├── Interactive-data-evidence/  #         动态证据  · 3,454
│   │   ├── static-data-evidence/       #         静态证据  · 1,412
│   │   ├── 动态数据子集/  ·  静态数据子集/ ·  600/   # 子集 / 抽样切片
│   ├── unified_candidates/             #     第一批候选（4,000）
│   ├── unified_candidates_600/         #     扩充候选（8,419）
│   ├── Interactive_export_manifest.json
│   ├── static_export_manifest.json
│   ├── slice_manifest_targeted.json
│   ├── slice_manifest_inactive+supple.json
│   └── P4_原始候选构造配额方案.md          #     候选构造的配额方案说明
│
├── 工程能力评价体系数据集/                # ④ 最终评测集
│   ├── engredubench_kept.jsonl         #     ★ 主集合 · 3,173 条（推荐使用）
│   └── engredubench_dropped.jsonl      #     被一致性过滤掉的 827 条（仅供研究）
│
└── golden-base-data-480-evaluation/    # ⑤ 人工校验黄金基准 · 480 条评估 JSON
```

---

## 📊 数据规模总览


| #   | 数据集                               | 体量                                 | 角色           |
| --- | --------------------------------- | ---------------------------------- | ------------ |
| ①   | `工程报告数据集`                         | 100 份报告（立项 40 + 结项 50 + 定向 10）     | 静态分支原始语料     |
| ②   | `工程行为数据集`                         | 226 份多轮协作日志，覆盖 10 个工程领域            | 动态分支原始语料     |
| ③   | `情境化证据数据集`                        | 12,419 条候选 + 4,866 条原子证据           | 证据抽取与候选构造    |
| ④   | `工程能力评价体系数据集`                     | **3,173** 条（kept） + 827 条（dropped） | **最终评测集**    |
| ⑤   | `golden-base-data-480-evaluation` | **480** 条                          | **人工校验黄金基准** |


---

## 🎯 11 项工程能力维度


| 维度        | 维度（英文）                            |
| --------- | --------------------------------- |
| 工程知识      | Engineering Knowledge             |
| 问题分析      | Problem Analysis                  |
| 设计/开发解决方案 | Design / Development of Solutions |
| 研究        | Investigation                     |
| 使用现代工具    | Modern Tool Usage                 |
| 工程与可持续发展  | The Engineer and the World        |
| 职业规范      | Ethics                            |
| 个人与团队     | Individual and Teamwork           |
| 沟通        | Communication                     |
| 项目管理      | Project Management                |
| 终身学习      | Lifelong Learning                 |


每个维度下设 3 个可观测打分指标（共 33 个），采用 1-5 分连续打分；维度总分按 `0.3 / 0.3 / 0.4` 加权求和。

---

## 🧬 数据格式

### ④ 最终评测集 `engredubench_kept.jsonl`（推荐入口）

每行一个 JSON 对象，主字段：

```jsonc
{
  "sample_id": "InfoComm_20__22598982_..._008_工程开发_8544a92f6c",
  "manifest": {                       // 样本元数据
    "modality": "dynamic",            //   dynamic / static
    "domain": "InfoComm",
    "stage": "工程开发",
    "evidence_count": "7",
    "quota_group": "InfoComm::工程开发"
  },
  "decision": {                       // 一致性过滤结果
    "flagged_dims": "2",
    "kept": "True"
  },
  "scores": {                         // 11 维评分 + 自然语言理由
    "工程知识":     { "score": 4.1, "reason": "观测点1：4.2分 …" },
    "问题分析":     { "score": 3.7, "reason": "…" },
    "...":          { "score":  ... , "reason": "…" }
  },
  "evidence":   { /* 按维度组织的证据片段，源自原始日志/报告 */ },
  "evaluation": "综合评语 …",
  "suggestions": "改进建议 …"
}
```

### ⑤ 黄金基准 `golden-base-data-480-evaluation/*.json`

每个文件为一条样本的完整人工校验评测结果，schema 与上一致；可作为**评测器 / 微调模型的对照基准**。

### ② 动态行为数据格式

```jsonc
[{
  "domain": "数字农业",
  "stages": [{
    "stageName": "需求与战略",
    "microScenes": [{
      "microScene": "……微场景情境描述……",
      "evidence": [
        { "behaviorLabel": "思考",     "behaviorEvidence": "……" },
        { "behaviorLabel": "分析取舍", "behaviorEvidence": "……" }
      ]
    }]
  }]
}]
```

### ① 静态报告数据格式

与动态类似，按 `domain → stages → microScenes` 组织；`data` 字段保留原始报告文本片段。

---

## 🚀 快速开始

### 加载主评测集

```python
import json

with open("工程能力评价体系数据集/engredubench_kept.jsonl", encoding="utf-8") as f:
    samples = [json.loads(line) for line in f]

print(f"共 {len(samples)} 条样本，第一条 11 维分数：")
print({d: s["score"] for d, s in samples[0]["scores"].items()})
```

### 加载黄金基准

```python
import json, pathlib

gold_dir = pathlib.Path("golden-base-data-480-evaluation")
gold = [json.loads(p.read_text(encoding="utf-8")) for p in gold_dir.glob("*.json")]
print(f"黄金基准样本数：{len(gold)}")
```

---

## 📐 推荐评测指标

为便于横向比较，推荐使用以下指标对模型输出与黄金基准对齐：


| 指标                                | 维度粒度                            | 用途            |
| --------------------------------- | ------------------------------- | ------------- |
| **QWK**（Quadratic Weighted Kappa） | 每个能力维度                          | 衡量打分一致性       |
| **Pearson r**                     | 每个能力维度                          | 衡量打分线性相关性     |
| **CosSim**（BGE-zh）                | `evaluation` / `suggestions` 文本 | 衡量评语与建议的语义一致性 |


---

## 📜 许可协议

本数据集采用 **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International（CC BY-NC-SA 4.0）** 协议发布。

- ✅ 允许：学术研究、教学、模型评测、二次衍生（同协议共享）
- ❌ 禁止：商业用途、直接售卖
- ⚠️ 使用时请保留原始署名

完整协议见：[https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)

---

## 📑 引用

如果本数据集对您的研究有帮助，请引用：

```bibtex
@misc{engredubench2026,
  title        = {EngrEduBench: A Chinese-Language Engineering-Capability
                  Benchmark for Large Language Models},
  author       = {EngrEduBench Team, Beijing University of Posts and Telecommunications},
  year         = {2026},
  howpublished = {\url{https://github.com/<your-org>/EngrEduBench}},
  note         = {Open-source dataset release}
}
```

> 📄 配套论文已投稿至 EMNLP 2026，发表后将在此处更新引用条目。

---

## 🤝 致谢

本基准由 **北京邮电大学** 与 **邮百工** 联合构建并发布。

<p align="center">
  <img src="logo/北京邮电大学-logo-2048px.png" alt="北京邮电大学" height="80"/>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="logo/邮百工logo.png" alt="邮百工" height="80"/>
</p>

感谢全体参与数据标注、专家咨询、报告整理工作的师生与工程师。所有原始报告作者已知情同意其数据被用于学术研究目的。

---

## 📮 联系方式

- 议题反馈：请在本仓库的 [Issues](../../issues) 中提出
- 合作 / 商业授权：请通过北京邮电大学相关院系或邮百工对接

---



**EngrEduBench**  ·  让大模型的工程能力可被严肃地度量
  

Made with ❤️ by BUPT × 邮百工

