+++
date = '2026-07-17T14:30:20+08:00'
draft = 'false'
title = 'ENM'
tags=['Metric', 'SpreadSheet', 'Predict', 'Fault']
categories = ['SpreadSheet']

+++





[TOC]





[总体解构](#overview) / [实验细节解构](#exp-details) / [AI中文讲解](#ai-zh) / [AI英文讲解](#ai-en)

| ITEM    | METADATA                                                     |
| ------- | ------------------------------------------------------------ |
| title   | Attention Is All You Need                                    |
| date    | 2026-07-17                                                   |
| venue   | NeurIPS                                                      |
| authors | ["Ashish Vaswani", "Noam Shazeer", "Niki Parmar", "Jakob Uszkoreit"] |
| year    | 2017                                                         |
| pdf     | https://arxiv.org/abs/1706.03762                             |
| code    | https://github.com/tensorflow/tensor2tensor                  |
| tags    | ["Transformer", "NLP", "Foundation Model"]                   |



## 总体解构 {#overview}

Transformer 的核心贡献是用自注意力替代 RNN/CNN 的序列建模主干，显著提升并行效率，并在机器翻译上达到 SOTA。  
这篇论文要解决的关键矛盾是：**长距离依赖建模能力 vs 训练并行效率**。  
作者给出的答案是：通过 Multi-Head Self-Attention 在全局范围直接建立 token 间关系，并用位置编码补充顺序信息。

---

## 实验细节解构 {#exp-details}

1. 数据集：WMT 2014 English-German、English-French。  
2. 对比基线：当时主流的 RNN/CNN 神经机器翻译模型。  
3. 评价指标：BLEU。  
4. 结果亮点：在更少训练成本下获得更高翻译质量。  
5. 消融重点：多头注意力头数、模型维度、残差与层归一化组合等对性能影响显著。  
6. 可复现关注点：学习率 warmup 策略、label smoothing、batch token 数设定。

---

## AI中文讲解 {#ai-zh}

你可以把 Transformer 理解为一个“全班同学同时互相看笔记”的机制。  
RNN 像是按座位一个个传纸条，信息传远了容易丢；  
Self-Attention 是每个词都能直接看所有词的“相关性分数”，  
所以既能抓全局依赖，又适合 GPU 并行。

---

## AI英文讲解 {#ai-en}

The Transformer replaces recurrence with self-attention, enabling fully parallel training across tokens.  
Its key idea is that each token dynamically weighs all other tokens, producing context-aware representations.  
Multi-head attention allows the model to capture different relation patterns simultaneously, while positional encodings preserve order information.





<script> document.addEventListener('DOMContentLoaded', function() {     const content = document.querySelector('.post-content');     if (!content) return;     const headings = content.querySelectorAll('h2, h3');     if (headings.length === 0) return;      
                                                                   const toc = document.createElement('aside');     toc.id = 'dynamic-toc';     toc.innerHTML = '<div class="toc-title">目录</div><ul></ul>';     const ul = toc.querySelector('ul');      headings.forEach((heading, index) => {         if (!heading.id) heading.id = 'toc-head-' + index;         const li = document.createElement('li');         const a = document.createElement('a');         a.href = '#' + heading.id;         a.textContent = heading.textContent;         li.appendChild(a);         ul.appendChild(li);     });     document.body.appendChild(toc);      
                                                                   const style = document.createElement('style');     style.textContent = `         #dynamic-toc { position: fixed; top: 50px; right: 30px; width: 240px; max-height: 70vh; overflow-y: auto; padding: 15px; background: rgba(45, 55, 72, 0.95); border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.3); font-size: 0.9rem; z-index: 100; font-family: sans-serif; }         #dynamic-toc .toc-title { font-weight: bold; margin-bottom: 10px; color: #00bcd4; border-bottom: 1px solid #4a5568; padding-bottom: 5px; }         #dynamic-toc ul { list-style: none; padding-left: 0; margin: 0; }         #dynamic-toc li { margin: 6px 0; }         #dynamic-toc a { color: #a0aec0; text-decoration: none; display: block; transition: color 0.2s; line-height: 1.4; }         #dynamic-toc a:hover { color: #fff; }         @media screen and (max-width: 1200px) { #dynamic-toc { display: none; } }     `;     document.head.appendChild(style); }); </script>