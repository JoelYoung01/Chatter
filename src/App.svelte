<script>
  import { Ollama } from "ollama";
  import { marked } from "marked";

  const ollama = new Ollama({ host: "http://localhost:11434" });

  let models = $state([]);
  let messages = $state([]);
  let selectedModel = $state(null);
  let error = $state(null);
  let userInput = $state(null);
  let isLoading = $state(false);
  let sidebarOpen = $state(false);

  async function sendMessage(evt) {
    evt.preventDefault();

    if (!userInput.trim()) return;

    const userMessage = { role: "User", content: userInput };
    messages = [...messages, userMessage];
    isLoading = true;
    userInput = null;

    try {
      // Add an initial assistant message that we'll update
      const assistantMessage = {
        role: "Assistant",
        content: "",
      };
      messages = [...messages, assistantMessage];

      const stream = await ollama.chat({
        model: selectedModel.name,
        messages: messages.slice(0, -1), // Exclude the empty assistant message
        stream: true,
      });

      for await (const chunk of stream) {
        // Update the last message's content with the new chunk
        assistantMessage.content += chunk.message.content;
        messages = [...messages.slice(0, -1), assistantMessage];
      }

      // Handle think/content split if present
      if (assistantMessage.content.includes("<think>")) {
        const [think, content] = assistantMessage.content
          .replace("<think>", "")
          .split("</think>");

        assistantMessage.think = think;
        assistantMessage.content = content;
        messages = [...messages.slice(0, -1), assistantMessage];
      }
    } catch (error) {
      console.error("Error:", error);
      error = "Error: Failed to get response";
    }

    isLoading = false;
    userInput = "";
  }

  function toggleSidebar() {
    sidebarOpen = !sidebarOpen;
  }

  function resetMessages() {
    messages = [];
  }

  async function initialize() {
    const listResult = await ollama.list();
    models = listResult.models;

    selectedModel = models.at(0);
  }

  initialize();
</script>

<aside
  class={`fixed top-0 z-50 h-screen w-64 bg-gray-800 p-4 transition-all duration-300 ease-in-out ${
    sidebarOpen ? "left-0" : "-left-64"
  }`}
>
  <div class="flex h-full flex-col items-stretch">
    <h4 class="mb-3 text-lg font-bold text-white">Menu</h4>

    <label for="model-select">Select Model</label>
    <select
      name="models"
      id="model-select"
      bind:value={selectedModel}
      class="mt-2 w-full rounded-md border border-gray-600 bg-gray-700 px-3 py-2 text-white"
    >
      {#each models as model}
        <option value={model} class="bg-gray-700 text-white"
          >{model.name}</option
        >
      {/each}
    </select>

    <div class="grow-1"></div>

    <button
      class="rounded-md border border-gray-600 bg-gray-700 px-3 py-2 text-white"
      onclick={resetMessages}
    >
      Reset Messages
    </button>
  </div>
</aside>

<button
  aria-label="Toggle menu"
  class="fixed top-4 left-4 z-60 cursor-pointer p-2 text-gray-400 hover:text-white"
  onclick={toggleSidebar}
>
  <svg viewBox="0 0 24 24" class="h-6 w-6">
    <path
      fill="currentColor"
      d="M3,6H21V8H3V6M3,11H21V13H3V11M3,16H21V18H3V16Z"
    />
  </svg>
</button>

<main class="flex">
  <div class="mx-auto w-full max-w-6xl p-2 pl-12">
    <div class="mb-8 h-[85vh] overflow-y-auto">
      {#each messages as message}
        {@const alignment = message.role === "User" ? "end" : "start"}
        <div
          class={`flex ${message.role === "User" ? "flex-row-reverse" : ""}`}
        >
          <div class="basis-content mb-2 text-{alignment}">
            <div
              class={`rounded-lg p-2 ${
                message.role === "User" ? "bg-blue-600" : "bg-gray-600"
              }`}
            >
              {@html marked(message.content)}
            </div>
          </div>
          <div class="shrink-0 grow-0 basis-1/5"></div>
        </div>
      {/each}
      {#if error}
        <div class="rounded-lg bg-gray-800 p-2">{error}</div>
      {/if}

      {#if isLoading}
        <div class="text-gray-500 italic">Thinking...</div>
      {/if}
    </div>

    <form onsubmit={sendMessage} class="flex gap-2">
      <input
        type="text"
        bind:value={userInput}
        placeholder="Type your message..."
        disabled={isLoading}
        class="flex-1 rounded-lg border border-gray-300 p-2 focus:border-blue-500 focus:outline-none"
      />
      <button
        type="submit"
        disabled={isLoading}
        class="rounded-lg bg-blue-600 px-5 py-2 text-white hover:bg-blue-700 disabled:bg-gray-400"
      >
        Send
      </button>
    </form>
  </div>
</main>
