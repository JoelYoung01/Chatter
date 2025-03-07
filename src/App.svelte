<script>
  import { Ollama } from "ollama";

  const ollama = new Ollama({ host: "http://localhost:11434" });
  const models = ["deepseek-r1:14b"];

  let messages = [];
  let userInput = "";
  let isLoading = false;

  async function sendMessage() {
    if (!userInput.trim()) return;

    const userMessage = { role: "user", content: userInput };
    messages = [...messages, userMessage];
    isLoading = true;
    userInput = null;

    try {
      const response = await ollama.chat({
        model: models[0],
        messages: [...messages],
      });

      const [think, content] = response.message.content
        .replace("<think>", "")
        .split("</think>");

      const assistantMessage = {
        role: "assistant",
        think,
        content,
      };
      messages = [...messages, assistantMessage];
    } catch (error) {
      console.error("Error:", error);
      messages = [
        ...messages,
        { role: "system", content: "Error: Failed to get response" },
      ];
    }

    isLoading = false;
    userInput = "";
  }
</script>

<main>
  <div class="chat-container">
    <div class="messages">
      {#each messages as message}
        <div class="{message.role} message-container">
          <div class="message {message.role}">
            {message.content}
          </div>
        </div>
        <strong>{message.role}</strong>
      {/each}
      {#if isLoading}
        <div class="loading">Thinking...</div>
      {/if}
    </div>

    <form on:submit|preventDefault={sendMessage}>
      <input
        type="text"
        bind:value={userInput}
        placeholder="Type your message..."
        disabled={isLoading}
      />
      <button type="submit" disabled={isLoading}>Send</button>
    </form>

    <button
      class="text-white px-4 sm:px-8 py-2 sm:py-3 bg-sky-700 hover:bg-sky-800"
    >
      Submit
    </button>
  </div>
</main>

<style>
  .chat-container {
    max-width: 75rem;
    margin: 0 auto;
    padding: 0.5rem;
  }

  .messages {
    flex-wrap: wrap;
    overflow-y: auto;
    margin-bottom: 2rem;
    border-radius: 5px;
    height: 80vh;
  }

  .message-container {
    display: flex;
  }

  .message {
    margin-bottom: 10px;
    padding: 10px;
    border-radius: 5px;
    flex: 0 1 80%;
  }

  .user.message-container {
    justify-content: end;
  }

  .user.message {
    background-color: #969696;
  }

  .assistant.message-container {
    justify-content: start;
  }

  .assistant.message {
    background-color: #636363;
  }

  .system {
    background-color: #161616;
  }

  .loading {
    font-style: italic;
    color: #666;
    flex: 80% 0 0;
  }

  form {
    display: flex;
    gap: 10px;
  }

  input {
    flex: 1;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
  }

  button {
    padding: 10px 20px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  button:disabled {
    background-color: #ccc;
  }
</style>
