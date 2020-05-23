# Latex基本使用教程

- [入门](#%e5%85%a5%e9%97%a8)
  - [开始](#%e5%bc%80%e5%a7%8b)
  - [条目编号](#%e6%9d%a1%e7%9b%ae%e7%bc%96%e5%8f%b7)
  - [对齐](#%e5%af%b9%e9%bd%90)
    - [行对齐](#%e8%a1%8c%e5%af%b9%e9%bd%90)
    - [段对齐](#%e6%ae%b5%e5%af%b9%e9%bd%90)
  - [字体](#%e5%ad%97%e4%bd%93)
    - [字体大小](#%e5%ad%97%e4%bd%93%e5%a4%a7%e5%b0%8f)
    - [字体变换](#%e5%ad%97%e4%bd%93%e5%8f%98%e6%8d%a2)
- [为Paper准备](#%e4%b8%bapaper%e5%87%86%e5%a4%87)
  - [添加参考文献](#%e6%b7%bb%e5%8a%a0%e5%8f%82%e8%80%83%e6%96%87%e7%8c%ae)
    - [直接插进文档](#%e7%9b%b4%e6%8e%a5%e6%8f%92%e8%bf%9b%e6%96%87%e6%a1%a3)
    - [使用BibTeX](#%e4%bd%bf%e7%94%a8bibtex)
- [Reference](#reference)
在测试一下
# 入门
一个基本样式
```
\documentclass{article}
\usepackage{ctex}
\CTEXoptions[today=old]
\title{Welcome to learn \LaTeX}
\author{anonymous \and another anonymous}
\date{An unknown day}
\begin{document}
\maketitle
\tableofcontents
\part{1}
\section{A}
\section{B}
\subsection{a}
\part{2}
\section{C}
\subsection{b}
\part{3}
\end{document} 
```
## 开始
```
\documentclass[a4paper]{article}
```

## 条目编号
```
\begin{itemize}
\item[*] a
\item[*] b
\end{itemize}

\begin{itemize}[step i]
\item a
\item b
\end{itemize}
%% step i a,step ii b
```

## 对齐
### 行对齐
```
\leftline{}
\centerline{}
\rightline{}
```
### 段对齐
```
\begin{flushleft}...\end{flushleft}
\begin{center}...\end{center}
\begin{flushright}...\end{flushright}
```

## 字体
### 字体大小
- 全局大小
  ```
  \documentclass[12pt]{article}
  ```
- 局部
  ```
  \tiny
  \scriptsize
  \footnotesize
  \small
  \normalsize
  \large
  \Large
  \LARGE
  \huge
  \Huge
  ```
### 字体变换
```
%% 加粗
\textbf{}
%% 下划线
\underline{}
%% 斜体
\emph{}
```

# 为Paper准备
## 添加参考文献
### 直接插进文档
```
\begin{thebibliography}{99}  
\bibitem{ref1}Zheng L, Wang S, Tian L, et al., Query-adaptive late fusion for image search and person re-identification, Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, 2015: 1741-1750.  
\bibitem{ref2}Arandjelović R, Zisserman A, Three things everyone should know to improve object retrieval, Computer Vision and Pattern Recognition (CVPR), 2012 IEEE Conference on, IEEE, 2012: 2911-2918.  
\bibitem{ref3}Lowe D G. Distinctive image features from scale-invariant keypoints, International journal of computer vision, 2004, 60(2): 91-110.  
\bibitem{ref4}Philbin J, Chum O, Isard M, et al. Lost in quantization: Improving particular object retrieval in large scale image databases, Computer Vision and Pattern Recognition, 2008. CVPR 2008, IEEE Conference on, IEEE, 2008: 1-8.  
\end{thebibliography}
```
在正文中使用  
```
\cite{ref1}
\cite{ref1, ref5}
```
进行引用  

### 使用BibTeX
- 常用的引用方式，大部分模板皆用,后缀名 '.bib'
```
@online{gameCost,
	key = {How Much Does It Cost To Make A Big Video Game?},
  	title = {How Much Does It Cost To Make A Big Video Game?},
  	howpublished = {\url{http://kotaku.com/how-much-does-it-cost-to-make-a-big-video-game-1501413649}},
  	note = {[Accessed: 2015-01-20]}
}
%% 格式如下
@article{name1,
author = {作者, 多个作者用 and 连接},
title = {标题},
journal = {期刊名},
volume = {卷20},
number = {页码},
year = {年份},
abstract = {摘要, 这个主要是引用的时候自己参考的, 这一行不是必须的}
}
@book{name2,
author ="作者",
year="年份2008",
title="书名",
publisher ="出版社名称"
}
```
- 说明：
  - @article文章分类
  - name1用于索引

- 在latex中的使用
```
\cite{name1}
```
- 如何获取引用信息呢?
  - 在所有期刊或会议的网站上，都会有cite this一类的标识
  - 选择bibtex，直接复制即可
  



# Reference
- [LaTeX笔记|基本功能](https://zhuanlan.zhihu.com/p/24394912)
- [latex里设置居中左对齐](https://blog.csdn.net/erwangshi/article/details/23022887)
- [Latex技巧：插入参考文献](https://www.cnblogs.com/yifdu25/p/8330652.html)
