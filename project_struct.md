# Overall Objective

The project is designed to let a multimodal model (for example, GPT‑4‑Vision or variants like GPT‑4‑with‑OCR) interact with the computer. In other words, the model “sees” a screenshot of your computer’s display, decides what action to take next (such as clicking a button, writing text, or pressing a key), and then your computer executes that action. This cycle allows for an autonomous loop that mimics a human operator interacting with the user interface.

---

## High-Level Structure

The project is organized as a Python package with several submodules. Here’s how the different parts fit together:

- **Entry Point (Command-Line Interface):**  
  The project is started via a console script named “operate.” When you run the command (for example, `operate` or `operate -m some-model`), the entry point is defined in the module responsible for parsing arguments and starting the application.

- **Configuration Management:**  
  A configuration module loads environment variables (such as API keys) and sets up API clients. This part of the project ensures that the required credentials are present and provides helper functions to initialize connections with external services like OpenAI, Google, or Anthropic.

- **Main Operation Loop:**  
  The central logic of the application that ties everything together is found in a module that:
  - Initializes the configuration (and, if needed, a voice input subsystem).
  - Displays introductory dialogs or messages.
  - Delegates execution to a main function that orchestrates the workflow.

- **Prompts and Instruction Templates:**  
  A set of prompt templates is provided in a dedicated module. These templates define:
  - The system instructions that tell the model how to operate the computer.
  - The “user question” prompt that invites the next action.
  
  In practice, the proper prompt is selected and formatted (for example, by inserting the current objective and operating system commands) before being sent to the multimodal model.

- **API Interaction and Model Requests:**  
  When it comes time to decide on an action, there is a module that:
  - Determines which model or API client to use (based on the selected mode such as “gpt-4-with-ocr”, “gemini-pro-vision”, etc.).
  - Prepares the request by (for example) capturing a screenshot, encoding it, and then calling the remote API.
  - Processes the response (often returning JSON instructions), which includes further cleaning up or verifying the data.
  
  This module also contains fallback methods for error handling or when a given model is not recognized.

- **Utilities:**  
  Several helper functions reside in a utilities subpackage. These include:
  - **Styling utilities:** For ANSI color codes and style definitions to format terminal output.
  - **OCR utilities:** For processing screenshots—extracting text, drawing bounding boxes, and calculating coordinates. These functions are critical for enabling the multimodal model to “understand” what it sees on screen.
  - **Other helpers:** Functions to capture screenshots, label clickable areas, or convert image coordinates.

- **Evaluation:**  
  An evaluation script exists to run automated test cases. It:
  - Invokes the main “operate” command in a subprocess with a specified prompt.
  - Captures the resulting screenshot.
  - Sends the screenshot to a secondary evaluation function or API to check if the expected guideline (for example, “a Github page is visible”) is met.
  
  The evaluation logic reports whether each test case passed or failed.

- **Packaging and Distribution:**  
  A setup script organizes the package, defines dependencies, declares the console script entry point, and even specifies additional package data (like model weights). This allows the project to be installed via pip.

---

## Detailed Walkthrough of Key Components

### 1. Entry Point and Argument Parsing (`operate/main.py`)
- **Purpose:**  
  This script is the user’s starting point. It uses the Python argparse module to read command-line arguments like:
  - Model selection (`-m` or `--model`).
  - Voice mode (`--voice`).
  - Verbose mode (`--verbose`).
  - Direct objective prompt (`--prompt`).
- **Function:**  
  The parsed arguments are passed to the main function (located in another module) which launches the operating loop.
  
### 2. Central Operation (`operate/operate.py`)
- **Purpose:**  
  This module contains the `main()` function, which:
  - Loads the configuration.
  - Validates the API keys.
  - Optionally initializes a voice input system if voice mode is enabled.
  - Displays an introductory dialog (or directly processes the prompt) before handing control over to the model interaction logic.
- **Interaction with Prompts:**  
  It imports prompt templates (like `USER_QUESTION`) and uses them to format the output you see on the terminal.

### 3. Prompt Templates (`operate/models/prompts.py`)
- **Purpose:**  
  Here, you’ll find all the template strings that define how instructions and questions are presented to the model:
  - The **USER_QUESTION** prompt (e.g., “Hello, I can help you with anything. What would you like done?”).
  - The **SYSTEM_PROMPT** templates (standard, labeled, OCR) that specify the available operations and provide examples.
- **Customization:**  
  The prompts are formatted with dynamic values such as the operating system type and the user’s objective, ensuring that the model receives context-aware instructions.

