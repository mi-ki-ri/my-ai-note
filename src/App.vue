<script setup>
import { onMounted, ref } from "vue";

const myText = ref("");
const myAns = ref("");
const myTags = ref([]);
const your_api_key = import.meta.env.VITE_OPENAI_API_KEY;

async function fetchStream(stream) {
  const reader = stream.getReader();
  let charsReceived = 0;
  // read() returns a promise that resolves
  // when a value has been received
  const result = await reader
    .read()
    .then(function processText({ done, value }) {
      // Result objects contain two properties:
      // done  - true if the stream has already given you all its data.
      // value - some data. Always undefined when done is true.
      if (done) {
        console.log("Stream complete");
        return li.innerText;
      }
      // value for fetch streams is a Uint8Array
      charsReceived += value.length;
      const chunk = value;
      console.log(
        `Received ${charsReceived} characters so far. Current chunk = ${chunk}`
      );
      li.appendChild(document.createTextNode(chunk));
      return reader.read().then(processText);
    });
  const list = result.split(",");
  const numList = list.map((item) => {
    return parseInt(item);
  });
  const text = String.fromCharCode(...numList);
  const response = JSON.parse(text);
  return response;
}

const myChat = async () => {
  const DEFAULT_PARAMS = {
    model: "gpt-3.5-turbo",
    messages: [
      {
        role: "system",
        content:
          "あなたはアプリのアシスタントです。ユーザーが与えてくるテキストを、ポジティブ/ネガティブで判断し、ポジティブな場合は褒めて、ネガティブな場合は励ましてください。また、ポジティブ度を0~100％で表示してください。JSON形式で返答してください。",
      },
      {
        role: "user",
        content:
          "次々とツイートをつなげていく感覚を導入したメモアプリを作成しようと思う",
      },
      {
        role: "assistant",
        content:
          "{positive: 75, message: 'それはいいアイデアですね！　どんどん進めましょう！'}",
      },
      { role: "user", content: myText.value },
    ],
    max_tokens: 1028,
    temperature: 0.5,
    // frequency_penalty: 1.0,
    // stream: true,
  };
  const params_ = { ...DEFAULT_PARAMS };
  const result = await fetch("https://api.openai.com/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",

      Authorization: "Bearer " + String(your_api_key),
    },
    body: JSON.stringify(params_),
  });
  const myjson = await result.json();
  myAns.value = myjson.choices[0].message.content;
};

const myChatTagger = async () => {
  const DEFAULT_PARAMS = {
    model: "gpt-3.5-turbo",
    messages: [
      {
        role: "system",
        content:
          "あなたはアプリのアシスタントです。ユーザーが与えてくるテキストを、ユーザーが与えてくるタグに当てはめて、当てはまる度合いを返してください。JSON形式で返してください。",
      },
      {
        role: "user",
        content:
          "{text: '次々とツイートをつなげていく感覚を導入したメモアプリを作成しようと思う', tag: ['app', 'idea', 'music', 'movie', 'dev', 'book']}",
      },
      {
        role: "assistant",
        content:
          "{app: 0.75, idea: 0.66, music: 0.0, movie: 0.0, dev: 0.5, book: 0.1}",
      },
      {
        role: "user",
        content: `text: ${myText.value}, tag: ['app', 'idea', 'music', 'movie', 'dev', 'book']`,
      },
    ],
    max_tokens: 1028,
    temperature: 0.5,
    // frequency_penalty: 1.0,
    // stream: true,
  };
  const params_ = { ...DEFAULT_PARAMS };
  const result = await fetch("https://api.openai.com/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",

      Authorization: "Bearer " + String(your_api_key),
    },
    body: JSON.stringify(params_),
  });
  const myjson = await result.json();
  myTags.value = myjson.choices[0].message.content;
};
</script>

<template>
  <textarea v-model="myTags" disabled />
  <textarea v-model="myAns" disabled />
  <textarea v-model="myText" />
  <button
    @click="
      () => {
        myChat(), myChatTagger();
      }
    "
  >
    Send
  </button>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
