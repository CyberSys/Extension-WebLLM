# WebLLM

Provides an interface for extensions to use language models directly in the browser.

Powered by [@mlc-ai/web-llm](https://github.com/mlc-ai/web-llm).

## Installation

Requirements: SillyTavern 1.12.4 or later.

Install using the link:

```txt
https://github.com/SillyTavern/Extension-WebLLM
```

## Public API

**Note: For simplicity, concurrent requests are disabled and queued.**

### Get engine

Access the default API engine instance from the `SillyTavern.llm` object, or create your private instance using the `SillyTavern.llm.getEngine` method.

Method arguments:

* modelId: The model ID to use.
* silent: If `true`, the engine will not display any toast messages.

```js
const modelId = 'gemma-2-2b-it-q4f16_1-MLC';
const silent = false;
const engine = SillyTavern.llm.getEngine(modelId, silent);
```

### Get models

To get all the available models, use the `getModels` method.

Returns an array of objects with the following properties:

* id: The model ID.
* vram_required: The amount of VRAM required to load the model in MB.
* context_size: The maximum number of tokens that can be processed in a single request.

```js
const models = await SillyTavern.llm.getModels();
// models = [{id: 'gemma-2-2b-it-q4f16_1-MLC', vram_required: 1895.3, context_size: 4096}, ...]
```

### Get current model

To get the current model (even if not loaded), use the `getCurrentModel` method.

If the engine has not been initialized, it will return `null`.

```js
const modelId = 'gemma-2-2b-it-q4f16_1-MLC';
const engine = SillyTavern.llm.getEngine(modelId);
const model = engine.getCurrentModel();
// model = {id: 'gemma-2-2b-it-q4f16_1-MLC', vram_required: 1895.3, context_size: 4096}
```

### Load model

The model will be loaded the first time it is accessed, or when the `loadModel` method is called.

```js
const modelId = 'gemma-2-2b-it-q4f16_1-MLC';
const engine = SillyTavern.llm.getEngine();
await engine.loadModel(modelId);
```

### Count tokens

To count the number of tokens in a string with the loaded model's tokenizer, use the `countTokens` method.

**Throws an error if the model provides no tokenizer.**

```js
const text = 'Hello, world!';
const engine = SillyTavern.llm.getEngine();
const tokens = await engine.countTokens(text);
// tokens = 5
```

### Set default completion parameters

Set the parameters to be used in all subsequent requests with `setDefaultParams`. If a parameter is not set, the model default will be used.

Supported parameters:

* max_tokens: The maximum number of tokens to generate.
* temperature: Controls the randomness of the generated text.
* top_p: An alternative way to control the randomness of the generated text.
* frequency_penalty: Controls the diversity of the generated text.
* presence_penalty: Controls the diversity of the generated text.
* stop: A list of tokens where the model should stop generating text.

```js
const params = {
  max_tokens: 100,
  temperature: 0.7,
  top_p: 1,
  frequency_penalty: 0,
  presence_penalty: 0,
  stop: ['\n']
};
engine.setDefaultParams(params);
```

### Generate text

`generateChatPrompt` accepts a list of chat message objects (similar to OpenAI format) and returns a generated text.

Additionally, specify override parameters to use in this request only.

```js
const prompt = [
  {role: 'user', content: 'Hello!'}
];
const overrideParams = {
  max_tokens: 50
};
const response = await engine.generateChatPrompt(prompt, overrideParams);
// response = 'Hello! How are you?'
```

### Generate JSON

`generateJSON` accepts a list of chat message objects (similar to OpenAI format) and returns a generated JSON object. If a model fails to produce a valid JSON object, it will return null. **You must instruct the model to generate JSON in the prompt!**

Additionally, specify override parameters to use in this request only.

```js
const prompt = [
    { role:'system',content:'Generate a JSON object.' }
    { role: 'user', content: 'Describe a person with the following attributes: name, age, city.' }
];
const overrideParams = {
  temperature: 0.2
};
const response = await engine.generateJSON(prompt, overrideParams);
// response = {name: 'John', age: 25, city: 'New York'}
```

### Generate streaming text

`generateChatStream` accepts a list of chat message objects (similar to OpenAI format) and returns an AsyncGenerator that yields the text already generated. **Returns the full text, not deltas.**

Additionally, specify override parameters to use in this request only.

```js
const prompt = [
  {role: 'user', content: 'Hello!'}
];
const overrideParams = { /* ... */};
const stream = await engine.generateChatStream(prompt, overrideParams);
for await (const { text } of stream) {
  console.log(text);
}
```

### Get embedding models

To get all the available embedding models, use the `getEmbeddingModels` method.

Returns an array of objects with the following properties:

* id: The model ID.
* vram_required: The amount of VRAM required to load the model in MB.

> -b in the model name means the batch size it allows. The smaller it is, the less memory the model consumes.

```js
const models = await SillyTavern.llm.getEmbeddingModels();
// models = [{id: 'snowflake-arctic-embed-m-q0f32-MLC-b32', vram_required: 1407.51}, ...]
```

### Generate an embedding

To generate an embedding for a given text or array of texts, use the `generateEmbedding` method.

Returns an array of array of numbers, where each inner array represents the embedding of a text with the same index.

```js
const modelId = 'snowflake-arctic-embed-m-q0f32-MLC-b32';
const text = 'Hello, world!';
const engine = SillyTavern.llm.getEngine(modelId);
const embedding = await engine.generateEmbedding([text]);
// embedding = [ [0.1, 0.2, ...] ]
```

## Configuration

Default parameters can be configured in the extension settings, including the preferred model.

These parameters will be used in requests for the default engine instance unless overridden by the calling code.

### Demo mode

To open a demo playground, use the "Try it out!" button in the extension settings.

This will use the default engine instance with the default parameters.

## How to build

```sh
npm install
npm run build
```

## License

AGPL-3.0
