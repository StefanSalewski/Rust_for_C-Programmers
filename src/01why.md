# Chapter 1: Rust for C-Programmers

**A Compact Introduction to the Rust Programming Language**

Preprint, created in 2024, 2025

(c) 2025 S. Salewski

All rights reserved.

Rust is a modern systems programming language built for safety, performance, and efficient concurrency. As a compiled language, it produces optimized, standalone machine code, making it an excellent choice for low-level development. Rust enforces strict static typing, preventing many common programming errors at compile time while maintaining high execution efficiency.  

With its unique ownership model, Rust guarantees memory safety without relying on a runtime garbage collector. This approach eliminates data races and prevents undefined behavior while preserving performance. Rust’s zero-cost abstractions enable developers to write concise, expressive code without sacrificing efficiency. As an open-source project licensed under the MIT and Apache 2.0 licenses, Rust benefits from a strong, community-driven development process.  

Rust’s growing popularity stems from its versatility, spanning applications such as operating systems, embedded systems, WebAssembly, networking, GUI development, and mobile platforms. It supports all major operating systems, including Windows, Linux, macOS, Android, and iOS. With active maintenance and continuous evolution, Rust remains a compelling choice for modern software development.  

This book offers a compact yet thorough introduction to Rust, intended for readers with experience in systems programming. Those new to programming may find it helpful to begin with an introductory resource, such as the official Rust guide, ["The Book"](https://doc.rust-lang.org/book/), or explore a simpler language before diving into Rust.

---

## 1.1 Why Rust?

Rust is a modern programming language that uniquely combines high performance with safety. Although concepts like **ownership** and **borrowing** can initially seem challenging, they enable developers to write efficient and reliable code. Rust's syntax may appear unconventional to those accustomed to other languages, yet it offers powerful abstractions that make it easier to create robust software.

So why has Rust gained popularity despite its complexities?

Rust aims to balance the performance benefits of low-level systems programming languages with the safety, reliability, and user-friendliness of high-level languages. Low-level languages like C and C++ offer high performance with minimal resource usage but can be prone to errors that affect reliability. High-level languages such as Python, Kotlin, Julia, JavaScript, C#, and Java are easier to learn and use but often rely on garbage collection and large runtime environments, making them less suitable for certain systems programming tasks.

Languages like Rust, Go, Swift, Zig, Nim, Crystal, and V seek to bridge this gap. Rust has been particularly successful, as shown by its growing popularity.

As a systems programming language, Rust enforces memory safety through its ownership model and borrow checker, preventing issues such as null pointer dereferencing, use-after-free, and buffer overflows—without using a garbage collector. Rust avoids hidden, expensive operations like implicit type conversions or unnecessary heap allocations, giving developers precise control over performance. Copying large data structures is typically avoided by using references or move semantics to transfer ownership. When copying is necessary, developers must explicitly request it with functions like `clone()`. Despite these performance-focused constraints, Rust provides conveniences such as **iterators** and **closures**, offering a user-friendly experience while retaining high efficiency.

Rust's ownership model also guarantees **fearless concurrency** by preventing data races at compile time, simplifying the creation of concurrent programs compared to languages that detect such errors at runtime—or not at all.

Although Rust does not use a traditional class-based object-oriented programming (OOP) approach, it incorporates OOP concepts via **traits** and **structs**, supporting polymorphism and code reuse in a flexible manner. Rust replaces **exceptions** with `Result` and `Option` types for error handling, encouraging explicit handling and avoiding unexpected runtime failures.

Rust’s development started in 2006 with Graydon Hoare, initially with volunteer help and later sponsored by Mozilla. The first stable version, Rust 1.0, was released in 2015. By version 1.84 and the Rust 2024 edition, stabilized in early 2025, Rust had continued to evolve while maintaining backward compatibility. Today, Rust benefits from a large, active developer community. After Mozilla scaled back its involvement, the Rust community formed the **Rust Foundation**, supported by AWS, Google, Microsoft, Huawei, and others, to ensure continued growth and sustainability. Rust is free, open-source software licensed under permissive MIT and Apache 2.0 terms for its tools, standard library, and most external packages.

Rust’s community-driven development process relies on **RFCs** (Requests for Comments) to propose and discuss new features. This open, collaborative approach has fueled Rust’s rapid growth and created a rich ecosystem of libraries and tools. The community’s emphasis on quality and collaboration has turned Rust from just a programming language into a movement advocating safer, more efficient development.

Well-known companies such as Facebook, Dropbox, Amazon, and Discord use Rust for various projects. Dropbox, for example, uses Rust to optimize file storage, while Discord relies on it for high-performance networking. Rust is also used in **system programming**, **embedded systems**, **WebAssembly** development, and building applications for **PCs** (Windows, Linux, macOS) and **mobile platforms**. Another milestone is Rust’s integration into the Linux kernel—the first time an additional language has been adopted alongside C. Rust is also gaining momentum in the **blockchain** industry.

Rust’s ecosystem is mature and well-supported. It features a powerful compiler, the modern Cargo build system, and **Crates.io**, an extensive repository of open-source libraries. Tools like `rustfmt` for formatting and `clippy` for linting help keep Rust code clean and consistent. Modern GUI frameworks such as **EGUI** and **Xilem**, game engines like **Bevy**, and entire operating systems such as **Redox-OS** are available in the Rust ecosystem.

As a statically typed, compiled language, Rust historically might not have been a top choice for rapid prototyping, where dynamically typed, interpreted languages (e.g., Python or JavaScript) have often excelled. However, Rust’s increasingly faster compile times—improved by incremental compilation and build artifact caching—together with its robust type system and strong IDE support, have made prototyping in Rust more efficient. Many developers now select Rust for its performance, safety guarantees, and straightforward path from prototypes to production-ready applications.

Since this book assumes familiarity with Rust’s main features, we will not analyze Rust's pros and cons further. Instead, we focus on its core features and its established ecosystem. The LLVM-based compiler (`rustc`), the Cargo package manager, Crates.io, and Rust’s large community are essential factors behind its growing importance.

---

## 1.2 What Makes Rust Special?

Rust stands out by offering **automatic memory management without a garbage collector**. It achieves this through strict rules around **ownership**, **borrowing**, **move semantics**, and by making **immutability** the default unless a variable is declared mutable with `mut`. Rust’s memory model ensures excellent performance while preventing issues like invalid memory access or data races. Its **zero-cost abstractions** enable high-level features without sacrificing performance. Although this system demands more attention from developers, the long-term benefits—better performance and fewer memory errors—are particularly valuable in large projects.

Here are some of the key features that set Rust apart:

### 1.2.1 Error Handling Without Exceptions

Rust does not use traditional exception handling (`try/catch`). Instead, it employs **`Result`** and **`Option`** types for error handling, requiring developers to address errors explicitly. This design prevents silently ignored failures, a common issue with exceptions that may remain undiscovered during development and later cause unexpected crashes in production. While explicit error handling can lead to more verbose code, the **`?` operator** offers a concise way to propagate errors, preserving readability. Rust’s error-handling strategy encourages predictable, transparent code.

### 1.2.2 A Different Approach to Object-Oriented Programming

Rust uses object-oriented concepts like encapsulation and polymorphism but does not rely on classical inheritance. Instead, Rust focuses on **composition** and uses **traits** to define shared behaviors and interfaces, resulting in flexible, reusable code. Through **trait objects**, Rust supports dynamic dispatch, providing polymorphism comparable to OOP in other languages. This design encourages clear, modular code while avoiding many of the complications associated with inheritance. For developers familiar with Java or C++, Rust’s traits serve as a modern, efficient alternative to traditional interfaces and abstract classes.

### 1.2.3 Pattern Matching and Enumerations

Rust’s **enumerations** (enums) are more advanced than in many other languages. They are **algebraic data types**, able to store different types and amounts of data for each variant, making them well-suited for modeling complex data structures. When paired with **pattern matching**, Rust allows concise, expressive handling of various cases. Although pattern matching may seem unfamiliar initially, it greatly simplifies dealing with complex data and boosts readability.

### 1.2.4 Threading and Parallel Processing

Rust excels at **safe concurrency** and **parallelism**. Ownership and borrowing rules detect data races at compile time, making it easier to write efficient, safe concurrent programs. Rust’s **fearless concurrency** concept gives developers confidence when building multithreaded applications, since the compiler flags data race or synchronization errors before execution. Libraries like **Rayon** provide simple, high-level APIs for parallel processing, making Rust an appealing choice for performance-critical applications that require safe concurrency across multiple threads.

### 1.2.5 String Types and Explicit Conversions

Rust primarily uses two string types: **`String`** and **`&str`**. **`String`** represents an owned, heap-allocated string, while **`&str`** is a borrowed string slice, often used for string literals ('like this') or substrings. Although managing these types can initially be confusing, Rust’s strict typing clarifies when data is owned or borrowed, ensuring safe memory handling. Conversions between these types generally require explicit functions like `String::from()`, or traits like `From`, `Into`, or `AsRef`. While this approach can introduce some verbosity, it improves performance, clarity, and safety.

Rust also demands explicit **type conversions** for numeric types. For example, integers do not automatically convert to floating-point numbers, and vice versa. This strict approach helps avoid errors and prevents unwanted performance overhead from implicit conversions.

### 1.2.6 Trade-offs in Language Features

Rust does not include some convenience features found in other languages, such as **default parameters**, **named function parameters**, or **subrange types**. It also lacks **type or constant sections** like those in Pascal, which can make Rust code appear more verbose. However, developers often use **builder patterns** or **method chaining** to simulate default and named parameters, producing clear, maintainable code. The Rust community continues to discuss adding named parameters and other features in future releases.

---

## 1.3 About the Book

There are already several thorough Rust books, including the official guide, [The Book](https://doc.rust-lang.org/book/), and more detailed works such as *Programming Rust* by Jim Blandy, Jason Orendorff, and Leonora F. S. Tindall. For a deeper dive, *Rust for Rustaceans* by Jon Gjengset and the online resource [Effective Rust](https://www.lurklurk.org/effective-rust/) are also excellent.  
Amazon indeed lists many other Rust books, but their quality is not always evident. Some might be great, while others may contain trivial or unhelpful content—possibly generated by AI or copied from free online sources.  
Additional learning resources include [Rust by Example](https://doc.rust-lang.org/rust-by-example/) and the [Rust Cookbook](https://rust-lang-nursery.github.io/rust-cookbook/). There are also many video tutorials for visual learners.

With so much available, you might wonder why another Rust book is needed. Traditionally, creating a solid technical book requires deep expertise, strong writing skills, and substantial time—often more than a thousand hours. Professional editing and proofreading by established publishers have typically been necessary to remove errors, ensure clarity, and make the text enjoyable to read.

Some Rust books become verbose by over-explaining certain topics. Books focusing on Rust, written in concise technical English, are somewhat scarce. One reason is that Rust is a complex language with some unusual concepts. Authors often compensate by adding elaborate explanations, sometimes adopting a style aimed at younger readers rather than adults with existing programming knowledge. A more compact, focused book may therefore be worthwhile, though whether the effort is justified remains open to debate.

However, AI tools have greatly reduced the workload involved in writing such a book, especially over the last two years. Routine tasks like grammar and spelling checks—which can be challenging for non-native English speakers—are now handled reliably by AI. AI can also enhance writing style by breaking up overly long sentences, reducing verbosity, and removing repetitive material. Beyond these tasks, AI can generate initial chapter drafts, insert new content, reorganize sections, suggest examples, or remove redundancies. Although AI still cannot produce a complete book by itself—especially one covering a complex topic like Rust—an iterative, AI-assisted approach with careful human review can save a significant amount of time and effort.

One of the greatest benefits is in grammar correction, a tedious and mistake-prone step for authors who are not native English speakers.

This book started in September 2024 as an experiment to test whether AI assistance could make it possible to produce a high-quality Rust book without investing an entire year. The results were promising: total work was cut by about half. For native speakers with strong writing skills, the time savings might be slightly lower.

Some might ask why not wait a few more years for AI to reach the point of generating complete, high-quality, and even personalized books on its own. We believe that day will come relatively soon. However, with this book now near completion, the hundreds of hours invested have already proven their value.

This book primarily targets those with systems programming experience—people familiar with statically typed, compiled languages such as C, C++, D, Zig, Nim, Ada, Crystal, and others. It is not intended for absolute beginners, and those who have only used languages like Python might prefer the official Rust book or another resource geared toward that background.

Our aim is to present Rust’s fundamentals as succinctly as possible, avoiding repetition, lengthy discussions, and broad coverage of basic programming or hardware topics. We focus on core Rust features (excluding macros and async) in fewer than 500 pages, limiting deeper explorations or larger code examples. We believe extensive details are less vital today since Rust’s language specification, specialized online resources, and AI tools are readily available. Most readers do not need to memorize every minor feature they might rarely use.

The title *Rust for C-Programmers* reflects the book’s goal: providing an efficient introduction to Rust for experienced developers, especially those coming from a C background.

Structuring a book about a language as complex as Rust was challenging. We tried to showcase Rust’s most compelling and practical features early, while acknowledging the dependencies between various topics. It is generally best to read the chapters in order, but they are not so interconnected that out-of-order reading is impossible—though you might occasionally encounter references to earlier content.

---

When viewing the online version of this book (generated by the *mdbook* tool), you can select different themes from a drop-down menu and use the built-in search function. If the default font size is too small, most web browsers let you zoom in using 'CTRL +'. Code examples with hidden lines can be expanded by clicking on them, and you can often run the examples directly in the Rust Playground. You can also edit the examples before running them, or copy and paste them into the [Rust Playground](https://play.rust-lang.org).  
We recommend reading the online version in a web browser that supports permanent text highlighting, such as Firefox with the text-marker plugin.  
Most browsers also have the ability to save web pages for offline reading, and you can optionally create a PDF using *mdbook*. Other book formats like Epub or Mobi for e-readers are currently not supported.

It is unclear whether a printed version of this book will be published. Printed computer books quickly become outdated, and publishing and promotion costs may consume most of the revenue. On the other hand, releasing the book on Amazon might be an effective way to reach a broader audience.

---

## 1.4 About the Authors

The main author, Dr. S. Salewski, studied Physics, Mathematics, and Computer Science at the University of Hamburg (Germany), earning his Ph.D. in laser physics in 2005. His work includes fiber laser research, electronics, and software development in Pascal, Modula-2, Oberon, C, Ruby, Nim, and Rust. Some of his projects—such as Nim GTK GUI bindings, Nim implementations of an N-dimensional RTree, and a fully dynamic constrained Delaunay triangulation—are freely available at [https://github.com/StefanSalewski](https://github.com/StefanSalewski). This repository also hosts a Rust version of his tiny chess engine (with GTK, EGUI, and Bevy graphical interfaces), selected chapters of this book in Markdown, and materials for another book by the same author about the Nim programming language, published online in 2020.

Naturally, much of this book’s content draws from various Rust community resources, including numerous printed books, the official online Rust Book, Rust’s language specification and library documentation, Rust-by-example, the Cargo Book, the Rust Performance Book, and many others.

As noted earlier, this book was written with significant AI assistance. In modern technical publishing, avoiding AI altogether would be highly inefficient and likely result in a lower-quality product. Virtually all high-quality goods we use daily are produced with advanced tools, and excluding such tools from creating a programming book makes little sense.

Initially, we tried listing each AI tool used, but the list grew cumbersome. Today’s large language models contain substantial knowledge of Rust and can produce helpful draft texts, as well as perform grammar and style refinements. For final editing, we primarily relied on GPT-4 o1, which is particularly skilled at creating succinct paraphrases and sometimes discards lengthy sections of the author’s text if it deems them too verbose. By using a paid OpenAI subscription, we guided the AI toward a concise, neutral technical style in each final iteration, ensuring a coherent and consistent presentation.

---
