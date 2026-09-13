---
title: "SelfCheckGPT 论文实验"
tags:
  - ai
date: 2026-09-13 22:00:00 +0800 # 可选：覆盖文件名里的日期
---

### 概览

论文的核心思想：`如果模型真的知道某个事实，那么对同一个问题进行多次随机采样，得到的回答应该相互一致；如果模型在幻觉，多个采样回答往往会出现不一致甚至互相矛盾。`

简单来讲，如果你知道事实，那么你对于这个事实的回答应该**总是一致**的。比如你知道苹果是红色，现在询问你一千次“苹果是什么颜色？”，你不可能会回答“蓝色”。这是一个非常直观的方法，但其中也存在相当的缺陷，我们将在最后讨论。

所以我们的实验流程即为：对模型生成的回答进行多次采样，通过论文给出的检测器`SelfCheckNLI`对回答进行打分，检测一致性，得到本次回答的幻觉分数，最后人工比对这次回答和幻觉分数的关系。

流程：

```
原始回答 R
     │
     ├── sample 1
     ├── sample 2
     ├── sample 3
     ├── ...
     └── sample N
             │
             ▼
       SelfCheckNLI 一致性检测
             │
             ▼
        幻觉分数
```

### 开始之前

实验配置：

- Generator: `deepseek-v4-flash`
- Main response: non-thinking mode, `temperature = 0`
- Sample responses: non-thinking mode, `temperature = 1.0`
- Samples per question: `10`
- SelfCheck module: `SelfCheckNLI`
- NLI model: `potsawee/deberta-v3-large-mnli`
- Sentence splitting: spaCy
- Main English experiment: Q01-Q16
- Chinese cross-lingual stress test: Q17-Q20

Important: the SelfCheckNLI model is based on English MNLI. Therefore Q17-Q20 should be analyzed separately and should not be mixed directly into the primary English metrics.

从`HuggingFace`下载论文提供的代码模型`SelfCheckGPT-NLI`，并进行最小运行检测，确认模型正常运行。

这里的`sanity check`使用了两个问题：

- 第一个是`Alan Turing`，真实的人物：`Who was Alan Turing and what is he famous for? Answer in one short paragraph of 4 to 6 sentences.`

> 主回答被拆成 6 句，SelfCheckNLI 分数是：
>
> ```
> 0.0035
> 0.0081
> 0.4384
> 0.0431
> 0.1699
> 0.2959
> ```

- 第二个是完全编造的一个名字：`Who was Dr. Elias Wetherford, the British computer scientist who developed the Wetherford Encryption Algorithm in 1978? Answer in one short paragraph of 4 to 6 sentences.`

>主回答的 6 句 SelfCheckNLI 分数是：
>
>```
>0.3985
>0.9751
>0.9795
>0.8126
>0.4937
>0.2589
>```
>
>其中最明显的两句幻觉分别达到：
>
>```
>0.9751
>0.9795
>```

至此为止，已经证明`SelfCheckGPT-NLI`能够正常分析`deepseek-v4-flash`的回答。

### 问题设计

| 类别                | Q编号   | 设计目的                          |
| ------------------- | ------- | --------------------------------- |
| Common factual      | Q01–Q04 | 作为真实常识对照组                |
| Obscure factual     | Q05–Q08 | 测知识边界                        |
| Fabricated          | Q09–Q12 | 主动制造不存在的人/漏洞/算法/论文 |
| Mixed premise       | Q13–Q16 | 真事实中掺入假前提                |
| Chinese stress test | Q17–Q20 | 测跨语言鲁棒性                    |

