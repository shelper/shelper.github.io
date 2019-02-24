<!--
.. title: setup emacs foc C++ with ccls and lsp
.. slug: setup-emacs-foc-c++-with-ccls-and-lsp
.. date: 2019-02-24 03:55:52 UTC
.. tags:
.. category: programming, c++, emacs
.. link:
.. description:
.. type: text
-->

So `ccls` is a lsp wrapper for clang
it works as a backend that indexes  source code and gives emacs index information for better navigation and refactoring of c++

I have had a few issues with the tool's setup in emacs enviroment.
and found the following things that may need extra attention

1. `.ccls` file, this file basically does two things
  - tell `ccls` how to index the file using `clang`
  - before index the file tell `clang` how to compile and interpret the code
as there is a mixture of options for `ccls` and arguments for `clang`, need to pay extra attention

usually lines in `.ccls` file start with `%` is for `ccls`, and lines start with `-` is for `clang`

2. c and c++ code/mode, it is better to set only for c++ as c++ is a super set of c

3. company completion of header files should work without input the surrounds of header file e.g, `""` or `<>`
4. some times ccls complains it cannot find header file, so you may include the root path of the header files by adding `-IincludePath` to `.ccls` files
5. for large projects using `compile_commands.json` to guide `ccls`indexing. you may of course set partial folder for index by setting the black list ow white list if indexing is way too slow

6. setting key bindings in emacs, the priority is: `local-settings > global-settings > define-key`
