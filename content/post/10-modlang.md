+++
author = "Pranav Shridhar"
title = "How I built my tiny programming language - ModLang"
date = "2021-06-05"
description = "ModLang is a minimal, toy programming language built with C++ and STL."
tags = [
    "c++",
    "project",
	"core",
	"computer science",
]
categories = [
    "tech"
]
+++

# Abstraction
As technology is advancing, the layers of abstraction are also increasing. What was once a necessity to learn in order to create something is now hidden away in abyss. It is not a bad thing but the very essence of advancement.

# Curiosity
I have always been very curious about learning how things worked at the deepest level. But often people (and I myself) have questioned if it is worth digging ocean deep when you know you won't be required to know it in your day to day work, or that you could mostly get away without it.

Even past this sort of reasoning, there comes the notion of reinventing the same wheel when you could do something else "worthwhile". I finally decided to put an end to this cycle of thoughts and just get to "doing" rather than "thinking of doing".

While tinkering around developing and competitive coding, I became ever-so curious about how programming languages came into being and how it all worked in the first place. I wanted to know the magic under the black box.

![Programming Language Chicken Egg](/images/prog_lang_chicken_egg.png)

There's so many languages out there today, each having their own strengths and weaknesses, or I should say, their own purpose(s). It doesn't feel right to compare two languages or claim one is better than the other.

![Programming Language Comparison](/images/prog_lang_comparison.png)

# What are Programming Languages truly?
Programming languages are sets of abstract mathematical rules, definitions, and restrictions to be followed by the programmer while writing the source code. But they are meaningless to the computer unless it is converted into a form that the computer can understand, ie, machine language, or machine code. This conversion (or translation) is done by programs called the Interpreter or the Compiler.

Every programming language can be implemented by a compiler. Every programming language can be implemented by an interpreter. Many modern mainstream programming languages have both interpreted and compiled implementations.

Interestingly, we have Compiler Design subject in this semester (6) and there couldn't be a better time to put the theory learnt into practise.
So I built a minimal, toy programming language, named Mod (or ModLang) in C++ and STL (Standard Template Library) with no other external dependencies. It is dynamically typed and interpreted and it's syntax is a blend of my favourite styles and features in C++, Python and JavaScript.

# How does an Interpreter work?
There are mainly 3 phases in an Interpreter:
1. Lexing (Scanning)
2. Parsing 
3. Evaluating

- Lexing or Scanning is the phase in which the source code is converted to a set of tokens.
- A Parser then builds a data structure called Parse Tree or Abstract Syntax Tree (AST) giving a structural representation of the input and checks for correct syntax in the process.
- Lastly, the Evaluator evaluates the nodes in the AST and produces the desired output.

# Implementation
Following are the rough blueprints of how each phase looks like in code:

## 1. Lexing
Initially, all the tokens - data types, data structures, keywords, and other symbols are defined as per the language design.

```
class Token
{
public:
	TokenType type;
	std::string literal;

	Token() {}
	Token(TokenType type, std::string literal) : type(type), literal(literal) {}
};

const TokenType INTEGER = "INTEGER"; // 'int' data type
const TokenType ARRAY = "ARRAY"; // 'array' data structure
const TokenType IDENT = "IDENT"; // identifiers
const TokenType LET = "LET"; // 'let' keyword
...
```

The sequence of input characters are then converted into corresponding tokens. If there's no match, an error is thrown.

```
class Lexer
{
private:
	std::string input;
	int pos; // current position in input
	int readPos; // current reading position in input
	char curChar; // current char under examination

	void skipWhitespace();
	void readChar();
	char peekChar();
	std::string readIdentifier();
	std::string readNumber();
	std::string readString();

public:
	void New(std::string &input);
	Token nextToken();
};
```

## 2. Parsing
The tokens are wrapped as corresponding nodes and inserted into an AST according to the syntax and statements defined in the language design.

Parsing can be done in 2 ways: 
1. Bottom-up
2. Top-down

