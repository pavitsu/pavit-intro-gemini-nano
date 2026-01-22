# Introduction to Gemini Nano

## Enabled Experimental Features

All of the built-in AI APIs are available on **localhost** in Chrome. Set the following flags to **Enabled**:

* `chrome://flags#prompt-api-for-gemini-nano`
* `chrome://flags/#prompt-api-for-gemini-nano-multimodal-input`
* `chrome://flags/#optimization-guide-on-device-model`

Then click **Relaunch** or restart Chrome. If you encounter errors, troubleshoot localhost.

---

## Download Model

1. **Check Availability**: To determine if the model is ready to use, call `LanguageModel.availability()`. ([The Prompt API](https://developer.chrome.com/docs/ai/prompt-api#add_expected_input_and_output) requires declaring language support).

    ```javascript
    const availability = await LanguageModel.availability({languages: ["en"]});
    console.log(availability); // Returns "readily", "after-download", or "no"
    ```

2. **Initialize and Monitor**: Initialize the session and monitor the download progress.

    ```javascript
    const session = await LanguageModel.create({
      monitor(m) {
        m.addEventListener('downloadprogress', (e) => {
          console.log(`Downloaded ${e.loaded * 100}%`);
        });
      },
    });
    ```

3. **Verify Status**: Check if the [model status](https://developer.chrome.com/docs/ai/get-started#model_download) has changed to available.

    ```javascript
    // You must re-check availability to see the updated status
    const newAvailability = await LanguageModel.availability({languages: ["en"]});
    console.log(newAvailability);
    ```

## Text

1. **Create Session**:

    ```javascript
    const session = await LanguageModel.create({
      expectedInputs: [
        { type: "text", languages: ["en"] }
      ],
      expectedOutputs: [
        { type: "text", languages: ["en"] }
      ],
      initialPrompts: [
        { role: 'system', content: 'You are a helpful and friendly assistant.' },
        { role: 'user', content: 'What is the capital of Italy?' },
        { role: 'assistant', content: 'The capital of Italy is Rome.' },
        { role: 'user', content: 'What language is spoken there?' },
        { role: 'assistant', content: 'The official language of Italy is Italian.' },
      ]
    });
    ```

2. **Send Prompt (Standard)**: You can send a prompt and await the full response.

    ```javascript
    // Set up abort controller
    const controller = new AbortController();
    // Call controller.abort() to interrupt the session.

    try {
      // Note: usage of 'await' is required here
      const response = await session.prompt('Write me a long poem!', {
        signal: controller.signal,
      });
      console.log(response);
    } catch (err) {
      if (err.name === 'AbortError') {
        console.log('User Abort!');
      } else {
        console.error(err.name, err.message);
      }
    }
    ```

3. **Send Prompt (Streaming)**: Alternatively, handle the return as a streaming chunk.

    ```javascript
    const controller = new AbortController();

    try {
      const stream = session.promptStreaming('Write me a long poem!', {
        signal: controller.signal,
      });
      for await (const chunk of stream) {
        console.log(chunk);
      }
    } catch (err) {
      if (err.name === 'AbortError') {
        console.log('User Abort!');
      } else {
        console.error(err.name, err.message);
      }
    }
    ```

## Image

1. **Create Session**:

    ```javascript
    const session = await LanguageModel.create({
      expectedInputs: [
        { type: "text", languages: ["en"] },
        { type: "image" },
      ],
      expectedOutputs: [{ type: "text", languages: ["en"] }],
    });
    ```

2. **Load Image**:

    ```javascript
    // Fetch the image
    const imageUrl = 'https://upload.wikimedia.org/wikipedia/commons/4/43/Bonnet_macaque_%28Macaca_radiata%29_Photograph_By_Shantanu_Kuveskar.jpg';
    const imageResponse = await fetch(imageUrl);
    const blob = await imageResponse.blob();
    ```

3. **Send Prompt**:

    ```javascript
    const response = await session.prompt([
      {
        role: "user",
        content: [
          {
            type: "text",
            value: "Describe what you see in the image:",
          },
          { type: "image", value: blob },
        ],
      },
    ]);
    console.log(response);
    ```

4. **Continue Conversation**: You can continue prompting on the same session.

    ```javascript
    const followUpResponse = await session.prompt([
      {
        role: "user",
        content: [
          {
            type: "text",
            value: "What is the species of animal?",
          },
        ],
      },
    ]);
    console.log(followUpResponse);
    ```

5. **Check Quota**: Check how much context window quota is left.

    ```javascript
    console.log(`${session.inputUsage}/${session.inputQuota}`);
    ```

## Audio

1. **Create Session**:

    ```javascript
    const session = await LanguageModel.create({
      expectedInputs: [
        { type: "text", languages: ["en"] },
        { type: "audio" },
      ],
      expectedOutputs: [{ type: "text", languages: ["en"] }],
    });
    ```

2. **Fetch Audio**:

    ```javascript
    const audioUrl = 'https://github.com/voxserv/audio_quality_testing_samples/raw/refs/heads/master/mono_44100/127389__acclivity__thetimehascome.wav';
    const response = await fetch(audioUrl);
    const audioBlob = await response.blob();
    ```

3. **Send Prompt**:

    ```javascript
    const result = await session.prompt([
      {
        role: "user",
        content: [
          {
            type: "text",
            value: "Transcribe this audio file:",
          },
          { type: "audio", value: audioBlob },
        ],
      },
    ]);
    console.log(result);
    ```
