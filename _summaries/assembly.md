---
title: "Programming from the ground up"
author: "Me"
date: "Fbruary 17, 2020"
output: html_document
---

<style type="text/css">

body{ /* Normal  */
      font-size: 12px;
  }
td {  /* Table  */
  font-size: 12px;
}
h1.title {
  font-size: 38px;
  color: DarkRed;
}
h1 { /* Header 1 */
  font-size: 28px;
  color: DarkRed;
}
h2 { /* Header 2 */
    font-size: 22px;
  color: DarkRed;
}
h3 { /* Header 3 */
  font-size: 18px;
  font-family: "Times New Roman", Times, serif;
  color: DarkRed;
}
code.r{ /* Code block */
    font-size: 12px;
}
pre { /* Code block - determines code spacing between lines */
    font-size: 14px;
}
</style>


# Programming from the ground up
{:.no_toc}

This is a resumee of the book “Programming from the ground up”. For this resumee, I asked myself different questions and used
the book as well as other resources to answer them.

The chapters are the following :

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# Introduction

What is assembly language ?
{:style="color:DarkRed; font-size: 150%;"}
There are 3 kinds of languages :
- Machine language : what the computer sees and uses (sequence of numbers) → Specific to the processor.
- Assembly language : same as machine language (also specific to the processor), but the numbers are replaced by letters. Easier to use for humans
- High-level language : closer to natural language. The purpose is to make programming easier. Not specific to the computer (portable across multiple systems).

The syntax of assembly instructions is the following:\\
Operation destination, source\\
Example :

| Assembly | Machine language (Intel) |
| ---: | ---:|
| mov epb, esp | 89 e5 |
| sub esp, 0x8 | 83 ec 08 |
{:.table-striped}
{:style="font-size: 150%;"}
		
→ This example transfers the value of esp into ebp, and subtracts 8 (0x8) to esp. The result is stored into esp. We will soon see what esp and ebp are.

Assembly language is converted into executable machine code by a utility program referred to as an assembler.
High-level languages are converted to machine language by a compiler.

Why learn assembly ?
Assembly language is very low level and close to the processor → assembly is great for speed optimization.
Also, understanding assembly language allows to fully understand what a program does and therefore is very useful for reverse engineering tasks.


# Computer Architecture

# Your First Programs

# All about functions

# Dealing with Files

# Reading and Writing Simple Records

# Developing Robust Programs

# Sharring Functions with Code Libraries

# Intermediate Memory Topics

# Counting Like a Computer

# High-Level Languages

# Optimization

# Moving On from Here