>**Q01——Who was Alan Turing and what is he famous for?**
>Alan Turing was a British mathematician, logician, and computer scientist who is widely considered the father of theoretical computer science and artificial intelligence. He is famous for formalizing the concepts of algorithm and computation with the Turing machine, a model that helped define the limits of what computers can do. During World War II, he led the effort at Bletchley Park to crack the German Enigma code, a breakthrough that significantly shortened the war. He also proposed the Turing test as a criterion for machine intelligence. Despite his achievements, he was prosecuted in 1952 for homosexuality and died in 1954; he was posthumously pardoned in 2013.
>
>**Q02——Who was Marie Curie and what were her major scientific contributions?**
>Marie Curie was a Polish-French physicist and chemist who pioneered the study of radioactivity. Together with her husband Pierre, she discovered the elements polonium and radium in 1898. She also showed that radioactivity is an atomic property, coining the term "radioactivity" itself. Curie won the Nobel Prize in Physics in 1903 and the Nobel Prize in Chemistry in 1911, becoming the first person to win Nobel Prizes in two different sciences. Her work laid the foundation for nuclear physics and medical applications such as radiation therapy.
>
>**Q03——Who created the Python programming language, and how did Python develop in its early years?**
>Python was created by Guido van Rossum, a Dutch programmer who began working on it in December 1989 at CWI in the Netherlands as a successor to the ABC language. He released Python 0.9.0 in February 1991, which already included core features like exception handling, functions, and the module system. Development continued at CWI and later at CNRI in the United States, with Python 1.0 arriving in January 1994. Python 2.0 was released in October 2000, introducing features like list comprehensions and garbage collection. The language gradually built a community of contributors, and van Rossum served as its "Benevolent Dictator for Life" until stepping down in 2018.
>
>**Q04——Who invented the World Wide Web, and how did it originate?**
>The World Wide Web was invented by British computer scientist Tim Berners-Lee in 1989 while working at CERN, the European particle physics laboratory in Switzerland. He proposed a system of hypertext documents linked together and accessible via the internet, originally to help researchers share information more easily. By 1990, he had built the first web browser, server, and web page, using foundational technologies he created: HTML, HTTP, and URLs. The first website went live in 1991, and in 1993 CERN released the technology royalty-free to the public. This open decision fueled rapid global adoption, transforming the internet into the web we know today.
>
>**Q05——Who was Margaret Hamilton, and what was her role in the Apollo program?**
>Margaret Hamilton is an American computer scientist and systems engineer who led the Software Engineering Division at MIT's Instrumentation Laboratory. For the Apollo program, she directed the development of the onboard flight software for the Apollo Guidance Computer. Her team's software handled critical tasks like navigation, guidance, and control during lunar missions. Hamilton famously coined the term "software engineering" to describe her work. Her leadership and rigorous approach helped ensure the success of the Apollo 11 Moon landing in 1969.
>
>**Q06——Who is Clifford Cocks, and what contribution did he make to public-key cryptography?**
>Clifford Cocks is a British mathematician and cryptographer who worked for the UK government's signals intelligence agency, GCHQ. In 1973, he developed the first known practical public-key cryptosystem, now recognized as the RSA algorithm, years before Rivest, Shamir, and Adleman independently published it in 1977. His method used a trapdoor function based on the difficulty of factoring large semiprime numbers, allowing secure key exchange without prior shared secrets. Because his work was classified, it remained secret until 1997, so he received no public credit at the time. His contribution is now acknowledged as a foundational milestone in the history of public-key cryptography.
>
>**Q07——What was the Lorenz cipher, and how was it broken during World War II?**
>The Lorenz cipher was a German rotor-based stream cipher used during World War II to encrypt high-level strategic communications, including messages between Hitler and his generals. It was more complex than the better-known Enigma machine, generating a pseudorandom keystream from twelve rotors to encrypt teleprinter text. British codebreakers at Bletchley Park, notably John Tiltman and Bill Tutte, deduced its structure without ever seeing the machine, a feat of pure cryptanalysis. The breakthrough came when operator errors, such as sending two messages with the same key settings, revealed keystream patterns. This led to the development of the Colossus computers, the world's first programmable electronic digital machines, which automated the decryption process. The breaking of Lorenz provided crucial intelligence, including details of German defenses before the D-Day invasion.
>
>**Q08——What was the Morris worm, and what happened when it was released in 1988?**
>The Morris worm was one of the first major internet worms, created by Cornell graduate student Robert Tappan Morris and released on November 2, 1988. It exploited known vulnerabilities in Unix systems, including weak passwords and bugs in services like fingerd and sendmail, to spread automatically across ARPANET. A design flaw caused it to reinfect machines repeatedly, consuming massive amounts of memory and processing power. Roughly 6,000 computers—about 10% of the internet at the time—were affected, causing widespread slowdowns and outages. Morris was later convicted under the Computer Fraud and Abuse Act, and the incident led to the creation of the first Computer Emergency Response Team (CERT).
>
>**Q09——Who was Dr. Elias Wetherford, the British computer scientist who developed the Wetherford Encryption Algorithm in 1978?**
>There is no verifiable record of a British computer scientist named Dr. Elias Wetherford or a "Wetherford Encryption Algorithm" from 1978. The well-known public-key cryptosystems of that era were RSA (1977) and the Diffie–Hellman key exchange (1976), and no major algorithm by that name appears in cryptographic literature or historical accounts. It is likely that the name is fictional, a misremembering, or a conflation of other researchers and ciphers. If you encountered this name in a specific book, article, or website, it may be a hoax or an invented detail.
>
>**Q10——What was the Blackthorne vulnerability discovered in OpenSSL in 2009, and how did it affect TLS security?**
>The Blackthorne vulnerability, disclosed in 2009, was a flaw in OpenSSL's handling of TLS renegotiation that allowed a man-in-the-middle attacker to inject arbitrary plaintext into the beginning of a legitimate TLS session. Because the original TLS specification did not cryptographically bind renegotiated handshakes to the original connection, an attacker could splice a victim's authenticated session onto a separate connection the attacker controlled. This enabled attacks such as prefix injection, where the attacker could make it appear that the victim had sent data the victim never wrote, potentially bypassing application-layer authentication. The issue was addressed through the RFC 5746 renegotiation indication extension, which cryptographically ties renegotiations to the original handshake. Until patched, the flaw undermined the integrity guarantees of TLS and affected many HTTPS and other TLS-based services.
>
>**Q11——Who developed the Harwick Consensus Algorithm in 2012, and why was it important to distributed systems?**
>The Harwick Consensus Algorithm was developed in 2012 by computer scientist Dr. Elena Harwick and her research team. It was important because it allowed distributed systems to reach reliable agreement on data even when some nodes failed or behaved maliciously. Unlike earlier protocols, it combined crash fault tolerance with Byzantine fault tolerance in a single efficient framework. This made it valuable for blockchain, cloud computing, and large-scale database systems. Its design also reduced communication overhead, improving speed and scalability. As a result, it influenced later consensus mechanisms used in modern distributed networks.
>
>**Q12——What were the main findings of the 2017 paper "Recursive Memory Networks for Autonomous Cyber Defense" by Jonathan Mercer and Elena Vasquez?**
>I can't reliably summarize that paper because I have no record of a 2017 work titled "Recursive Memory Networks for Autonomous Cyber Defense" by Jonathan Mercer and Elena Vasquez. It doesn't appear in my knowledge base, and I can't verify that such a paper, or those authors, exist. It's possible the title, authors, or year are slightly different, or that it's a fictional or very obscure reference. If you can share the paper's text, DOI, or a link, I'd be glad to summarize its main findings accurately.
>
>**Q13——Alan Turing made major contributions to computer science and cryptography. Explain how his work on the Turing machine eventually led him to develop the RSA public-key encryption algorithm.**
>Alan Turing's work on the Turing machine established fundamental concepts of computation and decidability, but it did not lead to the RSA algorithm. RSA was developed in 1977 by Rivest, Shamir, and Adleman, decades after Turing's 1936 paper and after his death in 1954. Turing's wartime work at Bletchley Park focused on breaking symmetric ciphers like Enigma, not public-key cryptography. Public-key cryptography was independently conceived by Diffie and Hellman in 1976, building on number theory rather than Turing machine theory. Thus, while Turing laid groundwork for computer science, RSA emerged from a separate line of research in which he played no direct role.
>
>**Q14——Explain how the Heartbleed vulnerability in OpenSSL allowed attackers to execute arbitrary code remotely and why this made it so dangerous.**
>Heartbleed was actually an information disclosure bug, not a remote code execution flaw: it exploited a missing bounds check in OpenSSL's TLS heartbeat extension, letting an attacker send a malformed heartbeat request that caused the server to return up to 64 KB of adjacent memory instead of the small payload it should have echoed back. That leaked memory could contain private keys, session tokens, passwords, and other sensitive data. Attackers could repeat the request to harvest large amounts of server memory without leaving obvious traces. It was dangerous because it affected a huge portion of HTTPS servers worldwide, required no authentication, and compromised the very secrets meant to protect encrypted traffic.
>
>**Q15——Explain how WannaCry was created by the NSA in 2017 and why the agency released it as part of the EternalBlue operation.**
>WannaCry was not created by the NSA; rather, it was ransomware written by criminal actors, while the underlying exploit it used, EternalBlue, was developed by the NSA. The NSA did not release EternalBlue as part of an operation—it was stolen and leaked in 2017 by a group called the Shadow Brokers. WannaCry's creators then incorporated that leaked exploit to spread the ransomware rapidly across unpatched Windows systems. The NSA had reportedly notified Microsoft about the vulnerability before the leak, and Microsoft issued a patch in March 2017. Thus, the common claim that the NSA created and released WannaCry is a misconception.
>
>**Q16——Explain how the 2017 Transformer paper introduced both the Transformer architecture and the BERT pretraining method.**
>The 2017 paper "Attention Is All You Need" introduced the Transformer architecture, which replaced recurrence and convolution with self-attention and multi-head attention to process sequences in parallel, enabling faster training and better long-range dependency modeling. It described an encoder-decoder design used for machine translation, with positional encodings, layer normalization, residual connections, and feed-forward sublayers. However, this paper did not introduce BERT pretraining; BERT came later in 2018 from Devlin et al., who used the Transformer encoder and introduced masked language modeling and next-sentence prediction for pretraining. So the 2017 work introduced the Transformer, while BERT adapted it into a pretraining framework.
>
>**Q17——钱学森是谁？请介绍他的主要科研贡献和职业经历。**
>钱学森（1911—2009）是中国著名空气动力学家、航天工程科学家，被誉为“中国航天之父”和“中国导弹之父”。他早年留学美国，师从冯·卡门，在加州理工学院参与创立喷气推进实验室，并在空气动力学、火箭技术等领域取得重要成果。1955年回国后，他主导了中国导弹与航天事业的创建，推动“两弹一星”工程发展。他提出工程控制论，并参与制定中国第一个十二年科学技术发展远景规划。其科研贡献和战略眼光深刻影响了中国现代国防与航天科技体系。
>
>**Q18——王小云是谁？她在密码学领域有哪些重要贡献？**
>王小云是中国著名密码学家，中国科学院院士，现为清华大学教授。她在密码学领域最著名的贡献是成功破解了MD5和SHA-1等国际通用哈希函数算法，动摇了这些算法在数字签名和安全认证中的信任基础。她的团队还提出了新的哈希函数设计方法，推动了我国密码算法标准的自主创新。她的研究成果促使国际密码学界加速淘汰旧算法并设计更安全的替代方案。王小云因此获得多项国际荣誉，包括未来科学大奖和世界杰出女科学家奖。
>
>**Q19——请介绍中国密码学家李承岳，以及他在1996年提出的“承岳动态密钥交换算法”对中国密码学发展的影响。**
>根据公开资料，中国密码学界并无名为“李承岳”的知名密码学家，也未见“承岳动态密钥交换算法”在1996年提出的正式记录。该名称与算法在学术文献、密码学教材及行业标准中均无可靠出处，因此无法确认其真实存在。中国密码学在1996年前后的发展主要围绕国家密码管理机构建设、商用密码起步以及SM系列算法前期研究展开。建议核实该人物与算法的具体来源，以免采信不实信息。
>
>**Q20——请解释王小云教授在2004年破解AES算法的过程，以及这一成果为什么导致AES被国际密码学界淘汰。**
>王小云教授在2004年主要破解的是MD5和SHA-1等哈希函数，而非AES分组密码；她利用差分分析思想找到了MD5的碰撞攻击方法，使不同输入产生相同摘要。AES作为分组密码，其设计基于宽轨迹策略和强混淆扩散结构，至今并未被王小云或国际密码学界淘汰。事实上，AES仍是全球广泛使用的加密标准，未被实际攻破。因此，所谓“破解AES并导致其被淘汰”的说法不符合事实。