### 4. API Functions (`operate/models/apis.py`)
- **Purpose:**  
  This module implements functions that directly interact with external APIs (for example, OpenAI’s GPT‑4‑Vision model or Google’s Gemini model). It:
  - Chooses the appropriate API based on the selected model.
  - Prepares a payload that includes both text and image (a captured screenshot).
  - Handles the API response, cleans the returned JSON, and then returns actionable instructions.
- **Error Handling:**  
  Contains logic to retry the API call or fall back to an alternate approach if something goes wrong.

### 5. Configuration (`operate/config.py`)
- **Purpose:**  
  This module creates the configuration object for the entire application:
  - Loads environment variables and API keys from a `.env` file.
  - Provides methods to ensure required keys (like OPENAI_API_KEY) are present. If not found, it prompts the user via a dialog.
  - Caches client objects for OpenAI, Google, and other services to avoid multiple initializations.
  
### 6. Utilities
- **Style Utilities (`operate/utils/style.py`):**  
  Contains ANSI escape codes and style definitions to give colored and formatted output in the console.
  
- **OCR Utilities (`operate/utils/ocr.py`):**  
  Implements functionality to process screenshots:
  - Uses EasyOCR to extract text.
  - Calculates bounding boxes and the center coordinates (expressed as percentages) where a click might be needed.
  
- **Other Helpers:**  
  Other modules (like screenshot capture or labeling utilities) help process the image and link the resulting coordinates to the action commands generated by the model.

### 7. Evaluation (`evaluate.py`)
- **Purpose:**  
  This script is used to automatically test the system:
  - It runs a test case by starting the “operate” command with a specific model and prompt.
  - After execution, it uses OCR and other image-based analysis to check if the expected outcome (for example, a particular webpage being visible) is met.
- **Feedback Loop:**  
  The tool prints out pass/fail status along with detailed justifications.

### 8. Packaging (`setup.py`)
- **Purpose:**  
  The setup script packages the project so that it can be installed and run from the command line. It specifies:
  - The project dependencies.
  - The console script entry point.
  - Any additional package data such as model weights.

---

## How It All Works Together

1. **Start-Up:**  
   When you run the command (e.g., `operate`), the entry point (`operate/main.py`) parses your command-line arguments and calls the main function in `operate/operate.py`.

2. **Initialization:**  
   The main function loads the configuration from the `.env` file, validates necessary API keys, and determines if additional modes (like voice) should be activated.

3. **Prompting the Model:**  
   In the workflow, the system displays a message (using prompt templates from `operate/models/prompts.py`) to activate the next action cycle. The user’s objective or a direct prompt may be included.

4. **Capturing the Screen & Sending API Request:**  
   The system captures a screenshot (possibly with the cursor visible) and prepares a message that contains both text (the prompt) and image data. In `operate/models/apis.py`, the appropriate API function is triggered based on the chosen model (for instance, GPT‑4‑Vision).

5. **Processing Response:**  
   The external model returns a response that typically contains a list of actions in JSON format. This response is cleaned, parsed, and then added to the list of messages.

6. **Executing the Action:**  
   In a real scenario, the action instructions might then be used to simulate mouse clicks, keyboard inputs, etc. (via libraries like pyautogui). Even if the actual execution isn’t shown in every snippet, the prompts and instructions are designed for that purpose.

7. **Evaluation Loop:**  
   For testing or validation, the evaluation script runs test cases that simulate objectives, automatically triggering the operating process and then verifying whether the output (the screenshots) meets the expected guidelines.

---

## Summary

- The project is structured as a multi-module Python package.
- The **entry point** (`main.py`) handles command-line arguments and starts the operating loop.
- The **operate module** orchestrates the work by loading configuration, setting up dialogs, and calling API functions.
- **Prompt templates** (in `models/prompts.py`) provide the natural language instructions for the model.
- **API functions** (in `models/apis.py`) interact with external services to obtain a decision (action) based on the current screen and prompt.
- **Configuration** (`config.py`) centralizes the management of API keys and client initialization.
- **Utilities** handle everything from console styling to OCR and image processing.
- **Evaluation** scripts run automated tests to verify that the system meets its objectives.
- **Packaging** allows the project to be easily installed and distributed.

Each component interacts with the others in a cycle: the system gathers data (a screenshot), sends it to an AI model with a prompt, receives instructions back in JSON format, and then (optionally) executes those instructions on the computer. This loop forms the backbone of the “self‑operating‑computer” framework, enabling it to autonomously interact with your operating system.