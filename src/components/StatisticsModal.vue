<template>
    <div class="statistics-container">
        <div class="statistics-container-inner">
            <h3>Statistics</h3>
            <table style="width: 100%">
                <thead>
                    <td>Current<br />Streak</td>
                    <td>Longest<br />Streak</td>
                    <td>Games<br />Played</td>
                    <td>Games<br />Won</td>
                    <td>Win<br />Percent</td>
                </thead>
                <tr>
                    <td>{{ stats.currentStreak }}</td>
                    <td>{{ stats.maxStreak }}</td>
                    <td>{{ stats.gamesPlayed }}</td>
                    <td>{{ stats.gamesWon }}</td>
                    <td>{{ stats.winPercentage }}</td>
                </tr>
            </table>
            <div class="guess-distribution">
                <h4>GUESS DISTRIBUTION</h4>
                <div class="guess-distribution-line">
                    <div>1:</div>
                    <div
                        class="guess-bar"
                        :class="{
                            'guess-bar-highlight': 1 === highlightedGuess,
                        }"
                        :style="{ width: guessPercents[1] + '%' }"
                    />
                    <div>{{ stats.guesses[1] }}</div>
                </div>
                <div class="guess-distribution-line">
                    <div>2:</div>
                    <div
                        class="guess-bar"
                        :class="{
                            'guess-bar-highlight': 2 === highlightedGuess,
                        }"
                        :style="{ width: guessPercents[2] + '%' }"
                    />
                    <div>{{ stats.guesses[2] }}</div>
                </div>
                <div class="guess-distribution-line">
                    <div>3:</div>
                    <div
                        class="guess-bar"
                        :class="{
                            'guess-bar-highlight': 3 === highlightedGuess,
                        }"
                        :style="{ width: guessPercents[3] + '%' }"
                    />
                    <div>{{ stats.guesses[3] }}</div>
                </div>
                <div class="guess-distribution-line">
                    <div>4:</div>
                    <div
                        class="guess-bar"
                        :class="{
                            'guess-bar-highlight': 4 === highlightedGuess,
                        }"
                        :style="{ width: guessPercents[4] + '%' }"
                    />
                    <div>{{ stats.guesses[4] }}</div>
                </div>
                <div class="guess-distribution-line">
                    <div>5:</div>
                    <div
                        class="guess-bar"
                        :class="{
                            'guess-bar-highlight': 5 === highlightedGuess,
                        }"
                        :style="{ width: guessPercents[5] + '%' }"
                    />
                    <div>{{ stats.guesses[5] }}</div>
                </div>
                <div class="guess-distribution-line">
                    <div>6:</div>
                    <div
                        class="guess-bar"
                        :class="{
                            'guess-bar-highlight': 6 === highlightedGuess,
                        }"
                        :style="{ width: guessPercents[6] + '%' }"
                    />
                    <div>{{ stats.guesses[6] }}</div>
                </div>
            </div>
            <div v-if="shareButtonEnabled">
                <div class="share-btn" @click="share">SHARE</div>
            </div>
            <div class="x-button" @click="$emit('close')">X</div>
        </div>
    </div>
</template>