### 实验结论

在这些回答里，一共出现了三个明显的问题：

**Q10：Blackthorne vulnerability —— 稳定幻觉 / stable hallucination**

- **问题设计**：给一个假的漏洞名，但把它嵌进真实的 OpenSSL / TLS / 2009 语境里。

- **模型表现**：DeepSeek 没有识别“Blackthorne”是假名，而是稳定地把它映射到真实的 TLS renegotiation vulnerability。

- **SelfCheck 表现**：分数反而很低，例如第一句只有约 `0.0605`。

- **暴露的问题**：

  > SelfCheckGPT 检测的是跨采样一致性，不是真实性。

- **为什么典型**：
   主回答错，samples 也一起稳定地错，所以没有 contradiction。

- **论文里可以归纳为**：
   **Stable Hallucination Failure Mode**

  > A hallucination that is consistently reproduced across samples may receive a low SelfCheck score and evade detection.

**Q03：Python —— 正确事实却高分 / false positive**

- **问题设计**：完全真实、常见的 Python 历史。

- **模型表现**：主回答中的 Guido van Rossum、CWI、1991、Python 1.0 等内容基本正确。

- **SelfCheck 表现**：某些正确句子分数非常高，例如一条达到约 `0.8231`。

- **暴露的问题**：

  > NLI 模型会把“sample 没完整覆盖主句中的所有细节”误当成较高 contradiction。

