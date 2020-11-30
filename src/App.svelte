<script>
  let textarea;
  export let text = `## Foo Bar

**This** is a test of bold and *italic* 

- This is
- a list
- with **bold** in it and *italic* too`;

  let pointerEvents = false;
  let isWritingList = false;
  let lastKeyPressed;

  function formatText(text) {
      var res = text;

      // Bold
      res = res.replace(
        /\*\*([A-z0-9 ]+)\*\*/gim,
        "<span class='text-indigo-600'>**$1**</span>"
      );

      // Italic
      res = res.replace(
        /[ ]\*([A-z0-9 ]+)\*/gim,
        "<span class='text-green-600'> *$1*</span>"
      );

      // Code
      res = res.replace(
        /`(\S(.*?\S)?)`/gm,
        "<span class='bg-gray-100 rounded-sm text-gray-600'>`$1`</span>"
      );

      // Link title
      res = res.replace(
        /\[(\S(.*?\S)?)\]/gm,
        "[<span class='text-green-600'>$1</span>]"
      );

      // Match URLs
      res = res.replace(
        /(https?:\/\/[^ \)\n]*)/g,
        "<a class='text-blue-600 underline' href='$1'>$1</a>"
      );

      // Match @done
      res = res.replace(
        /\-(.*?)@done/g,
        "<span class='text-gray-400 line-through' href='$1'>- $1 @done</span>"
      );

      // Highlight headers
      res = res.replace(/#/g, `<span class="text-gray-400">#</span>`);

      // Highlight UL lists
      res = res.replace(/\n\-\s/g, `<br/><span class="text-gray-400">- </span>`);

      // Replace newlines with breaks
      res = res.replace(/\n/g, `<br/>`);

    return res;
  }

  function parseKeydown(e) {
    // Disabled tabbing
    if (e.key === "Tab") {
      e.preventDefault();
    }

    // Begin creating Unordered List
    if (e.key === " " && lastKeyPressed === "-") {
      if (text.slice(text.length - 2) === "\n-") {
        isWritingList = true;
      } else {
        isWritingList = false;
      }
    }

    // Autoformat Unordered Lists
    if (e.key === "Enter" && isWritingList) {
      var lastTwoChars = text.slice(-2); // Check the 2 characters on the left

      if (lastTwoChars === "- ") {
        isWritingList = false;
        text = text.substring(0, text.length - 2);
      } else {
        e.preventDefault(); // Stop default linebreak behaviour
        text = `${text}\n- `;
      }
    }

    lastKeyPressed = e.key;
  }

  function handleWindowKeydown(e) {
    if (e.key === "Meta") {
      pointerEvents = true;
    }
  }

  function handleWindowKeyUp(e) {
    if (e.key === "Meta") {
      pointerEvents = false;
    }
  }

  $: formattedText = formatText(text);
</script>

<style>
  textarea {
    caret-color: red;
  }

  textarea::selection {
    background: lightpink;
  }
</style>

<svelte:window on:keydown={handleWindowKeydown} on:keyup={handleWindowKeyUp} />

<div class="flex w-screen h-screen relative">
  <textarea
    spellcheck="false"
    style="-webkit-text-fill-color: transparent;"
    class="p-2 dark:bg-black w-screen h-screen absolute outline-none font-mono"
    on:keydown={parseKeydown}
    bind:this={textarea}
    bind:value={text} />
  <div
    class="p-2 absolute w-screen h-screen select-none text-black dark:text-white font-mono"
    class:pointer-events-none={!pointerEvents}>
    {@html formattedText}
  </div>
</div>
