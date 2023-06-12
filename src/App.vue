<script setup>
import { ref, computed, watch, onMounted, onBeforeUnmount } from "vue"
import {Html5Qrcode} from "html5-qrcode"
import ErrorInfo from './components/ErrorInfo.vue'
import TextPanel from './components/TextPanel.vue'

const scanner = ref(undefined)
const decodedText = ref('')
const status = ref(undefined)
const errorInfo = ref('')
const counter = ref(3)
const timer = ref(undefined)

onMounted(() => startScanner())
onBeforeUnmount(() => {
    stopScanner()
    clearInterval(timer.value)
    timer.value = undefined
})
watch(decodedText, (nv) => {
    if(nv.length === 0) return
    fetch(`/api/srp/check?data=${decodedText.value}`)
    .then(response => {
        status.value = response.status
        if(response.status === 200) {
            // OK
            console.log('OK:', decodedText)

            counter.value = 3
            timer.value = setInterval(() => {
                counter.value -= 1
                if(counter.value === 0) {
                    clearInterval(timer.value)
                    timer.value = undefined
                    onResumeScan()
                }                
            }, 1000)
        }
        else {
            // błąd
            console.error('Error:', decodedText, 'status:', response.status)
        }
        return response.text()
    })
    .then(d => errorInfo.value = d)
})

const main_background = computed(() => {
    if(status.value === 200) return "bg-success"
    if(status.value !== undefined) return "bg-danger"
    return ""
})

const pass_nr = computed(() => decodedText.value.split('-')[0] || "")
const regnum = computed(() => {
    let d = new Date()
    switch(d.getDay()) {
        case 5: return decodedText.value.split('-')[1]
        case 6: return decodedText.value.split('-')[2]
        case 0: return decodedText.value.split('-')[3]
    }
    return decodedText.value.split('-')[1] || ""
})

function startScanner() {
    scanner.value = new Html5Qrcode("preview")
    scanner.value.start(
        { facingMode: "environment" },
        { fps: 10, qrbox: 200 },
        (text, result) => {
            console.log('Detected code:', text, result)
            decodedText.value = text
            scanner.value.pause(true)
        }
    )
}

function stopScanner() {
    if(scanner.value != undefined) {
        scanner.value.stop()
        scanner.value = undefined
    }
}

function onResumeScan() {
    status.value = undefined
    decodedText.value = ''    
    scanner.value?.resume() 
}
</script>

<template>
    <div id="main" :class="main_background">
        <div id="preview"></div>

        <div id="second">
            <ErrorInfo v-if="status > 200"
                label="Opis przypadku"
                :text="errorInfo"
            />
            <TextPanel 
                label="Numer rejestracyjny pojazdu"
                :text="regnum" 
            />
            <TextPanel 
                label="Numer identyfikatora parkingowego"
                :text="pass_nr" 
            />

            <div class="btns-layout">
                <button v-if="scanner?.getState() === 3"
                    class="btn btn-success"
                    @click="onResumeScan"
                >
                    <div><i class="fa-solid fa-qrcode fa-3x"/></div>
                    <div>Skanuj ponownie</div>
                    <small v-if="status===200">Autostart za {{ counter }} sekund</small>
                </button>
            </div>
        </div>
    </div>
</template>

<style scoped>
#id {
    width: 300px;
    height: 300px;
}

.decoded-text {
    width: 100%;
    text-align: center;
}
.btns-layout {
    margin: 0;
    padding: 0;

    display: flex;
    flex-direction: row;
    justify-content: center;
    gap: 4pt;
}

#second {
    padding: 9pt;
    display: flex;
    flex-direction: column;
    gap: 9pt;
    justify-items: stretch;
}

#main {
    min-height: 100vh;
    display: grid;
    grid-template-columns: 1fr;
    grid-template-rows: auto 1fr;
    gap: 0;
}

@media screen and (orientation: landscape) {
    #main {
        grid-template-columns: 1fr 1fr;
        grid-template-rows: 1fr;
    }
}
</style>