<script>
import { useToast } from "vue-toastification";
import { getItem, setItem } from "../SaveDataManager";
import { getDistance } from "../qwerty";
export default {
    mounted() {
        const existingStats = getItem("stats");
        if (!existingStats) {
            setItem("stats", this.stats);
        } else {
            this.stats = existingStats;
        }
    },
    computed: {
        guessPercents() {
            if (!this.stats) {
                return {};
            }
            const ret = {};
            const maxGuesses = Math.max(...Object.values(this.stats.guesses));
            for (let i = 1; i <= 6; i++) {
                ret[i] = Math.max(
                    (100 * this.stats.guesses[i]) / maxGuesses,
                    2
                );
            }
            return ret;
        },
        highlightedGuess() {
            if (this.$parent.$parent.$refs.gameMain.gameState === "WIN") {
                return getItem("guesses").length;
            }
            return -1;
        },
        shareButtonEnabled() {
            return (
                this.$parent.$parent.$refs.gameMain.gameState !== "IN_PROGRESS"
            );
        },
    },
    data() {
        return {
            stats: {
                guesses: { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0, 6: 0, fail: 0 },
                currentStreak: 0,
                gamesPlayed: 0,
                gamesWon: 0,
                maxStreak: 0,
                winPercentage: 0,
            },
        };
    },
    methods: {
        share() {
            //
            const guesses = getItem("guesses");
            const maxDistance = getDistance("Q", "P");
            const str = guesses
                .map((g) => {
                    return g
                        .map((l) => {
                            if (l.distance === 0) {
                                return "🟩";
                            }
                            if (l.distance / maxDistance < 0.25) {
                                return "🟨";
                            }
                            return "⬛";
                        })
                        .join("");
                })
                .join("\n");
            const today = new Date();
            const qwertleEpoch = new Date("04/14/2022");
            const todaysNumber = Math.floor(
                (today.getTime() - qwertleEpoch.getTime()) / (1000 * 3600 * 24)
            );
            const todaysGuesses =
                this.$parent.$parent.$refs.gameMain.gameState === "WIN"
                    ? guesses.length
                    : "X";
            const toast = useToast();
            try {
                navigator.clipboard
                    .writeText(`QWERTLE #${todaysNumber} ${todaysGuesses}/6
${str}`);
                toast.success("Copied to clipboard", { timeout: 2500 });
            } catch (error) {
                toast.error("Error copying to clipboard", { timeout: 2500 });
            }
        },
        exportStats() {
            const toast = useToast();
                        try {

                navigator.clipboard
                    .writeText(JSON.stringify(this.stats));
                toast.success("Copied to clipboard", { timeout: 2500 });
            } catch (error) {
                toast.error("Error copying to clipboard", { timeout: 2500 });
            }
        },
        validateImport(imported) {
            const checkKeys = (source,test) => {
                for (const k in source) {
                    if (typeof source[k] == "object") {
                        if (typeof test[k] !== "object" || !checkKeys(source[k],test[k])) {
                            return false
                        }
                    }
                    if (typeof source[k] !== typeof test[k]) {
                        return false
                    }
                }
                return true

            }
            const expected = JSON.parse(JSON.stringify(this.stats))
            console.log(expected)
            return checkKeys(expected,imported) && checkKeys(imported,expected)
        },
        async importStats() {
            const toast = useToast();
                                    try {

                                            const imported = JSON.parse(prompt("Paste stats to import"))
                                        

                                        if (this.validateImport(imported)) {
                                            this.stats = imported
                                            setItem("stats",this.stats)
                                            toast.success("Successfully imported stats", { timeout: 2500 });
                                        } else {

                                            toast.error("Error copying to clipboard", { timeout: 2500 });
                                        }

            } catch (error) {
                console.error(error)
                toast.error("Error copying to clipboard", { timeout: 2500 });
            }
        }
    },
};
</script>

<style>
.guess-bar {
    height: 20px;
    background-color: gray;
    display: inline-block;
    margin: 8px;
}
.guess-bar-highlight {
    background-color: forestgreen;
}
.guess-distribution {
    padding: 5px;
    width: 75%;
    margin: auto;
}
.guess-distribution-line {
    display: flex;
    align-items: center;
}
.statistics-container-inner {
    position: relative;
    width: 90%;
    max-width: 500px;
    height: 80%;
    max-height: 750px;
    background-color: #121213;
        overflow-y: auto;

}
.statistics-container {
    position: fixed;
    padding: 5vw;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    color: white;
    display: flex;
    justify-content: center;
    align-items: center;
}
.x-button {
    position: absolute;
    top: 15px;
    right: 15px;
    width: 25px;
    height: 25px;
}
.statistics-container tr td {
    font-size: 32px;
}
.statistics-container thead td {
    font-size: 14px;
}
.share-btn {
    user-select: none;
    border-radius: 5px;
    height: 60px;
    color: white;
    font-weight: bold;
    background-color: forestgreen;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 50px;

}
</style>
