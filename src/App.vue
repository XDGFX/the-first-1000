<template>
    <div
        id="app"
        class="selection:bg-cyan-500 selection:text-white h-screen overflow-y-scroll bg-slate-100"
    >
        <IntroPage v-if="settings.firstLoad" />
        <LanguageSelectPage
            v-if="settings.language === null"
            @set-language="setLanguage($event)"
        />
        <!-- Debug mode window hover -->
        <div
            v-if="debugMode"
            class="absolute top-0 left-0 max-h-screen overflow-y-scroll bg-white text-black p-4 border-2 border-slate-200 w-96 opacity-20 hover:opacity-100 text-xs z-[9999]"
        >
            <code>
                <b>Ctrl + Alt + D to toggle debug mode</b> <br />

                <br />

                Search: {{ search }} <br />
                Hide translations: {{ hideTranslations }} <br />
                Quiz mode: {{ quizMode }} <br />

                <br />

                Search results:
                <pre v-if="search !== ''">{{
                    JSON.stringify(
                        wordsFiltered.map((word) => word.word),
                        null,
                        4
                    )
                }}</pre>

                <br />

                Settings: <br />
                <pre>{{ JSON.stringify(settings, null, 4) }}</pre>
            </code>
        </div>

        <section
            v-if="settings.language !== null"
            class="flex flex-col items-center h-screen bg-slate-100 text-slate-900 md:px-4 md:py-8 pb-16"
        >
            <!-- Desktop menu -->
            <div
                class="w-full lg:w-[990px] hidden md:flex flex-wrap items-center"
            >
                <h1 class="text-4xl flex-1">
                    <img
                        src="@/assets/logo.svg"
                        alt="LingoBolt logo"
                        class="h-12"
                    />
                </h1>

                <div v-if="!quizMode" class="flex flex-2 gap-4 items-center">
                    <!-- Search box -->
                    <input
                        v-model="search"
                        class="h-12 shrink rounded-full border-2 border-slate-200 p-4 text-2xl focus:border-rose-500 outline-none"
                        type="text"
                        placeholder="Search"
                    />

                    <!-- Hide translations -->
                    <button
                        class="h-12 w-12 flex justify-center items-center rounded-full border-2 border-slate-200 p-4 text-2xl bg-white hover:bg-slate-100 outline-none"
                        @click="hideTranslations = !hideTranslations"
                        title="Hide translations"
                    >
                        <span>🔖</span>
                    </button>

                    <span
                        class="text-slate-300 text-4xl select-none -translate-y-1"
                        >·</span
                    >

                    <!-- Quiz button -->
                    <button
                        class="h-12 w-12 flex justify-center items-center rounded-full border-2 border-slate-200 text-slate-500 p-4 text-2xl bg-white hover:bg-slate-100 outline-none"
                        @click="quizMode = true"
                        title="Practice"
                    >
                        <span class="translate-y-px"> 🎓 </span>

                        <!-- Notification circle -->
                        <div
                            v-if="settings.firstLoad"
                            class="absolute h-4 w-4 translate-x-full -translate-y-full"
                        >
                            <div
                                class="h-4 w-4 rounded-full bg-cyan-400 animate-ping"
                            ></div>
                            <div
                                class="h-4 w-4 rounded-full bg-cyan-400 -translate-y-full"
                            ></div>
                        </div>
                    </button>

                    <!-- Language select -->
                    <button
                        class="h-12 w-12 flex justify-center items-center rounded-full border-2 border-slate-200 p-4 text-2xl bg-white hover:bg-slate-100 outline-none"
                        @click="settings.language = null"
                        title="Change language"
                    >
                        <span class="translate-y-px">🌐</span>
                    </button>
                </div>

                <div v-if="quizMode" class="flex flex-2 gap-4 items-center">
                    <!-- Home button -->
                    <button
                        v-if="quizMode"
                        class="h-12 w-12 flex justify-center items-center rounded-full border-2 border-slate-200 text-slate-500 p-4 text-2xl bg-white hover:bg-slate-100 outline-none"
                        @click="quizMode = false"
                        title="Home"
                    >
                        <span class="translate-y-px"> 🏠 </span>
                    </button>
                </div>
            </div>

            <carousel
                v-if="!quizMode"
                :words="wordsFiltered"
                :wordScores="settings.stats[settings.language].wordScores"
                :hide-translations="hideTranslations"
            ></carousel>

            <!-- Quiz -->
            <quiz
                v-if="quizMode"
                :words="wordsFiltered"
                :stats="settings.stats[settings.language]"
                :language="settings.language"
                @test-result="onTestResult"
            ></quiz>

            <!-- Mobile menu -->
            <div
                class="fixed bottom-0 w-full z-[999] md:hidden flex justify-around bg-slate-200 p-2 border-t-2 rounded-t-[24px] border-slate-300"
            >
                <div
                    v-if="!quizMode"
                    class="flex gap-4 items-center justify-between w-full"
                >
                    <!-- Search box -->
                    <input
                        v-model="search"
                        class="h-12 w-12 shrink focus:w-full peer transition-all flex justify-center p-4 rounded-full border-2 border-slate-200 outline-none text-2xl bg-white"
                        type="text"
                    />

                    <!-- Div which overlays the previous input -->
                    <div
                        v-if="search === ''"
                        class="h-12 w-12 peer-focus:hidden pointer-events-none absolute flex justify-center items-center rounded-full border-2 border-slate-200 text-2xl bg-white"
                    >
                        <span>🔎</span>
                    </div>

                    <!-- Clear search button -->
                    <button
                        v-else
                        class="h-12 w-12 peer-focus:hidden absolute flex justify-center items-center rounded-full border-2 border-slate-200 text-2xl bg-white"
                        @click="search = ''"
                    >
                        <span>❌</span>
                    </button>

                    <!-- Hide translations -->
                    <button
                        class="h-12 w-12 peer-focus:hidden flex justify-center items-center rounded-full border-2 border-slate-200 text-2xl bg-white"
                        @click="hideTranslations = !hideTranslations"
                        title="Hide translations"
                    >
                        🔖
                    </button>

                    <!-- Quiz button -->
                    <button
                        class="h-12 w-12 peer-focus:hidden flex justify-center items-center rounded-full border-2 border-slate-200 text-2xl bg-white"
                        @click="quizMode = true"
                        title="Practice"
                    >
                        <span class="translate-y-px"> 🎓 </span>

                        <!-- Notification circle -->
                        <div
                            v-if="settings.firstLoad"
                            class="absolute h-4 w-4 translate-x-full -translate-y-full"
                        >
                            <div
                                class="h-4 w-4 rounded-full bg-cyan-400 animate-ping"
                            ></div>
                            <div
                                class="h-4 w-4 rounded-full bg-cyan-400 -translate-y-full"
                            ></div>
                        </div>
                    </button>

                    <!-- Language select -->
                    <button
                        class="h-12 w-12 peer-focus:hidden flex justify-center items-center rounded-full border-2 border-slate-200 text-2xl bg-white"
                        @click="settings.language = null"
                        title="Change language"
                    >
                        <span class="translate-y-px">🌐</span>
                    </button>
                </div>

                <div v-if="quizMode" class="flex flex-2 gap-4 items-center">
                    <!-- Home button -->
                    <button
                        v-if="quizMode"
                        class="h-12 w-12 flex justify-center items-center rounded-full border-2 border-slate-200 text-2xl bg-white"
                        @click="quizMode = false"
                        title="Home"
                    >
                        <span class="translate-y-px">🏠</span>
                    </button>
                </div>
            </div>

            <!-- Footer -->
            <div
                class="hidden md:flex flex-col flex-1 items-center place-content-end pt-8"
            >
                <div
                    class="text-xs md:text-sm text-slate-500 flex gap-2 text-center items-center"
                >
                    <div>
                        Made with ❤️ by

                        <a
                            class="text-slate-900 underline decoration-rose-500"
                            href="https://github.com/xdgfx"
                            target="_blank"
                        >
                            xdgfx</a
                        >
                    </div>
                    ·
                    <div>
                        <a
                            class="text-slate-900"
                            href="https://ko-fi.com/callumm"
                            target="_blank"
                        >
                            Buy me a coffee ☕</a
                        >
                    </div>
                    ·
                    <div>
                        Notice a mistake?
                        <a
                            class="text-slate-900 underline decoration-rose-500"
                            href="https://github.com/XDGFX/LingoBolt/issues/new"
                            target="_blank"
                        >
                            Let me know!</a
                        >
                    </div>
                </div>
            </div>
        </section>

        <!-- Spacer for mobile menu -->
        <!-- <div
            class="bottom-0 shrink-0 w-full pointer-events-none md:hidden m-2 h-12"
        ></div> -->
    </div>
