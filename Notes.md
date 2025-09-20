# Technical Notes & Assumptions

## AI Model Choice: Ollama with `llama3.1:8b`

The choice of `llama3.1:8b` was driven by the task's constraints and goals:

1.  **Open-Source & Local:** The primary requirement was to use a local, open-source model. Ollama is the industry standard for simplifying the setup and serving of such models.
2.  **Performance vs. Resources:** The 8-billion parameter model strikes an excellent balance. It is powerful enough for the required task of entity extraction (item codes, quantities) and JSON formatting, while still being lightweight enough to run on consumer-grade hardware without excessive resource consumption.
3.  **Instruction Following:** Llama 3 models are renowned for their strong instruction-following capabilities. This is crucial for reliably adhering to the prompt's request for strict JSON output, which is the cornerstone of the AI parsing logic. Using the `format: "json"` parameter in the Ollama API call further enhances this reliability.

## Fallback and Parsing Logic

The workflow employs a "structured-first" parsing strategy for efficiency and resilience.

1.  **Primary Method (Structured Parsing):** The bot first attempts to parse the user's message using a regular expression (`/\b((BG|SD|DR)\d+)(\s*x\s*(\d+))?/gi`). This method is extremely fast, computationally cheap, and 100% accurate for structured commands (e.g., `BG1 x2, DR1`). This handles power-users and reduces reliance on the LLM.
2.  **Secondary Method (AI Fallback):** If the regex finds no matches, the workflow assumes the user has entered free-text. It then falls back to the LLM. The entire menu is injected into the prompt to provide the necessary context, and the model is instructed to extract the relevant entities. This provides a user-friendly, natural language interface.

This dual-method approach ensures the bot is both efficient and intelligent.

## Assumptions and Simplifications

- **Simplified Checkout:** The checkout flow was intentionally simplified. After the user confirms the total price, the order is finalized immediately. The collection of user details (Name, Phone, Address) was omitted to meet the core requirements within the timebox, though the state-management framework (`checkout_state`) was designed to be easily extendable for this purpose.
- **Hardcoded Menu:** The menu and prices are hardcoded within the workflow's `Code` nodes. For a production system, this would be moved to an external database or a file like `menu.json` and read at runtime.
- **No Authentication:** The bot identifies users solely by their Telegram `chat.id`. There is no formal user authentication system.
- **Synchronous AI Call:** The call to the Ollama LLM is synchronous. For a high-traffic bot, this might be moved to a separate, queued workflow to prevent blocking.
