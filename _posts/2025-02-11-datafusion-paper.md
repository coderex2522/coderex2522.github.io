---
layout:     post
title:      "【论文笔记】Apache Arrow DataFusion: a Fast, Embeddable, Modular Analytic
Query Engine"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:
    - 论文笔记
---

论文笔记 《Apache Arrow DataFusion: a Fast, Embeddable, Modular Analytic
Query Engine》

## ABSTRACT
> Apache Arrow DataFusion[25] is a fast, embeddable, and extensible query engine written in Rust[76] that uses Apache Arrow[24] as its memory model. In this paper we describe the technologies on which it is built, and how it fits in long-term database implementation trends. We then enumerate its features, optimizations, architecture and extension APIs to illustrate the breadth of requirements of modern OLAP engines as well as the interfaces needed by systems built with them. Finally, we demonstrate open standards and extensible design do not preclude state-of-the-art performance using a series of experimental comparisons to DuckDB[66]. While the individual techniques used in DataFusion have been previously described many times, it differs from other industrial strength engines by providing competitive performance and an open architecture that can be customized using more than 10 major extension APIs. This flexibility has led to use in many commercial and open source databases, machine learning pipelines, and other data-intensive systems. We anticipate that the accessibility and versatility of DataFusion, along with its competitive performance, will further the proliferation of high-performance custom data infrastructures tailored to specific needs assembled from modular components[18, 61].