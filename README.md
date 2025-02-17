<div align="center">

# parrot.nvim 🦜

This is [parrot.nvim](https://github.com/frankroeder/parrot.nvim), the ultimate [stochastic parrot](https://en.wikipedia.org/wiki/Stochastic_parrot) to support your text editing inside Neovim.

[Features](#features) • [Demo](#demo) • [Getting Started](#getting-started) • [Commands](#commands) • [Configuration](#configuration)

<img src="https://github.com/frankroeder/parrot.nvim/assets/19746932/b19c5260-1713-400a-bd55-3faa87f4b509" alt="parrot.nvim logo" width="50%">

</div>

> [!NOTE]⚠️
> This repository is still a work in progress, as large parts of the code are still being simplified and restructured.
> It is based on the brilliant work [gp.nvim](https://github.com/Robitx/gp.nvim) by https://github.com/Robitx.

I started this repository because a perplexity subscription provides $5 of API credits every month for free.
Instead of letting them go to waste, I modified my favorite GPT plugin, [gp.nvim](https://github.com/Robitx/gp.nvim), to meet my needs - a new Neovim plugin was born! 🔥

Unlike [gp.nvim](https://github.com/Robitx/gp.nvim), [parrot.nvim](https://github.com/frankroeder/parrot.nvim) prioritizes a seamless out-of-the-box experience by simplifying functionality and focusing solely on text generation, excluding the integration of DALLE and Whisper.

## Features

- Persistent conversations as markdown files stored within the Neovim standard path or a user-defined location
- Custom hooks for inline text editing with predefined prompts
- Support for multiple providers:
    + [Anthropic API](https://www.anthropic.com/api)
    + [perplexity.ai API](https://blog.perplexity.ai/blog/introducing-pplx-api)
    + [OpenAI API](https://platform.openai.com/)
    + Local and offline serving via [ollama](https://github.com/ollama/ollama)
- Custom agent definitions to determine specific prompt and API parameter combinations, similar to [GPTs](https://openai.com/index/introducing-gpts/)
- Flexible support for providing API credentials from various sources, such as environment variables, bash commands, and your favorite password manager CLI
- Provide repository-specific instructions with a `.parrot.md` file using the command `PrtContext`
- **No** autocompletion and **no** hidden requests in the background to analyze your files

## Demo

Seamlessly switch between providers and agents.
<div align="left">
    <img src="https://github.com/frankroeder/parrot.nvim/assets/19746932/da44ebb0-e705-4ea6-b7c0-1a93c6ba034f" width="100%">
</div>

---

Trigger code completions based on comments.
<div align="left">
    <img src="https://github.com/frankroeder/parrot.nvim/assets/19746932/dc5a0790-b9a2-45ff-90c8-e67eb02f26f3" width="100%">
</div>

---

Let the parrot fix your bugs.
<div align="left">
    <img src="https://github.com/frankroeder/parrot.nvim/assets/19746932/a77fa8b2-9714-42da-bafe-645b540931ab" width="100%">
</div>

## Getting Started

### Dependencies
- [`neovim`](https://github.com/neovim/neovim/releases)
- [`fzf`](https://github.com/junegunn/fzf)
- [`plenary`](https://github.com/nvim-lua/plenary.nvim)
- [`ripgrep`](https://github.com/BurntSushi/ripgrep)

### lazy.nvim
```lua
{
  "frankroeder/parrot.nvim",
  dependencies = { 'ibhagwan/fzf-lua', 'nvim-lua/plenary.nvim' },
  config = function()
    require("parrot").setup {
      providers = {
        pplx = {
          api_key = os.getenv "PERPLEXITY_API_KEY",
          -- OPTIONAL
          -- gpg command
          -- api_key = { "gpg", "--decrypt", vim.fn.expand("$HOME") .. "/pplx_api_key.txt.gpg"  },
          -- macOS security tool
          -- api_key = { "/usr/bin/security", "find-generic-password", "-s pplx-api-key", "-w" },
        },
        openai = {
          api_key = os.getenv "OPENAI_API_KEY",
        },
        anthropic = {
          api_key = os.getenv "ANTHROPIC_API_KEY",
        },
      },
    }
  end,
}
```

## Commands

Below are the available commands that can be configured as keybindings.
These commands are included in the default setup.
Additional useful commands are implemented through hooks (see my example configuration).

### General
| Command                   | Description                                  |
| ------------------        | -------------------------------------------- |
| `PrtNew`                  | open a new chat                              |
| `PrtProvider <provider>`  | switch the provider (empty arg triggers fzf) |
| `PrtAgent <agent>`        | switch the agent (empty arg triggers fzf)    |
| `PrtChatToggle <target>`  | toggle chat window                           |
| `PrtInfo`                 | print plugin config                          |
| `PrtContext`              | edits the local context file                 |
| `PrtAsk`                  | ask the selected agent a single question     |
| `PrtChatFinder`           | fuzzy search chat files using fzf            |
| `PrtChatPaste <target>`   | paste visual selection into the latest chat  |

### Interactive
The following commands can be triggered with visual selections.

| Command                  | Description                                                        |
| ------------------------ | ------------------------------------------------------------------ |
| `PrtChatNew <target>`    | paste visual selection into new chat (defaults to `toggle_target`) |
| `PrtChatToggle <target>` | paste visual selection into new chat (defaults to `toggle_target`) |
| `PrtImplement`           | implements selected comment/instruction                            |

### Chat
The following commands are available within the chat files.

| Command          | Description                                          |
| ---------------- | ---------------------------------------------------- |
| `PrtChatDelete`  | delete the present chat file (requires confirmation) |
| `PrtChatRespond` | trigger chat respond                                 |
| `PrtAsk`         | ask the selected agent a single question             |
| `PrtStop`        | interrupt ongoing respond                            |


## Configuration

### For now, refer to my personal lazy.nvim setup for custom hooks and key bindings.
https://github.com/frankroeder/dotfiles/blob/master/nvim/lua/plugins/parrot.lua

### Adding a new agents

We provide two types of agents that might need different system prompts and API parameters.
To make a new chat agent available, one simply adds a new entry to the list `chat` or to `command`, respectively.

```lua
require("parrot").setup {
    -- ...
    agents = {
        chat = {
          {
              name = "CodeLlama",
              model = { model = "codellama", temperature = 1.5, top_p = 1, num_ctx = 8192, min_p = 0.05 },
              system_prompt = "Help me!",
              provider = "ollama",
          }
        }
    },
    -- ...
}
```
### Adding a new command

```lua
require("parrot").setup {
    -- ...
    hooks = {
      -- PrtAsk simply ask a question that should be answered short and precisely.
      Ask = function(parrot, params)
            local template = [[
            In light of your existing knowledge base, please generate a response that
            is succinct and directly addresses the question posed. Prioritize accuracy
            and relevance in your answer, drawing upon the most recent information
            available to you. Aim to deliver your response in a concise manner,
            focusing on the essence of the inquiry.
            Question: {{command}}
            ]]
            local agent = parrot.get_command_agent()
            parrot.logger.info("Asking agent: " .. agent.name)
            parrot.Prompt(params, parrot.ui.Target.popup, "🤖 Ask ~ ", agent.model, template, "", agent.provider)
      end,
    }
    -- ...
}
```

### Utilizing Template Placeholders

Users can utilize the following placeholders in their templates to inject
specific content into the user messages:

| Placeholder             | Content                              |
|-------------------------|--------------------------------------|
| `{{selection}}`         | Current visual selection             |
| `{{filetype}}`          | Filetype of the current buffer       |
| `{{filepath}}`          | Full path of the current file        |
| `{{filecontent}}`       | Full content of the current buffer   |

Below is an example of how to use these placeholders in a completion hook, which
receives the full file context and the selected code snippet as input.


```lua
require("parrot").setup {
    -- ...
    hooks = {
    	CompleteFullContext = function(prt, params)
    	  local template = [[
          I have the following code from {{filename}}:
        
          ```{{filetype}}
          {{filecontent}}
          ```
          
          Please look at the following section specifically:
          ```{{filetype}}
          {{selection}}
          ```
          
          Please finish the code above carefully and logically.
          Respond just with the snippet of code that should be inserted.
          ]]
    	  local agent = prt.get_command_agent()
    	  prt.Prompt(params, prt.ui.Target.append, nil, agent.model, template, agent.system_prompt, agent.provider)
    	end,
    }
    -- ...
}
```


## Known Issues

- In case of a corrupted state, simply remove the file `~/.local/share/nvim/parrot/persisted/state.json`

more to come ...