Mod has a Bottom-Up Recursive Descent Parser or [Pratt Parser](https://en.wikipedia.org/wiki/Operator-precedence_parser) named after it's inventor Vaughan Pratt (in 1973). It works by calling associated functions repeatedly in a recursive fashion.

```
// Interfaces 

class Node
{
public:
	virtual std::string tokenLiteral() = 0;
	virtual std::string getStringRepr() = 0;
	virtual std::string nodeType() = 0;

	virtual ~Node() {}
};

class Statement : public Node
{
public:
	// " "
	virtual ~Statement() {}
};

class Program : public Node
{
public:
	std::vector<Statement *> statements;
	// " "
	virtual ~Program() {}
};

class Expression : public Node
{
public:
    // " "
	virtual ~Expression() {}
};
```

```
class Parser
{
private:
	Lexer lexer;
	Token curToken;
	Token peekToken;
	std::vector<std::string> errors;

	std::unordered_map<TokenType, Precedence> precedences;

	std::unordered_map<TokenType, PrefixParseFn> prefixParseFns;
	std::unordered_map<TokenType, InfixParseFn> infixParseFns;

	Statement *parseStatement();

	Expression *parseExpression(Precedence precedence);
	Expression *parsePrefixExpression();
	Expression *parseInfixExpression(Expression *);
    ...

	Expression *parseIdentifier();
	Expression *parseIntegerLiteral();
    ...

public:
	void New(Lexer &lexer);
	void nextToken();

	Program *ParseProgram();
};
```

This is by far the hardest phase as there are a lot of aspects to consider in order to correctly process the tokens and create the AST.
Expressions need to be evaluated based on precedence. Based on the location the token is found, we apply associated infix, prefix or postfix functions to it. The whole logic must be very generalized and be applicable in different contexts. This is where it's beauty lies in too! It is so beautifully vowen to produce exactly the intended result whilst returning error(s) by the programmer (if any).

## 3. Evaluating
The nodes of the AST are then evaluated on-the-fly by the Interpreter and hence is named - Tree-Walking.

```
class Evaluator
{
private:
	Object *evalProgram(Program *program, Environment *env);

	Object *evalPrefixExpression(std::string operand, Object *right);
	Object *evalInfixExpression(std::string operand, Object *left, Object *right);

	Object *evalIdentifier(Identifier *ident, Environment *env);
    ...

	Environment *extendFunctionEnv(Function *fn, std::vector<Object *> &args, Environment *outer);

public:
	Object *Eval(Node *node, Environment *env);
};
```

Objects are created during variable definition and are kept track of by the Environment, which is essentially a hash map storing variable name (key) and it's object (value).

# Features of ModLang
- Integer, boolean, and string primitive data types.
- Basic arithmetic for integer expressions.
- If-else conditional statement and while loop statement.
- First class functions, higher-order functions, closures and recursion.
- Data structures like arrays, hashmaps, hashsets, stacks, queues and more.
- Built-in utility functions.
- An REPL (Read Evaluate Print Loop) shell for experimenting.

> The link to the source code of Mod can be found [here](https://github.com/pranavmodx/modlang). I am open to suggestions and contributions.

It is mesmerizing how all of these features are packed into few thousands of lines of code giving a really good bird's eye view of the real world implementation of the same, which in contrast to this, consists of millions of lines of code.

# Future Plans
There are a couple of features I'd love to work on incrementally:  
1. Garbage collection - Currently unused objects or objects out of scope are not deallocated automatically leading to a lot of memory usage and wastage.
2. Class - Bring Object oriented programming paradigm to Mod.
3. Macro - Piece of code in a program that is replaced by the value of the macro.

# What I learnt with this project
- How to write my own programming language!
- Modern C++ and STL
    - better understanding of C++ and its working.
    - implementing interfaces using pure virtual functions.
    - making proper use of pointers - function pointers.
- Use 'make' tool for speeding up development and automating code compilation.

## Mistakes
- Reading too much - I spent too much timing reading and researching and kept implementation to the last. Building on the fly would have been a lot better.
- Making things unecessarily repetitive and boring - failing to adhere to DRY principle, not looking up ways to improve development productivity sooner ('make' tool), silly bugs, forgetting to commit to GitHub, having to rework on things, etc.

## Takeaways
- Low Level / System programming is not too hard as I thought it would be. It just requires a bit more in-depth know-how of things and dedication.

# Acknowledgement
While looking up for resources, I came across few books worth gold, without which the project could not have been possible. Mod is based on the implementation given in these books.
1. [Writing an Interpreter in Go](https://interpreterbook.com/) - [Thorsten Ball](https://www.linkedin.com/in/thorsten-ball-3142b652/)
2. [Crafting Interpreters](http://craftinginterpreters.com/) - [Bob Nystrom](https://www.linkedin.com/in/robert-nystrom-3124052/)

Special thanks to [Lijo Zechariah](https://www.linkedin.com/in/lijo-zechariah-james-12091999/) for coming up with this cool logo for Mod:

![Logo](/images/mod_logo.jpg)