</template>

<script>
import { computed } from "vue";

import Fuse from "fuse.js";

import IntroPage from "@/components/IntroPage.vue";
import LanguageSelectPage from "@/components/LanguageSelectPage.vue";
import Carousel from "@/components/Carousel.vue";
import Quiz from "@/components/Quiz.vue";

// Default settings object
const defaultSettings = {
    firstLoad: true, // If this is the first time the user has loaded the page
    language: null, // The language the user is learning
    stats: {}, // The user's stats for each language
    hideTranslations: false,
};

// Stats object used for initialising new languages
const defaultStats = {
    elo: 500, // -2 for each wrong answer, +2 for each correct answer
    score: 0, // +10 for each correct answer
    count: 0, // Number of words the user has been tested on
    wordScores: null,
};

let settings = JSON.parse(localStorage.getItem("settings"));

// If settings is defined, set firstLoad to false
if (settings) {
    settings.firstLoad = false;
} else {
    settings = defaultSettings;
}

// Make sure all the requred keys are present
for (const key in defaultSettings) {
    if (!(key in settings)) {
        settings[key] = defaultSettings[key];
    }
}

export default {
    name: "App",
    components: {
        IntroPage,
        LanguageSelectPage,
        Carousel,
        Quiz,
    },
    data() {
        return {
            debugMode: false,

            words: [], // The wordlist for the current language

            search: "",
            hideTranslations: false,
            quizMode: false,

            // User settings
            settings,

            // Languages
            languages: ["fr", "es", "it", "de", "pt", "nl", "lb", "cs", "no"],
            newLanguages: ["lb", "cs", "no"],
            flagOverrides: {
                lb: "lu",
                cs: "cz",
            },
        };
    },
    provide() {
        return {
            debugMode: computed(() => this.debugMode),
            languages: this.languages,
            newLanguages: this.newLanguages,
            flagOverrides: this.flagOverrides,
        };
    },
    created() {
        this.loadWords();
    },
    computed: {
        wordsFiltered() {
            // If language is not set, return an empty array
            if (!this.settings.language) {
                return [];
            }

            // When not in quiz mode, words are filtered by the search box
            if (!this.quizMode) {
                this.words.sort((a, b) => {
                    // Sort the words based on their score in the wordScores
                    // object, and then based on difficulty (easy first)
                    const aScore =
                        this.settings.stats[this.settings.language].wordScores[
                            a.word
                        ] || 10;
                    const bScore =
                        this.settings.stats[this.settings.language].wordScores[
                            b.word
                        ] || 10;

                    if (aScore === bScore) {
                        return a.difficulty - b.difficulty;
                    }

                    return aScore - bScore;
                });

                // If the search box is empty, return all words
                if (this.search === "") {
                    return this.words;
                }

                // Otherwise, return the words that match the search query
                const fuse = new Fuse(this.words, {
                    keys: [
                        { name: "word", weight: 6 },
                        { name: "translation", weight: 6 },
                        "example",
                        "example_en",
                        { name: "tags", weight: 2 },
                        { name: "synonyms", weight: 3 },
                    ],
                    shouldSort: true,
                    ignoreLocation: true,
                });
                const results = fuse.search(
                    this.search.replace(/\s+/g, " ").toLowerCase()
                );

                return results.map((result) => result.item);
            } else {
                // When in quiz mode, words are filtered by your elo, and the
                // difficulty of the word.
                //
                // Difficulty Level | Elo
                // -------------------------
                // 1                | 0 - 1000
                // 1, 2             | 1000 - 1500
                // 1, 2, 3          | 1500 - 2000
                // all              | 2000+
                return this.words.filter((word) => {
                    if (
                        this.settings.stats[this.settings.language].elo < 1000
                    ) {
                        return word.difficulty === 1;
                    } else if (
                        this.settings.stats[this.settings.language].elo < 1500
                    ) {
                        return word.difficulty <= 2;
                    } else if (
                        this.settings.stats[this.settings.language].elo < 2000
                    ) {
                        return word.difficulty <= 3;
                    } else {
                        return true;
                    }
                });
            }
        },
    },

    methods: {
        // When the quiz is finished, update the elo and wordScores
        onTestResult(word, correct) {
            const stats = this.settings.stats[this.settings.language];

            stats.count++;
            stats.wordScores[word.word] = stats.wordScores[word.word] || 5;

            // Adjust the score based on whether the user got the answer right or wrong
            const adjustment = correct ? 1 : -1;
            stats.wordScores[word.word] = Math.min(
                Math.max(stats.wordScores[word.word] + adjustment, 0),
                10
            );

            // Update the elo
            stats.elo = Math.min(
                Math.max(stats.elo + (correct ? 2 : -2), 0),
                2000
            );

            // Update the score
            stats.score += correct ? 10 : 0;
            stats.score = Math.min(stats.score, 20000);
        },

        setLanguage(language) {
            this.settings.language = language;
            this.settings.stats[language] = this.settings.stats[language] || {
                ...defaultStats,
            };

            // Do this separately to avoid referencing the same object after
            // switching languages (resulting in the same wordScores object)
            this.settings.stats[language].wordScores =
                this.settings.stats[language].wordScores || {};

            this.loadWords();
        },

        async loadWords() {
            if (this.settings.language === null) {
                return;
            }

            // Load the words
            const words = await import(
                `@/languages/${this.settings.language}.json`
            );

            // Set the words
            this.words = words.default;
        },
    },

    watch: {
        settings: {
            handler(newValue, oldValue) {
                localStorage.setItem("settings", JSON.stringify(this.settings));
            },
            deep: true,
        },
    },

    mounted() {
        // Add a listener to trigger debug mode with extra info
        document.addEventListener("keydown", (event) => {
            if (event.key === "d" && event.ctrlKey && event.altKey) {
                this.debugMode = !this.debugMode;
            }
        });
    },
};
</script>

<style></style>
