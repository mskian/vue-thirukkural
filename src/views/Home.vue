<template>
  <div class="container mx-auto px-4">
    <div class="flex flex-col justify-center items-center">
      <form method="GET" class="m-7 flex">
        <input
          id="match"
          name="match"
          class="w-15 text-center rounded-l-lg p-0 border-t mr-0 border-b border-l text-gray-800 border-gray-200 bg-white"
          placeholder="Enter No 1 to 1330"
          autocomplete="off"
          required
        />
        <button
          class="w-15 px-4 rounded-r-lg bg-purple-400 text-gray-800 font-bold p-4 uppercase border-purple-500 border-t border-b border-r"
        >
          Get Kural
        </button>
      </form>
      <button
        class="Gethome bg-green-400 text-black font-medium py-2 px-4 rounded-full mt-4"
        @click.prevent="getResult"
      >
        {{ loading ? "🔄 Updating Kural" : "🔄 Next Kural" }}
      </button>
      <div
        class="dark:bg-pink-200 dark:border-pink-200 bg-white rounded-2xl border shadow-xl p-10 max-w-lg mt-6"
      >
        <div class="flex flex-col items-center space-y-4">
          <h1 class="font-bold text-2xl text-gray-700 w-4/6 text-center">
            {{ loading ? "" : "Kural No : " + results.Number }}
          </h1>
          <p class="text-center text-sm text-gray-700 w-5/6 font-bold">
            {{ loading ? "" : results.Line1 + "\n" + results.Line2 }}
          </p>
          <button
            v-if="supportsCB"
            @click="copy"
            class="bg-purple-500 text-white font-bold py-2 px-4 rounded-full mt-4"
          >
            {{ loading ? "📝 Loading" : "📝 Copy Kural" }}
          </button>
          <div
            v-if="message"
            class="checkmsg p-4 mb-4 text-sm text-gray-700 bg-gray-100 rounded-lg dark:bg-gray-700 dark:text-gray-300"
            role="alert"
          >
            {{ message }}
          </div>
          <div
            class="w-full flex flex-col justify-between dark:bg-gray-800 bg-yellow-200 dark:border-gray-700 rounded-lg border border-yellow-400 mb-6 py-5 px-4"
          >
            <div>
              <h4 class="text-gray-800 dark:text-gray-100 font-bold mb-3">
                Kural Translation
              </h4>
              <p class="text-gray-800 dark:text-gray-100 text-sm">
                {{ loading ? "Loading Kural" : results.Translation }}
              </p>
            </div>
          </div>
          <div
            class="w-full flex flex-col justify-between dark:bg-gray-800 bg-red-300 dark:border-gray-700 rounded-lg border border-red-400 mb-6 py-5 px-4"
          >
            <div>
              <h4 class="text-gray-800 dark:text-gray-100 font-bold mb-3">
                Kural Meaning
              </h4>
              <p class="text-gray-800 dark:text-gray-100 text-sm">
                {{ loading ? "Loading Kural" : results.mv }}
              </p>
            </div>
          </div>
          <div
            class="w-full flex flex-col justify-between dark:bg-gray-800 bg-blue-300 dark:border-gray-700 rounded-lg border border-blue-400 mb-6 py-5 px-4"
          >
            <div>
              <h4 class="text-gray-800 dark:text-gray-100 font-bold mb-3">
                Kural Example
              </h4>
              <p class="text-gray-800 dark:text-gray-100 text-sm">
                {{ loading ? "Loading Kural" : results.mk }}
              </p>
            </div>
          </div>
          <button
            class="Getview bg-yellow-400 text-black font-medium py-2 px-4 rounded-full mt-4"
            @click.prevent="getResult"
          >
            {{ loading ? "🔄 Updating Kural" : "🔄 Next" }}
          </button>
        </div>
      </div>
      <div class="mt-6"></div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { createToast } from "mosha-vue-toastify";
import "mosha-vue-toastify/dist/style.css";

export default {
  name: "KuralHome",
  data() {
    return {
      results: {},
      loading: false,
      deferredPrompt: null,
      code: "",
      supportsCB: false,
      message: "",
    };
  },
  created() {
    if (navigator.clipboard) {
      this.supportsCB = true;
    }
  },
  mounted() {
    this.getResult();
    this.captureEvent();
  },
  methods: {
    getResult() {
      this.loading = true;
      if (this.$route.query.match) {
        axios
          .get("https://thirukkural.up.railway.app/" + this.$route.query.match)
          .then((response) => {
            this.results = response.data;
            this.loading = false;
            this.code = response.data.Line1 + "\n" + response.data.Line2;
            document.querySelector(".Getview").style.display = "none";
            document.querySelector(".Gethome").style.display = "none";
          });
      } else {
        axios.get("https://thirukkural.up.railway.app/").then((response) => {
          this.results = response.data;
          this.loading = false;
          this.code = response.data.Line1 + "\n" + response.data.Line2;
        });
      }
    },
    copy() {
      navigator.clipboard
        .writeText(this.code)
        .then(() => {
          this.message = "😀 Kural Copied.";
          setTimeout(() => {
            document.querySelector(".checkmsg").style.display = "none";
          }, 900);
          createToast(" Kural Copied", {
            timeout: 1000,
            type: "info",
            position: "bottom-right",
            transition: "zoom",
            hideProgressBar: "true",
            showIcon: "true",
          });
        })
        .catch((e) => {
          // eslint-disable-next-line
          console.error(e);
          this.message = "😐 Sorry, unable to copy.";
          setTimeout(() => {
            document.querySelector(".checkmsg").style.display = "none";
          }, 900);
          createToast("Sorry, unable to copy", {
            timeout: 1000,
            type: "danger",
            position: "bottom-right",
            transition: "zoom",
            hideProgressBar: "true",
            showIcon: "true",
          });
        });
    },
    captureEvent() {
      window.addEventListener("beforeinstallprompt", (e) => {
        e.preventDefault();
        this.deferredPrompt = e;
      });
    },
    clickCallback() {
      this.deferredPrompt.prompt();
      this.deferredPrompt.userChoice.then((choiceResult) => {
        if (choiceResult.outcome === "accepted") {
          //do something
        }
        this.deferredPrompt = null;
      });
    },
  },
};
</script>
