# n8n WhatsApp & Google Drive AI Assistant: System Documentation


<div align="center">
# WATCH THE VIDEO
[![Watch the video](https://img.youtube.com/vi/dqWlBKcJMMk/0.jpg)](https://youtu.be/dqWlBKcJMMk)

</div>

<img width="1721" height="895" alt="n8nWorkflow" src="https://github.com/user-attachments/assets/dc295e84-2c9f-4db3-8370-c74033577d03" />


## Abstract
This document provides a comprehensive technical overview of a workflow that transforms the WhatsApp messaging platform into a robust command-line interface for managing Google Drive files. The system is enhanced with Artificial Intelligence-powered summarization capabilities, enabling users to execute list, move, delete, and summarize operations on files and folders remotely.

---

## Table of Contents
- [Features](#features)  
- [Architecture](#architecture)  
- [Prerequisites](#prerequisites)  
- [Setup and Configuration](#setup-and-configuration)  
  - [Step 1: Facebook Developer Portal and WhatsApp Setup](#step-1-facebook-developer-portal-and-whatsapp-setup)  
  - [Step 2: Google Cloud Console and Credential Provisioning](#step-2-google-cloud-console-and-credential-provisioning)  
  - [Step 3: n8n Credential Configuration](#step-3-n8n-credential-configuration)  
  - [Step 4: Environment Variables](#step-4-environment-variables)  
- [Deployment](#deployment)  
- [Command Syntax Reference](#command-syntax-reference)  
- [Workflow Design](#workflow-design)  
- [Error Handling](#error-handling)  
- [Security](#security)  
- [Extending the Workflow](#extending-the-workflow)  

---

## 1. System Features
- **Remote File Management:** Enables interaction with Google Drive through standardized text-based commands via the WhatsApp interface.  
- **Core Functionality:** Includes essential commands such as LIST, DELETE, MOVE, and SUMMARY.  
- **Artificial Intelligence Integration:** Provides concise, AI-generated summaries for various file formats, including text documents and images, utilizing Google's Gemini models.  
- **Comprehensive Audit Trail:** Maintains a detailed log of all executed commands within a designated Google Sheet for security and tracking purposes.  
- **Secure, Scoped Access:** Operates on the principle of least privilege using the OAuth 2.0 protocol, ensuring access is strictly limited to the authenticated user's Google Drive.  
- **Modular and Extensible Architecture:** Designed to facilitate the straightforward integration of new commands and functionalities.  

---

## 2. Workflow Architecture
The workflow is initiated by an incoming message on the WhatsApp platform. A parsing mechanism interprets the command, which is then routed through a conditional switch to the appropriate functional branch. This branch interacts with the Google Drive API to perform the requested action. If required, Artificial Intelligence models are invoked. Subsequently, a confirmation or result message is transmitted back to the user, and the transaction is recorded in an audit log.

- **WhatsApp Trigger:** The workflow commences upon receipt of an incoming message.  
- **Command Parser:** A Python-based node employs regular expressions to deconstruct the message into a primary command and its corresponding arguments.  
- **Conditional Routing (Switch Node):** The parsed command dictates the execution path, routing the process to the relevant logic branch (LIST, DELETE, MOVE, or SUMMARY).  
- **Google Drive API Integration:** Each branch retrieves the necessary file or folder metadata from Google Drive.  

### Action Execution:
- **LIST:** Formats and returns a structured list of files and folders.  
- **DELETE:** Executes the deletion of the specified resource.  
- **MOVE:** Relocates a file to a new destination folder.  
- **SUMMARY:** Manages the download of specified files, dispatches the content to a Gemini model for summarization, and processes various MIME types.  

- **User-Facing Responder:** Transmits a status update or the result of the operation back to the originating user.  
- **Audit Log:** Appends a new record to a Google Sheet, documenting the command, user details, and execution status.  

---

## 3. Implementation and Configuration Guide

### Prerequisites
- A Meta for Developers account.  
- A Google Cloud Platform account.  
- Docker and Docker Compose installed on the host machine.  
- A registered domain or a network tunneling service (e.g., ngrok) to expose the local n8n instance to the public internet for webhook functionality.  

### Step 1: Facebook Developer Portal and WhatsApp Configuration
- **Application Creation:** Navigate to Meta for Developers, select "Create App," and choose the "Business" type. Assign a name to the application and associate it with a Meta Business Account.  
- **WhatsApp Product Integration:** From the application dashboard, locate and set up the "WhatsApp" product. This action will provision a Business Account and a test phone number.  
- **Credential Retrieval:** Proceed to the WhatsApp -> API Setup section. The temporary Access Token and Phone Number ID required for n8n credentials will be displayed. Add and verify a recipient phone number for testing purposes.  
- **Webhook Configuration:** The n8n "WhatsApp Trigger" node will generate a unique webhook URL. Return to the WhatsApp -> Configuration section in the Meta Developer Portal, edit the webhook settings, and input the n8n URL into the Callback URL field. Establish a secure, unique string for the Verify Token, which must be replicated in the n8n WhatsApp credential configuration.  

### Step 2: Google Cloud Console and Credential Provisioning
- **Project Creation:** Institute a new project within the Google Cloud Console.  
- **API Enablement:** Navigate to the APIs & Services -> Library and enable the following APIs: Google Drive API, Google Sheets API, and Vertex AI API.  
- **OAuth 2.0 Credential Generation:** In the APIs & Services -> Credentials section, create an OAuth client ID for a Web application. In the Authorized redirect URIs field, add the specific OAuth redirect URL provided by the n8n Google credential configuration page (`https://<your-n8n-domain>/rest/oauth2-credential/callback`). Upon creation, the Client ID and Client Secret will be displayed; these must be stored securely.  
- **API Key Generation:** In the same credentials section, create a new API key. This key should be restricted to only permit access to the "Vertex AI API" to enhance security.  

### Step 3: n8n Credential Configuration
Within the n8n instance, navigate to the Credentials section and create the following:
- WhatsApp: Utilize the Access Token and Verify Token obtained from the Facebook Developer Portal.  
- Google (OAuth2): Utilize the Client ID and Client Secret from the Google Cloud Console. This will require an authentication flow to grant the necessary permissions.  
- Google Gemini (PaLM) API: Utilize the API Key generated from the Google Cloud Console.  

### Step 4: Environment Variable Setup
- A `.env` file must be created in the root directory of the Docker Compose project. The provided `.env-sample` file should be used as a template to populate the required secret values.  

---


## 4. Deployment via Docker

The recommended method for deployment is via Docker.

1.  **File Colocation**: Ensure the `docker-compose.yml`, `.env`, and `start-n8n.sh` files are located in the same directory.

2.  **Script Execution Permissions**: Grant execution permissions to the startup script by running the following command in your terminal:
    ```bash
    chmod +x start-n8n.sh
    ```

3.  **Service Initiation**: Execute the startup script:
    ```bash
    ./start-n8n.sh
    ```

4.  **Workflow Importation**: Access the n8n instance via a web browser and import the `workflow.json` file.

5.  **Credential Association**: Manually associate the previously configured credentials with their respective nodes within the workflow.

6.  **Workflow Activation**: Save and activate the workflow to begin operation.

---

## 5. Command Syntax Reference

The system employs a standardized command syntax for all operations.

* `LIST [path]`
    * **Function**: Lists the contents of a specified directory. If no path is provided, it defaults to the root directory.
    * **Example**: `LIST /Projects/Q3`

* `DELETE <path/to/resource>`
    * **Function**: Deletes the specified file or folder.
    * **Example**: `DELETE /Archive/draft.txt`

* `MOVE <source_path> to <destination_path>`
    * **Function**: Relocates a source file to a new destination folder.
    * **Example**: `MOVE /report.pdf to /Archived/Reports`

* `SUMMARY <path/to/resource>`
    * **Function**: Generates a summary of a single file or of all files within a specified folder.
    * **Example**: `SUMMARY /Meeting Notes/summary.docx`

---

## 6. Workflow Design Principles

The workflow's architecture prioritizes clarity and maintainability through the strategic use of Node Groups and descriptive Annotations.

* **Groups**: Each primary command's logic is encapsulated within a distinct, color-coded group box for visual organization.
* **Annotations**: "Sticky notes" are utilized to document the purpose of complex nodes, particularly code nodes responsible for path parsing and message formatting.
* **Nomenclature**: All nodes are assigned clear, descriptive names (e.g., `"Parse Command"`, `"Search Files and Folders for Deletion"`) to explicitly state their function.

---

## 7. Error Handling and System Feedback

A comprehensive error-handling system is implemented to provide users with clear and timely feedback.

* **Invalid Command**: If a command fails to parse, a message is returned indicating an invalid format.
* **Resource Not Found**: If a specified file or folder does not exist, a notification is sent to the user.
* **Action-Specific Errors**: Contextual error messages are provided for specific failures, such as an invalid destination type for the `MOVE` command.
* **Success Confirmation**: All successful operations are confirmed with an explicit message or by the delivery of the requested data.

---

## 8. Security Protocols

* **Environment Variables**: All secrets, including API keys, tokens, and identifiers, are managed exclusively through a `.env` file to prevent hardcoding.
* **Scoped Permissions (OAuth)**: The Google OAuth credential requests a minimal set of permissions (`drive.readonly`, `drive.file`, `sheets.append`), adhering to the principle of least privilege.
* **Webhook Verification**: The WhatsApp webhook is secured with a secret verify token, ensuring that inbound requests originate exclusively from Meta's servers.
* **Audit Logging**: All executed commands are systematically logged, providing a complete and immutable record of system activity.

---

## 9. System Extensibility

The modular design of the workflow architecture is intended to facilitate the straightforward extension of its functionality with new commands. The process for integrating a new capability is systematic and involves the following key stages, which collectively ensure that new features are added in a structured, maintainable, and secure manner.

1.  **Update Command Parser Node**
    The initial and most critical step is to modify the `"parseCode"` node. This node functions as the gateway for all incoming commands, utilizing regular expressions to validate and deconstruct user input into a standardized format. To introduce a new command, a new regex pattern must be carefully engineered to recognize its unique syntax and arguments. This pattern must be specific enough to prevent ambiguity with existing commands and should structure its output consistently (e.g., producing `command`, `arg1`, `arg2` fields) to ensure seamless processing by all downstream nodes.

2.  **Extend Conditional Routing (Switch Node)**
    Following the parser update, the primary `"Switch"` node, which serves as the central router for the workflow, must be configured. This involves adding a new output branch that corresponds to the new command string produced by the parser. This simple configuration change ensures that upon recognizing the new command, the workflow's execution is correctly directed to the appropriate logic path, a design choice that underscores the system's inherent modularity and ease of modification.

3.  **Implement Command Logic**
    This stage involves constructing the core operational logic for the new command. It is considered best practice to encapsulate this logic within a dedicated, clearly labeled **Node Group**. This approach enhances organizational clarity and simplifies future maintenance. For instance, implementing a `RENAME` command would necessitate a logical sequence of nodes: first, a Google Drive `"Search"` node to locate the target file; second, an `"Update File"` node to execute the rename operation itself; and finally, a `"Set"` node to formulate a context-aware response message for the user.

4.  **Integrate, Test, and Finalize**

    The final stage involves integrating the new logic group into the main workflow. The corresponding output from the `Switch` node must be connected to the input of the new logic group. Crucially, robust user feedback mechanisms must be implemented to return clear success or failure messages to the user via WhatsApp. A successful rename, for example, should confirm both the old and new filenames. The final connection in the flow must be to the `"Append row in sheet"` node to ensure the action is recorded in the audit log, thereby maintaining a complete and immutable record of all system activity. End-to-end testing, initiated from a WhatsApp client, is essential to validate the entire process, from command input to final confirmation and logging.


