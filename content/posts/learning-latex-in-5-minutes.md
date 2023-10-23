+++
title = 'Learning Latex in 5 Minutes'
date = 2023-10-17T16:43:29+01:00
draft = false
+++

LaTeX is a powerful typesetting system commonly used for creating documents with complex formatting, such as research papers, theses, articles, and CVs. In LaTeX, you work with a set of notations and commands to define the structure and formatting of your document. Let's explore some essential LaTeX notations and provide examples to help you get started.

## LaTeX Notations

### 1. Document Class

In LaTeX, you start by specifying the type of document you want to create using the \documentclass command. For example, to create an article, you would use:

```latex
\documentclass{article}
```

Other common document classes include book, report, and letter.

### 2. Sections and Headings

You can organize your document into sections and subsections using commands like \section and \subsection:

```latex
\section{Introduction}
This is the introduction section.

\subsection{Background}
This is a subsection under the introduction.
```

### 3. Text Formatting

LaTeX provides various commands to format text. For instance, to make text bold, you can use \textbf:

```latex
This is \textbf{bold text}.
```

### 4. Lists

You can create ordered and unordered lists using the itemize and enumerate environments:

```latex
\begin{itemize}
    \item Item 1
    \item Item 2
\end{itemize}

\begin{enumerate}
    \item First item
    \item Second item
\end{enumerate}
```

### 5. Mathematical Equations

LaTeX is renowned for its mathematical typesetting capabilities. To include an equation, use the \begin{equation} environment:

```latex
\begin{equation}
    E=mc^2
\end{equation}
```

### 6. Citations and References

For academic writing, LaTeX allows you to manage citations and references with tools like BibTeX. You can use the \cite command and a BibTeX file:

```latex
According to \cite{einstein},...

\bibliographystyle{plain}
\bibliography{references.bib}

```

## LaTeX Example Snippets

Here are a few practical LaTeX examples:

### Creating a Simple Table

```latex
\begin{tabular}{|c|c|c|}
    \hline
    Item & Quantity & Price (\$) \\
    \hline
    Item 1 & 5 & 10 \\
    Item 2 & 3 & 8 \\
    \hline
\end{tabular}
```

### Inserting an Image

```latex
\usepackage{graphicx}

\begin{figure}
    \centering
    \includegraphics[width=0.7\textwidth]{image.jpg}
    \caption{A beautiful image.}
\end{figure}
```

### Creating a List of References

```latex
\begin{thebibliography}{9}
    \bibitem{einstein} A. Einstein, "On the electrodynamics of moving bodies," Annalen der Physik, vol. 17, no. 10, pp. 891-921, 1905.
\end{thebibliography}
```

## Useful LaTeX Learning Resources

If you want to explore LaTeX further, here are some external links to help you get started:

- [LaTeX Project Official Site](https://www.latex-project.org/): The official source for LaTeX information and documentation.
- [Overleaf](https://www.overleaf.com/): A popular online LaTeX editor with many templates and guides.
- [ShareLaTeX](): An online LaTeX editor and collaboration platform, now part of Overleaf.
- [The Not So Short Introduction to LaTeX](https://tobi.oetiker.ch/lshort/lshort.pdf): A comprehensive and beginner-friendly guide to LaTeX.
- [LaTeX Wikibook](https://en.wikibooks.org/wiki/LaTeX): An extensive collection of LaTeX tutorials and guides.
- [CTAN](https://www.ctan.org/) - Comprehensive TeX Archive Network: A repository of LaTeX packages, tools, and documentation.

LaTeX offers a rich set of notations and commands, making it a versatile tool for document creation. With these examples and external resources, you'll be well-equipped to create beautifully formatted documents and explore LaTeX further.
