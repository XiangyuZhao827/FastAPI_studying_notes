FastAPI Study Notes
A Chinese-language collection of FastAPI study notes, chapter exercises, and review materials, organized into 50 chapters following the learning sequence of the official FastAPI Tutorial — User Guide.

This repository provides a structured path for learning FastAPI, revisiting key concepts, and checking your understanding through exercises.
Project status: The repository contains 50 chapter directories. Six chapters currently include learning-status notes, exercises, and official documentation references instead of complete explanatory notes.

Contents
- Chapter notes covering request handling, validation, dependencies, security, databases, testing, and more.
- Two multiple-choice questions with answers and explanations for each chapter.
- A comprehensive review containing 100 multiple-choice questions and 10 discussion questions with explanations.
- A focused review of larger applications, JSON Lines, and Server-Sent Events.
- Supplementary explanations in selected chapters.
The learning materials use Markdown for convenient reading, editing, and version control.
Topics Covered
Area	Topics
Getting started	First steps, path parameters, query parameters, and request bodies
Validation and models	String and numeric validation, nested models, example data, and additional data types
HTTP inputs and responses	Cookies, headers, response models, status codes, forms, and file uploads
Application behavior	Error handling, path operation configuration, JSON encoding, and partial updates
Dependencies	Dependency injection, classes, sub-dependencies, global dependencies, and dependencies with yield
Security	OAuth2, bearer authentication, current-user handling, password hashing, and JWT
Application structure	Middleware, CORS, relational databases, and applications with multiple files
Streaming	JSON Lines and Server-Sent Events
Development and delivery	Background tasks, documentation configuration, frontend topics, static files, testing, and debugging


Repository Structure
FastAPI_study_notes/
├── README.md
├── README_从这里开始.md
├── 01_第一步/
├── 02_路径参数/
├── 03_查询参数/
├── ...
├── 49_测试/
├── 50_调试/
├── 综合复习_50章100道选择题与10道思考题_含解析.md
└── 综合复习_更大的应用_JSONL_SSE_导读与三章复盘.md
Common chapter filenames include:
Filename	Purpose
学习文档.md	Main chapter notes
本章自测_两道题含答案解析.md	Two self-assessment questions with answers and explanations
学习文档_待系统讲解.md	Learning-status information for a chapter awaiting complete notes
补充_*.md	Supplementary explanations, where available


Getting Started
Clone the repository:
git clone https://github.com/XiangyuZhao827/FastAPI_study_notes.git
cd FastAPI_study_notes
Open the folder in your preferred Markdown reader or editor, such as VS Code or Typora. You can also browse the documents directly on GitHub.

Reading the materials requires no Python installation. Running code examples requires a suitable Python environment and the dependencies used by each example.
Suggested Learning Path
1. Read README_从这里开始.md for the detailed Chinese-language guide.
2. Work through the chapter directories in numerical order.
3. Read the chapter notes and try the examples.
4. Answer the chapter questions before reviewing their explanations.
5. Use the comprehensive review to identify topics that need further practice.
6. Revisit the focused review when studying application structure and streaming.
Coverage and Material Sources
The collection includes materials with different levels of completeness and provenance:
Material type	Chapters
Original standalone Markdown notes	9
Explanations recovered from earlier learning conversations and separated by chapter	24
Review outlines reorganized from earlier conversations	11
Chapters awaiting complete explanatory notes	6
Total	50


The chapters awaiting complete explanatory notes are:
- 10: Nested Models
- 12: Extra Data Types
- 19: Response Status Codes
- 39: Middleware
- 40: CORS
- 41: SQL Databases
Reorganized outlines and recovered explanations may differ in depth from the original standalone notes.
Official Documentation
Consult the official FastAPI documentation alongside these notes, especially when checking API behavior, dependency versions, and newer features.

This repository is an independent learning collection and is not an official FastAPI publication.
