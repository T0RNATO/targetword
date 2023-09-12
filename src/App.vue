<template>
<div class='main' @click='selected = []'>
    <h1>Target Word</h1>
    <div class='game'>
        <div class='board-container'>
            <h2>{{selected.map(i => scrambled[i]).join("")}}</h2>
            <div class='board'>
                <Square v-for='(letter, i) in scrambled'
                        :key='letter' :letter='letter' :i='i'
                        @click.stop='squareClick(letter, i)'
                        :class='{selected: selected.includes(i)}'
                />
            </div>
            <div class='buttons'>
                <button id='check' @click='check'>CHECK WORD</button>
                <button>⌫</button>
            </div>
        </div>
        <div class='desc'>
            <p>
                Find as many words as you can using the letters.
                <br>
                Words must be >3 letters and include the center letter.
                <br>
                You can only use each letter once per word.
            </p>
            <div class='found'>
                {{foundWords.join(", ")}}
            </div>
            <div class='message'>{{message}}</div>
        </div>
    </div>
</div>
<!--    <div v-for='word in random' @click='out.push(word)' :key='word'>{{word}}</div>-->
<!--    <button @click='console.log(out)'>hi</button>-->
</template>

<script setup>
import Square from '@/components/Square.vue';
import allValidWords from 'an-array-of-english-words';
import nineLetterWords from '@/assets/validNineLetters.json';
import { ref } from 'vue'

const index = Math.round(Math.random() * nineLetterWords.length);
const target = nineLetterWords[index].toUpperCase();

// TEMP
// const random = allValidWords.filter(word => word.length === 9 && Math.random() > 0.995);
// const out = ref([]);

console.log(target);

const scrambled = target.split('').sort(() => 0.5-Math.random()).join('');

const foundWords = ref([]);

const selected = ref([]);
const message = ref("");

function squareClick(letter, i) {
    if (!selected.value.includes(i)) {
        selected.value.push(i);
    }
}

document.addEventListener("keydown", key);

function key(ev) {
    if (ev.ctrlKey || ev.altKey || ev.metaKey) return;
    if (ev.key === "Enter") {
        check();
    } else if (ev.key === "Backspace") {
        selected.value.pop();
    } else if (ev.key === "Escape") {
        selected.value = [];
    } else {
        const unusedLetters = []
        for (const [i, letter] of scrambled.split('').entries()) {
            unusedLetters.push(selected.value.includes(i) ? "" : letter)
        }
        const index = unusedLetters.indexOf(ev.key.toUpperCase());
        if (index >= 0) {
            squareClick(ev.key, index);
        }
    }
}

let messageTimeout;

function setMessage(msg) {
    message.value = msg;
    clearTimeout(messageTimeout);
    messageTimeout = setTimeout(() => {
        message.value = "";
    }, 1500)
}

function check() {
    let failed = false;
    if (selected.value.length < 4) {
        setMessage("Must be 4 letters or more.");
        failed = true;
    }
    if (!selected.value.includes(4)) {
        setMessage("Must include center letter.");
        failed = true;
    }
    const word = selected.value.map(i => scrambled[i]).join("").toLowerCase();
    if (!allValidWords.includes(word)) {
        setMessage("Not a valid English word.");
        failed = true;
    }
    if (foundWords.value.includes(word)) {
        setMessage("Duplicate word.");
        failed = true;
    }
    if (failed) {
        selected.value = [];
        return;
    }
    selected.value = [];
    if (word.length === 9) {
        foundWords.value.push(word.toUpperCase());
        return;
    }
    foundWords.value.push(word);
}
</script>

<style>
.main {
    display: flex;
    width: 100%;
    height: 100vh;
    justify-content: center;
    align-items: center;
    flex-direction: column;
}
.game {
    display: flex;
    gap: 30px;
}
.found {
    background-image: repeating-linear-gradient(transparent 0 28px, rgb(255 255 255 / 0.2) 28px 30px);
    width: 100%;
    line-height: 30px;
    font-size: 20px;
    height: 40vh;
    margin-top: 30px;
}
.desc {
    width: 55vh;
}
p {
    margin: 0;
    font-size: 16px;
    line-height: 30px;
}
.board {
    aspect-ratio: 1/1;
    height: 50vh;
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
}
.board-container {
    display: flex;
    justify-content: center;
    flex-direction: column;
}
h1 {
    margin: 0;
}
button {
    margin: 20px 5px 0 5px;
    background: var(--bg);
    border: 4px solid white;
    padding: 10px;
}
#check {
    width: 80%;
}
.message {
    height: 15px;
    width: 100%;
    text-align: center;
    color: #de6262;
}
h2 {
    text-align: center;
    font-size: 40px;
    height: 60px;
    margin: 0 0 10px 0;
    background: rgb(255 255 255 / 0.1);
    border-radius: 5px;
}
.buttons {
    display: flex;
    justify-content: center;
}
</style>
