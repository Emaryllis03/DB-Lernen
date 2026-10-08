<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { course } from './data/course.js'

const selected = ref(Number(localStorage.getItem('selectedDay') || 1))
const tab = ref('learn')
const cardIndex = ref(0)
const flipped = ref(false)
const quizAnswers = ref({})
const completed = ref(JSON.parse(localStorage.getItem('completedDays') || '[]'))
const reviewOnly = ref(false)

const day = computed(() => course.find(d => d.day === selected.value) || course[0])
const progress = computed(() => Math.round(completed.value.length / course.length * 100))
const currentCard = computed(() => day.value.cards[cardIndex.value])

function selectDay(n){ selected.value=n; tab.value='learn'; cardIndex.value=0; flipped.value=false; quizAnswers.value={}; localStorage.setItem('selectedDay',n); window.scrollTo({top:0,behavior:'smooth'}) }
function toggleComplete(){ if(completed.value.includes(day.value.day)) completed.value=completed.value.filter(x=>x!==day.value.day); else completed.value=[...completed.value,day.value.day]; localStorage.setItem('completedDays',JSON.stringify(completed.value)) }
function nextCard(){ if(cardIndex.value < day.value.cards.length-1){cardIndex.value++;flipped.value=false} else tab.value='quiz' }
function resetQuiz(){quizAnswers.value={}}
function quizScore(){return day.value.quiz.reduce((s,q,i)=>s+(quizAnswers.value[i]===q[1]?1:0),0)}
function allDone(){ return day.value.quiz.length>0 && Object.keys(quizAnswers.value).length===day.value.quiz.length }
function goNext(){ if(selected.value<14) selectDay(selected.value+1) }
watch(selected,()=>{cardIndex.value=0;flipped.value=false})
onMounted(()=>{})
</script>

<template>
<div class="app-shell">
  <aside class="sidebar">
    <div class="brand"><div class="brand-icon">🚆</div><div><strong>Bahn Lernen</strong><span>14-Tage-Kurs</span></div></div>
    <div class="progress-box"><div class="progress-head"><span>Fortschritt</span><b>{{ progress }}%</b></div><div class="bar"><i :style="{width:progress+'%'}"></i></div><small>{{ completed.length }} / 14 Tage abgeschlossen</small></div>
    <nav><button v-for="d in course" :key="d.day" :class="{active:d.day===selected}" @click="selectDay(d.day)"><span class="day-number">{{ d.day }}</span><span class="day-title">{{ d.title }}</span><span v-if="completed.includes(d.day)" class="check">✓</span></button></nav>
  </aside>

  <main class="main">
    <header class="topbar"><div><span class="eyebrow">TAG {{day.day}} / 14</span><h1>{{day.title}}</h1></div><button class="complete" @click="toggleComplete">{{completed.includes(day.day)?'✓ Tag abgeschlossen':'Tag abschließen'}}</button></header>
    <section class="hero"><div><p class="label">Lernziel</p><h2>{{day.goal}}</h2></div><div class="hero-badge">{{day.cards.length}} Karten · {{day.quiz.length}} Quizfragen</div></section>

    <div class="tabs"><button :class="{active:tab==='learn'}" @click="tab='learn'">📖 Lernen</button><button :class="{active:tab==='cards'}" @click="tab='cards'">🧠 Karteikarten</button><button :class="{active:tab==='quiz'}" @click="tab='quiz'">✏️ Quiz</button></div>

    <section v-if="tab==='learn'" class="content-grid">
      <article class="panel"><h3>🟢 Muss ich können</h3><ul><li v-for="x in day.must" :key="x">{{x}}</li></ul></article>
      <article class="panel"><h3>🟡 Muss ich verstehen</h3><ul><li v-for="x in day.understand" :key="x">{{x}}</li></ul></article>
      <article class="panel"><h3>⚪ Sollte ich kennen</h3><ul><li v-for="x in day.know" :key="x">{{x}}</li></ul></article>
      <article class="panel wide"><h3>🎯 Lernstrategie für heute</h3><ol><li>Lernziele einmal laut lesen.</li><li>Die Kernbegriffe ohne Spickzettel erklären.</li><li>Karteikarten durchgehen.</li><li>Quiz ohne Hilfe lösen.</li><li>Fehler direkt noch einmal lernen.</li></ol></article>
    </section>

    <section v-if="tab==='cards'" class="cards-view">
      <div class="card-counter">Karte {{cardIndex+1}} von {{day.cards.length}}</div>
      <button class="flashcard" @click="flipped=!flipped"><div class="card-label">{{flipped?'ANTWORT':'FRAGE'}}</div><h2>{{flipped?currentCard[1]:currentCard[0]}}</h2><p>Zum Umdrehen klicken</p></button>
      <div class="card-actions"><button class="secondary" @click="flipped=!flipped">{{flipped?'Frage zeigen':'Antwort zeigen'}}</button><button class="primary" @click="nextCard">{{cardIndex===day.cards.length-1?'Zum Quiz':'Nächste Karte →'}}</button></div>
    </section>

    <section v-if="tab==='quiz'" class="quiz-view">
      <div v-for="(q,i) in day.quiz" :key="i" class="quiz-question"><div class="q-head"><span>{{i+1}}</span><h3>{{q[0]}}</h3></div><label v-for="option in [q[1],...q[2]]" :key="option" :class="{chosen:quizAnswers[i]===option, correct:allDone() && option===q[1], wrong:allDone() && quizAnswers[i]===option && option!==q[1]}"><input type="radio" :name="'q'+i" :value="option" v-model="quizAnswers[i]"/>{{option}}</label></div>
      <div class="quiz-result" v-if="allDone()"><strong>{{quizScore()}} / {{day.quiz.length}}</strong><span>{{quizScore()===day.quiz.length?'Perfekt – weiter zum nächsten Tag!':'Noch einmal die Karten zu den Fehlern ansehen.'}}</span></div>
      <div class="card-actions"><button class="secondary" @click="resetQuiz">Zurücksetzen</button><button class="primary" v-if="selected<14" @click="goNext">Nächster Tag →</button></div>
    </section>

    <footer><button class="nav-day" :disabled="selected===1" @click="selectDay(selected-1)">← Vorheriger Tag</button><span>Quelle: hochgeladene DB-InfraGO-Ausbildungspräsentationen, September 2026</span><button class="nav-day" :disabled="selected===14" @click="goNext">Nächster Tag →</button></footer>
  </main>
</div>
</template>