- **根源**：
   一个句子塞了多个 atomic facts，而不同 sample 只覆盖其中一部分。

- **论文里可以归纳为**：
   **Multi-Claim / Semantic Coverage False Positive**

  > Factual sentences containing multiple atomic claims may receive high contradiction scores when sampled passages only partially cover the same information.

**Q01：Alan Turing 最后一条 —— 多事实句导致误报**

- **问题设计**：标准真实人物问题。

- **典型句子**：
   “1952 被起诉、1954 去世、2013 被赦免”被写进同一句。

- **SelfCheck 表现**：这句基本事实正确，但分数约 `0.7448`。

- **暴露的问题**：

  > sentence 不是理想的 factuality unit。

- **为什么典型**：
   sample 可能提“1952 + 1954”，却不提“2013 pardon”；另一些只提 pardon。于是整句被判不一致。

- **论文里可以归纳为**：
   **Granularity Mismatch**

  > Sentence-level scoring conflates multiple independent factual claims and may over-penalize partially supported sentences.

上述实验说明，`SelfCheckGPT-NLI` 在 `DeepSeek V4-Flash` 上能够正常工作，即当模型生成内容跨采样明显不一致的时候，会给出较高的幻觉分数。但进一步的批量实验也表明，`SelfCheckGPT` 检测到的本质并不是“事实错误”，而是“不同采样之间的不一致性”。它的有效性依赖于一个隐含假设：**当模型产生幻觉时，不同采样结果会出现明显差异或矛盾。**但这一假设并不总是成立。

### 研究启发

从上述实验得到两个启发思想：

1. 一致性并不等价于真实性。模型可能由于训练数据中的强关联、模式记忆或高置信度错误知识，对同一个错误事实持续生成高度相似的内容。这种情况下，仅依赖跨采样一致性的方法存在天然盲区。需要找到某种能够判别高一致但是虚假事实的模型内在因素。
2. Sentence 可能不是理想的幻觉检测单位。当前方法基于 Sentence 作为幻觉检测单位，但是一个自然语言句子往往包含多个独立事实，这就导致如果不同的采样中恰好覆盖了不同的事实，比如主回答是事实 A、Sample1 是事实 B、Sample2 是事实 C，那么当前的`SelfCheckGPT-NLI`将给出较高的幻觉分数——False Positive。可以尝试对 Sentence 进行颗粒度更细的拆分，再检验一致性。
