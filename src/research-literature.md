# 研究文献

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/research-literature.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/research-literature.md)ください。

以下の研究論文は、Flix の特定の側面を扱っています。これらは研究者向けに書かれており、必ずしも一般の読者にとって読みやすいものではありません。これらの論文の多くが発表されて以降、Flix は大きく進化しているため、現在の言語の正確な参照先としてはこのドキュメントを参照してください。

## 言語デザイン

- **[The Principles of the Flix Programming Language](https://dl.acm.org/doi/10.1145/3563835.3567661)**\
  *Magnus Madsen @ Onward! 2022*

## 型とエフェクト

- **[Qualified Types with Boolean Algebras](https://dl.acm.org/doi/10.1145/3763096)**\
  *Edward Lee, Jonathan Lindegaard Starup, Ondřej Lhoták, Magnus Madsen @ OOPSLA 2025*
- **[Associated Effects: Flexible Abstractions for Effectful Programming](https://dl.acm.org/doi/10.1145/3656393)**\
  *Matthew Lutze, Magnus Madsen @ PLDI 2024*（[関連エフェクト](./associated-effects.md)を参照）
- **[Fast and Efficient Boolean Unification for Hindley-Milner-Style Type and Effect Systems](https://dl.acm.org/doi/10.1145/3622816)**\
  *Magnus Madsen, Jaco van de Pol, Troels Henriksen @ OOPSLA 2023*
- **[With or Without You: Programming with Effect Exclusion](https://dl.acm.org/doi/10.1145/3607846)**\
  *Matthew Lutze, Magnus Madsen, Philipp Schuster, Jonathan Immanuel Brachthäuser @ ICFP 2023*
- **[Programming with Purity Reflection: Peaceful Coexistence of Effects, Laziness, and Parallelism](https://doi.org/10.4230/LIPIcs.ECOOP.2023.18)**\
  *Magnus Madsen, Jaco van de Pol @ ECOOP 2023* — Distinguished Paper Award（[純粋性リフレクション](./purity-reflection.md)を参照）
- **[Restrictable Variants: A Simple and Practical Alternative to Extensible Variants](https://doi.org/10.4230/LIPIcs.ECOOP.2023.17)**\
  *Magnus Madsen, Jonathan Lindegaard Starup, Matthew Lutze @ ECOOP 2023*
- **[Relational Nullable Types with Boolean Unification](https://dl.acm.org/doi/10.1145/3485487)**\
  *Magnus Madsen, Jaco van de Pol @ OOPSLA 2021*
- **[Polymorphic Types and Effects with Boolean Unification](https://dl.acm.org/doi/10.1145/3428222)**\
  *Magnus Madsen, Jaco van de Pol @ OOPSLA 2020*

## Datalog と不動点

- **[Flix: A Design for Language-Integrated Datalog](https://dl.acm.org/doi/10.1145/3763126)**\
  *Magnus Madsen, Ondřej Lhoták @ OOPSLA 2025* — Distinguished Artifact（[不動点](./fixpoints.md)を参照）
- **[Breaking the Negative Cycle: Exploring the Design Space of Stratification for First-Class Datalog Constraints](https://doi.org/10.4230/LIPIcs.ECOOP.2023.31)**\
  *Jonathan Lindegaard Starup, Magnus Madsen, Ondřej Lhoták @ ECOOP 2023*（[層化否定](./stratified-negation.md)を参照）
- **[Flix: A Meta Programming Language for Datalog](https://ceur-ws.org/Vol-3203/short8.pdf)**\
  *Magnus Madsen, Jonathan Lindegaard Starup, Ondřej Lhoták @ Datalog 2.0 2022*
- **[Fixpoints for the Masses: Programming with First-Class Datalog Constraints](https://dl.acm.org/doi/10.1145/3428193)**\
  *Magnus Madsen, Ondřej Lhoták @ OOPSLA 2020*
- **[Implicit Parameters for Logic Programming](https://dl.acm.org/doi/10.1145/3236950.3236953)**\
  *Magnus Madsen, Ondřej Lhoták @ PPDP 2018*
- **[Safe and Sound Program Analysis with Flix](https://dl.acm.org/doi/10.1145/3213846.3213847)**\
  *Magnus Madsen, Ondřej Lhoták @ ISSTA 2018*
- **[From Datalog to Flix: A Declarative Language for Fixed Points on Lattices](https://dl.acm.org/doi/10.1145/2908080.2908096)**\
  *Magnus Madsen, Ming-Ho Yee, Ondřej Lhoták @ PLDI 2016*（[束意味論](./lattice-semantics.md)を参照）
- **[Programming a Dataflow Analysis in Flix](https://staticanalysis.org/tapas2016/abstracts/TAPAS_2016_MadsenEtAl.pdf)**\
  *Magnus Madsen, Ming-Ho Yee, Ondřej Lhoták @ TAPAS 2016*

## コンパイル

- **[Overloading the Dot](https://dl.acm.org/doi/10.1145/3708493.3712684)**\
  *Joseph Tan, Magnus Madsen @ CC 2025*
- **[Tail Call Elimination and Data Representation for Functional Languages on the Java Virtual Machine](https://dl.acm.org/doi/10.1145/3178372.3179499)**\
  *Magnus Madsen, Ramin Zarifi, Ondřej Lhoták @ CC 2018*

<!--
# Research Literature

The following research papers cover specific aspects of Flix. They are written
for a research audience and not necessarily accessible to the general reader.
Flix has evolved significantly since many of these papers were published; the
book is the authoritative reference for the current language.

## Language Design

- **[The Principles of the Flix Programming Language](https://dl.acm.org/doi/10.1145/3563835.3567661)**\
  *Magnus Madsen @ Onward! 2022*

## Types and Effects

- **[Qualified Types with Boolean Algebras](https://dl.acm.org/doi/10.1145/3763096)**\
  *Edward Lee, Jonathan Lindegaard Starup, Ondřej Lhoták, Magnus Madsen @ OOPSLA 2025*
- **[Associated Effects: Flexible Abstractions for Effectful Programming](https://dl.acm.org/doi/10.1145/3656393)**\
  *Matthew Lutze, Magnus Madsen @ PLDI 2024* (see [Associated Effects](./associated-effects.md))
- **[Fast and Efficient Boolean Unification for Hindley-Milner-Style Type and Effect Systems](https://dl.acm.org/doi/10.1145/3622816)**\
  *Magnus Madsen, Jaco van de Pol, Troels Henriksen @ OOPSLA 2023*
- **[With or Without You: Programming with Effect Exclusion](https://dl.acm.org/doi/10.1145/3607846)**\
  *Matthew Lutze, Magnus Madsen, Philipp Schuster, Jonathan Immanuel Brachthäuser @ ICFP 2023*
- **[Programming with Purity Reflection: Peaceful Coexistence of Effects, Laziness, and Parallelism](https://doi.org/10.4230/LIPIcs.ECOOP.2023.18)**\
  *Magnus Madsen, Jaco van de Pol @ ECOOP 2023* — Distinguished Paper Award (see [Purity Reflection](./purity-reflection.md))
- **[Restrictable Variants: A Simple and Practical Alternative to Extensible Variants](https://doi.org/10.4230/LIPIcs.ECOOP.2023.17)**\
  *Magnus Madsen, Jonathan Lindegaard Starup, Matthew Lutze @ ECOOP 2023*
- **[Relational Nullable Types with Boolean Unification](https://dl.acm.org/doi/10.1145/3485487)**\
  *Magnus Madsen, Jaco van de Pol @ OOPSLA 2021*
- **[Polymorphic Types and Effects with Boolean Unification](https://dl.acm.org/doi/10.1145/3428222)**\
  *Magnus Madsen, Jaco van de Pol @ OOPSLA 2020*

## Datalog and Fixpoints

- **[Flix: A Design for Language-Integrated Datalog](https://dl.acm.org/doi/10.1145/3763126)**\
  *Magnus Madsen, Ondřej Lhoták @ OOPSLA 2025* — Distinguished Artifact (see [Fixpoints](./fixpoints.md))
- **[Breaking the Negative Cycle: Exploring the Design Space of Stratification for First-Class Datalog Constraints](https://doi.org/10.4230/LIPIcs.ECOOP.2023.31)**\
  *Jonathan Lindegaard Starup, Magnus Madsen, Ondřej Lhoták @ ECOOP 2023* (see [Stratified Negation](./stratified-negation.md))
- **[Flix: A Meta Programming Language for Datalog](https://ceur-ws.org/Vol-3203/short8.pdf)**\
  *Magnus Madsen, Jonathan Lindegaard Starup, Ondřej Lhoták @ Datalog 2.0 2022*
- **[Fixpoints for the Masses: Programming with First-Class Datalog Constraints](https://dl.acm.org/doi/10.1145/3428193)**\
  *Magnus Madsen, Ondřej Lhoták @ OOPSLA 2020*
- **[Implicit Parameters for Logic Programming](https://dl.acm.org/doi/10.1145/3236950.3236953)**\
  *Magnus Madsen, Ondřej Lhoták @ PPDP 2018*
- **[Safe and Sound Program Analysis with Flix](https://dl.acm.org/doi/10.1145/3213846.3213847)**\
  *Magnus Madsen, Ondřej Lhoták @ ISSTA 2018*
- **[From Datalog to Flix: A Declarative Language for Fixed Points on Lattices](https://dl.acm.org/doi/10.1145/2908080.2908096)**\
  *Magnus Madsen, Ming-Ho Yee, Ondřej Lhoták @ PLDI 2016* (see [Lattice Semantics](./lattice-semantics.md))
- **[Programming a Dataflow Analysis in Flix](https://staticanalysis.org/tapas2016/abstracts/TAPAS_2016_MadsenEtAl.pdf)**\
  *Magnus Madsen, Ming-Ho Yee, Ondřej Lhoták @ TAPAS 2016*

## Compilation

- **[Overloading the Dot](https://dl.acm.org/doi/10.1145/3708493.3712684)**\
  *Joseph Tan, Magnus Madsen @ CC 2025*
- **[Tail Call Elimination and Data Representation for Functional Languages on the Java Virtual Machine](https://dl.acm.org/doi/10.1145/3178372.3179499)**\
  *Magnus Madsen, Ramin Zarifi, Ondřej Lhoták @ CC 2018*
-->
