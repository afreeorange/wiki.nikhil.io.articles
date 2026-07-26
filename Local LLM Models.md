On an M2 MacBook Air with 24GB memory. There's [`llmfit`](https://www.llmfit.org/) which I found to be extremely helpful and useful.

## Running Models Locally

TODO: There's a host of options.

### Formats

TODO: MLX, GGUF, etc

### Nomenclature

TODO: Reading files....

### Llama.cpp

This is what I'm using. [Homepage](https://llama.app/) and [list of models](https://llama.app/models). It has a nice Web Interface.

```bash
# Start a web UI
llama-server -hf bartowski/Qwen_Qwen3.5-2B-GGUF:Q4_K_M

# Start a CLI
llama-cli -hf bartowski/Qwen_Qwen3.5-2B-GGUF:Q4_K_M
```

### Trials

- `bartowski/Qwen_Qwen3.5-2B-GGUF` -- First one! Was fine.
- `bartowski/Qwen_Qwen3.5-9B-GGUF` -- Slower but fine.
- `bartowski/Qwen_Qwen3.6-27B-GGUF` -- Did not run. Memory problems. Expected.

## Model Terminology

TODO:

## Other

Models are stored in `~/.cache/huggingface`.

## Pi Agent Harness

[This is a really cool project](https://pi.dev/). You can type `/model` and switch models easily (i.e. you're not just restricted to Claude Code, for instance) and this includes local models. Installed via

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent

# Now link it to Llama and it will autodiscover local models!

# First serve the models. Note the host and port.
llama serve --host localhost --port 8080

# Then install the extension
pi install git:github.com/huggingface/pi-llama

# Now run
pi
```

You will see the local models when you type `/models` and search for "llama-cpp". [Here's a good post](https://roman.pt/posts/pi-dev-version/) on using it. You get very few tools and are expected to build the rest and place them in `$HOME/.pi`.

## Resources

- https://huggingface.co/bartowski
- https://huggingface.co/Brooooooklyn
- [Friends Don't Let Friends Use Ollama](https://sleepingrobots.com/dreams/stop-using-ollama/) -- Not sure what it offers above llama.cpp tbh. Great article with history.
- [Running local models on an M4 with 24GB memory](https://jola.dev/posts/running-local-models-on-m4)
