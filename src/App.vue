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
                <button @click.stop='selected.pop()'>⌫</button>
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
            <button id='reveal' @click='reveal'>REVEAL</button>
            <div class='message'>{{message}}</div>
        </div>
    </div>
    <dialog ref='revealDialog' @click='revealDialog.close()'>
        <button class='close'>×</button>
        <div @click.stop id='dialog'>
            <h1>Words:</h1>
            <div class='reveal'>
                <span v-for='answer in reasonableAnswers' :key='answer' :class='{green: foundWords.includes(answer)}'>
                    {{answer}},
                </span>
            </div>
        </div>
    </dialog>
    <dialog ref='credits' @click='credits.close()'>

    </dialog>
</div>

<!--TODO: ADD CREDITS DIALOG-->
<!--    Custom wordlist generated from http://app.aspell.net/create using SCOWL-->

<!--    Copyright 2000-2019 by Kevin Atkinson-->

<!--    Permission to use, copy, modify, distribute and sell these word-->
<!--    lists, the associated scripts, the output created from the scripts,-->
<!--    and its documentation for any purpose is hereby granted without fee,-->
<!--    provided that the above copyright notice appears in all copies and-->
<!--    that both that copyright notice and this permission notice appear in-->
<!--    supporting documentation. Kevin Atkinson makes no representations-->
<!--    about the suitability of this array for any purpose. It is provided-->
<!--    "as is" without express or implied warranty.-->

<!--    Copyright (c) J Ross Beresford 1993-1999. All Rights Reserved.-->

<!--    The following restriction is placed on the use of this publication:-->
<!--    if The UK Advanced Cryptics Dictionary is used in a software package-->
<!--    or redistributed in any form, the copyright notice must be-->
<!--    prominently displayed and the text of this document must be included-->
<!--    verbatim.-->

<!--    CREDIT ENGLISH WORDS LIST NPM LIB-->


    <!--    <div v-for='word in random' @click='out.push(word)' :key='word'>{{word}}</div>-->
<!--    <button @click='console.log(out)'>hi</button>-->
</template>

<script setup>
import Square from '@/components/Square.vue';
import allValidWords from '@/assets/allValidWords.json';
import reasonableWords from '@/assets/reasonableWords.json';
import Trie from 'trie-prefix-tree';
import nineLetterWords from '@/assets/validNineLetters.json';
import { ref } from 'vue'

const trie = Trie(reasonableWords);

const revealDialog = ref(null);
const credits = ref(null);

const index = Math.round(Math.random() * nineLetterWords.length);
const target = nineLetterWords[index].toUpperCase();
const scrambled = target.split('').sort(() => 0.5-Math.random()).join('');

const reasonableAnswers = trie.getSubAnagrams(target.toLowerCase()).filter(
    word => word.includes(scrambled[4].toLowerCase())
).map(
    word => word.length === 9 ? word.toUpperCase() : word
);

if (!reasonableAnswers.includes(target)) {
    reasonableAnswers.push(target);
}

// TEMP
// const random = allValidWords.filter(word => word.length === 9 && Math.random() > 0.995);
// const out = ref([]);

console.log(target);

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
        if (selected.value.length) {
            check();
        }
    } else if (ev.key === "Backspace") {
        selected.value.pop();
    } else if (ev.key === "Escape") {
        selected.value = [];
    } else {
        const unusedLetters = []
        for (const [i, letter] of scrambled.split('').entries()) {
            unusedLetters.push(selected.value.includes(i) ? "" : letter)
        }
        // In the case of duplicate letters where one of them is the center letter,
        // use the center letter to ensure validity
        if (unusedLetters[4] === ev.key.toUpperCase()) {
            return squareClick(ev.key, 4);
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
    const sel = selected.value;
    selected.value = [];
    if (sel.length < 4) {
        setMessage("Must be 4 letters or more."); return;
    }
    if (!sel.includes(4)) {
        setMessage("Must include center letter."); return;
    }
    const word = sel.map(i => scrambled[i]).join("").toLowerCase();
    if (!allValidWords.includes(word)) {
        setMessage("Not a valid English word."); return;
    }
    if (foundWords.value.includes(word)) {
        setMessage("Duplicate word."); return;
    }
    if (word.length === 9) {
        foundWords.value.push(word.toUpperCase());
        return;
    }
    foundWords.value.push(word);
}

function reveal() {
    revealDialog.value.showModal();
}
</script>

<style>
.green {
    color: #21bf73;
}
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
.found, .reveal {
    background-image: repeating-linear-gradient(transparent 0 28px, rgb(255 255 255 / 0.2) 28px 30px);
    width: 100%;
    line-height: 30px;
    font-size: 20px;
    height: 40vh;
    margin-top: 30px;
}
.reveal {
    height: auto;
}
.desc {
    width: 55vh;
    position: relative;
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
    cursor: pointer;
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
#reveal {
    width: 100%;
    position: absolute;
    bottom: 0;
}
dialog {
    width: 40%;
    height: 50%;
    background: var(--bg);
    color: white;
    outline: 4px solid white;
    padding: 0;
    font-size: 20px;
    box-shadow: 0 0 10px 15px rgb(0 0 0 / 0.2);
}
.close {
    padding: 0 15px 0 15px;
    margin: 10px;
    position: absolute;
    top: 0;
    right: 0;
}
#dialog {
    margin: 0;
    width: calc(100% - 40px);
    height: calc(100% - 40px);
    padding: 20px;
}
</style>
