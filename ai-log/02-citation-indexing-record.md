# AI 使用记录：综述引用索引整理

## 1. 任务概述

本次使用 AI 的任务是：为最终论文综述补充正文引用索引，使关键论断能够对应到原论文的具体章节、定理或引理，并使相关工作能够对应到相应的原始文献。

本次任务的重点不是继续扩写综述内容，而是改善已有正文的可核查性。AI 根据论文原文和已有综述，协助建立“正文论断—论文依据—参考文献”之间的对应关系，并将引用写入 LaTeX 文档。

## 2. 原始材料

- 论文原文：`paper/A Hash Table Without Hash Functions, and How to Get the Most Out of Your Random Bits.pdf`
- 综述源文件：`review/review.tex`
- 综述 PDF：`review/review.pdf`
- 严谨性修订笔记：`notes/07-相关工作与严谨性修订.md`

本次 AI 处理以仓库中的论文原文为主要依据。对于论文发表后的相邻研究，引用信息以论文页面、正式出版页面或 DOI 信息为依据。

## 3. AI 使用目的

本次使用 AI 的具体目的包括：

1. 找出综述中需要文献支持的关键技术论断。
2. 将主要结果对应到原论文的 Theorem 2、Theorem 5、Theorem 7 和 Lemma 3。
3. 将模型边界对应到原论文的 Sections 2、6 和 Appendix A。
4. 为相关工作补充正文编号引用，而不是只在文末列出论文名称。
5. 整理统一的参考文献列表，并检查 LaTeX 引用能否正确解析。

## 4. AI 参与内容

### 4.1 原文依据定位

- 从论文 PDF 中定位主要定理、引理、模型定义和附录结论。
- 核对 amplified rotated trie、budget rotated trie 和 succinct transformation 的结果口径。
- 核对 failure semantics、key/value 长度、机器字长、操作阶段和概率保证等模型边界。

### 4.2 正文引用映射

- 为主要结果补充精确到定理或引理的引用。
- 为 rotated radix trie、McDiarmid 分析、gradually-increasing-independence 和 Raman--Rao reduction 等技术描述补充章节引用。
- 对固定操作序列、failure 含义和大 universe 情形等边界性表述补充原论文依据。

### 4.3 相关工作整理

- 为 universal hashing、perfect hashing 和 cuckoo hashing 补充原始文献。
- 为 fusion tree、dynamic fusion node、polynomial hash functions、succinct dynamic dictionaries、load-balancing families 和 simple tabulation hashing 补充对应引用。
- 核对 `Modern Hashing Made Simple` 与 `Optimal Non-Oblivious Open Addressing` 的出版信息，并继续把它们表述为主题相邻研究，而不是原论文的直接定理级后续。

### 4.4 LaTeX 与 PDF 检查

- 将原有的简单文献清单改为带 `\bibitem` 索引的参考文献表。
- 检查所有 `\cite` 项是否能够解析。
- 重新编译 `review/review.pdf`，并检查正文引用、分页和参考文献排版。

## 5. AI 使用方式说明

本次 AI 使用方式属于“文献定位、引用整理和格式校验辅助”。AI 没有替代论文原文成为结论依据，也没有因为模型判断而新增不可核查的学术结论。

具体来说，AI 的角色主要是：

- 提议哪些正文论断需要引用；
- 从论文原文中定位相应依据；
- 整理参考文献元数据；
- 修改 LaTeX 引用结构并执行编译检查。

引用是否成立以论文原文、原始相关工作和可核验的出版信息为准。AI 生成的引用映射需要能够被这些材料直接复查。

## 6. 用户提出的关键约束

本次任务遵循的关键要求包括：

- 先保留成员贡献修改的独立提交，再处理引用问题。
- 重点解决关键论断和相关工作的正文引用缺失。
- 不继续堆积翻译或资料。
- 保留已有综述的评价结构和研究边界，不把相邻论文夸大为直接后续工作。

## 7. 最终输出

本次任务修改并保留的结果文件为：

- 带正文引用索引的 LaTeX 综述：`review/review.tex`
- 重新编译的综述 PDF：`review/review.pdf`

修改后的正文包含可跳转的编号引用，并在文末保留与正文引用对应的参考文献表。

## 8. 记录用途说明

本记录用于说明：

- AI 在综述引用补充过程中的实际参与范围；
- 引用索引所依据的原始材料；
- AI 在原文定位、参考文献整理、LaTeX 修改和 PDF 校验中的作用；
- 最终引用仍需以论文原文和原始出版信息为判断依据。

如果后续需要说明“综述中的引用如何形成、AI 参与到什么程度”，可以直接参考本记录。
