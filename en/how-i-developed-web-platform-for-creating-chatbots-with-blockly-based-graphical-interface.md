# 🤖 How I Developed a Web Platform for Creating Chatbots with a Blockly-Based Graphical Interface 🧩

Creating a chatbot platform that combines ease of use with flexibility for complex flows was the main goal of [Carubbi.ChatbotStudio](https://github.com/rcarubbi/Carubbi.ChatbotStudio). The platform was designed with a visual interface based on Blockly to enable users to create conversation flows using visual blocks while also implementing a range of advanced features to ensure customization and interactivity. Here, I share how I developed this solution, highlighting the key features, processes, and challenges.

---

### 1. **Platform Goal and Scope**

From the start, the platform was designed to simplify chatbot development without requiring users to program directly. The main requirements included:

- **Intuitive Visual Interface**: Where users could drag and arrange blocks representing each step of the dialogue.
- **Real-Time Testing**: So users could simulate the bot’s behavior while configuring the flow.
- **Development and Production Environments**: To ensure the development environment and adjustments do not affect the bot in production.

### 2. **Choosing Tools and Technologies**

For the bot backend, I used the **Microsoft Bot Framework**, an extensible framework that allows developing scalable and customizable bots. The graphical interface was developed using **Blockly**, which facilitates visual logic and flow construction.

#### Technologies Used:

- **Backend**: Microsoft Bot Framework, C#.
- **Frontend**: Blockly, React, and HTML5.
- **Real-Time Testing Simulator**: Allows testing the flow as it is configured.

### 3. **Implementing the Graphical Interface with Blockly**

The implementation of Blockly was one of the most important aspects, as it enables users to structure the chatbot flow using visual blocks that represent different actions.

1. **Configuring Blockly**: The Blockly interface was set up on the frontend and customized with blocks for common chatbot actions, such as displaying a message, capturing user input, and conducting conditional questions.
2. **Custom Blocks**: Each block represents a type of interaction in the bot flow. For this project, I created specific blocks that allow configuring each part of the dialogue, such as questions and responses.
3. **Converting Blocks to JSON**: Blockly generates a JSON containing the structure of the flow created by the user. This JSON is then sent to the backend, which interprets and executes the flow as configured.

### 4. **Backend Integration with the Microsoft Bot Framework**

The platform's backend was developed to process the JSON generated on the frontend and translate the visual instructions into a conversation flow in the Microsoft Bot Framework. This enables the bot to follow the sequence configured by the user, ensuring flexibility and customization in each conversation.

- **Processing JSON in the Bot Framework**: The JSON is interpreted by the backend, which converts it into dialogues and flows within the Microsoft Bot Framework. Each instruction configured in Blockly is translated into a bot action, such as asking a question or responding to the user.
- **Retrieving User-Provided Values**: To enable flow customization, the bot stores and retrieves information provided by the user at each step. This feature allows the data collected during the conversation to be used later.

### 5. **Value Retrieval and Expressions with Razor Engine**

The platform uses **Razor Engine** to process dynamic expressions and customize the bot’s responses based on values provided by users in previous steps. This feature allows the bot to retrieve variables configured in earlier steps, making the flow more flexible and contextual.

#### Example of Expressions Used in the Project

In the code, a common expression used to access previous values is:

- **`@@Step2.Output.Answer`**: This expression captures the answer (`Answer`) given by the user in step `Step2`, which is a `Question` type step.

This feature is particularly useful for personalizing the dialogue based on the user's previous responses, adjusting the conversational experience in real-time.

### 6. **Intellisense for Suggesting Captured Value Expressions**

To facilitate the use of expressions, the interface includes an **Intellisense** feature that automatically suggests previously captured steps and values as the user types the `@@` expression. This functionality helps users configure the bot more accurately and reduces the likelihood of configuration errors.

### 7. **Managing Development and Production Environments**

The platform also allows configuring and managing **development** and **production** environments using specific variables, ensuring that the bot can be adjusted without affecting the live version.

1. **Environment Variable Configuration**: The project uses environment variables to define important parameters, such as bot identifiers and endpoint URLs. This way, the same code can be used in different environments by simply changing the configuration variables.
2. **Bot Framework-Specific Settings**: The Microsoft Bot Framework settings enable managing production and development flows separately, ensuring that tests and configuration adjustments occur safely in the development environment before any updates are made in production.

### 8. **Real-Time Testing and Validation**

The real-time testing simulator allows users to interact with the bot during the configuration process, testing conversation flows and adjusting them as needed. This feature was crucial to ensure that the bot worked as expected, making it easier to identify necessary adjustments.

---

This project is available as open-source on [GitHub](https://github.com/rcarubbi/Carubbi.ChatbotStudio) and serves as a starting point for anyone interested in developing chatbots with a visual and customizable interface. I invite you to check out the repository and contribute with suggestions, improvements, or adaptations for other projects!

Also, check out the demo I posted on YouTube: https://www.youtube.com/watch?v=Ri5ABGHxGq0
