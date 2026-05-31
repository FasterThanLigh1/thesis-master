# 🚀 AutoITGen: Context-Aware Integration Test Generator

**AutoITGen** is an IntelliJ IDEA plugin that leverages the power of local Large Language Models (LLMs) to automatically generate ready-to-run, True Integration Tests for Java Spring Boot Microservices.

Unlike standard AI coding assistants (e.g., GitHub Copilot or ChatGPT), AutoITGen is specifically designed with deep architectural understanding via **Abstract Syntax Tree (AST)** analysis and a **Self-Healing Agentic Loop**. This ensures highly accurate test generation without ever leaking enterprise source code to external servers.

---

## 🎯 1. Purpose & Motivation

Writing integration tests is often a time-consuming process involving complex data setups, environment configurations, and repetitive boilerplate code. This project was created to:

* **Fully Automate** integration test generation for the Controller layer in Spring Boot applications.
* **Shift the Testing Paradigm:** Upgrade from Slice Tests (using `@MockBean`, which offers lower reliability) to **True Integration Tests** (end-to-end testing from the Controller down to an actual Database using Testcontainers).
* **Eliminate LLM "Hallucinations":** Provide the AI with hyper-detailed context to prevent it from fabricating non-existent methods, fields, or parameters.
* **Ensure Maximum Security (100% Local):** Integrate seamlessly with a local LLM engine (Ollama), guaranteeing that proprietary source code is never sent to the cloud.

---

## 🧠 2. Core Architecture & Methodology

The project adopts a **Multi-Agent System** approach combined with **Static AST Analysis**. It features the following core technologies:

### A. Deep PSI Scanning (Context Extraction)
Instead of feeding raw file text to the AI, the plugin uses IntelliJ's Program Structure Interface (PSI) to intelligently extract contextual information:
* **Dependency Resolution:** Automatically traces the data flow from `Controller` -> `Service` -> `Repository` to accurately identify the required Database interactions.
* **Domain Model Extraction:** Dives directly into `Entity` and `DTO` classes to extract fields and constructors. This forces the LLM to initialize objects correctly, effectively eliminating parameter-mismatch errors.

### B. Template-Driven Prompting (Structural Enforcement)
Establishes strict rules for the AI. It requires absolute adherence to the **Testcontainers (PostgreSQL)** setup and the use of **Hamcrest Matchers** for assertions, strictly prohibiting the use of Mocking libraries (e.g., Mockito).

### C. Post-Generation Sanitization (Source Code Filtering)
Applies *Defensive Programming* principles. Even if the AI generates incorrect imports due to training data remnants, the Regex-based Sanitizer automatically refines the code, enforcing modern libraries and standardizing package/class structures before writing to the file system.

### D. Self-Healing Agentic Loop
The most advanced architectural component of the project:
1. After code generation, the plugin writes the file and invokes the `CodeSmellDetector` (IntelliJ's internal compiler API) to scan for syntax and reference errors.
2. If compilation errors are detected (e.g., *Cannot resolve symbol*), the system packages the error list and sends it back to the LLM with instructions to correct them.
3. This feedback loop runs up to 5 times until the test file achieves **Zero Compilation Errors**.

---

## ⚙️ 3. Execution Flow

1. **Trigger:** The user right-clicks inside a Controller file and selects *Generate Integration Test*.
2. **Select API:** A UI dialog displays available Endpoints for the user to select.
3. **Context Building:** The system triggers Deep PSI Scanning to gather contextual data (Routes, Implementation Code, Repositories, Entity Schemas).
4. **LLM Generation:** Asynchronous communication with Ollama (port 11434). This includes safe timeouts and a Kill Switch (the user can abort the process at any time).
5. **Validation & Healing:** Static error scanning. If errors are found, the system automatically loops back to step 4 for self-healing.
6. **Final Polish:** Auto-imports standard libraries, formats the code according to IDE style guidelines, and presents the final test to the user.

---

## 📊 4. Experimental Results

The system has proven vastly superior to manually copying and pasting code into standard Chatbots:

* **High Compilation Rate:** Achieved exceptionally high success rates thanks to the robust Auto-Import system and Regex Sanitizer.
* **Significantly Improved Pass@1:** The code generated on the first attempt (or after self-healing) can often be executed and passed immediately without human intervention (Zero-touch generation).
* **Zero Hallucination Context:** Utilizing Deep PSI to scan Entity Constructors completely eliminated the AI's tendency to "guess" database structures and object parameters.
* **Execution Speed:** Operates smoothly and seamlessly with small to medium models (like Qwen 2.5 Coder 7B) running locally on Apple Silicon architecture.

---

## 🛠 5. Tech Stack

* **Language:** Kotlin (Plugin Development), Java (Target Code)
* **IDE Core:** IntelliJ Platform SDK, PSI (Program Structure Interface), CodeSmellDetector
* **AI Backend:** Ollama (Local LLM Engine)
* **Recommended Model:** `qwen2.5-coder:7b`
* **Target Frameworks:** Spring Boot 3.x, JUnit 5, MockMvc, Testcontainers, PostgreSQL
