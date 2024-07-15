<script setup>
import { ref, computed, watch, nextTick, onMounted, onBeforeUnmount } from "vue"
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome"
import { faQrcode } from "@fortawesome/free-solid-svg-icons"
import { Html5Qrcode } from "html5-qrcode"
import ErrorInfo from './components/ErrorInfo.vue'
import SuccessInfo from './components/SuccessInfo.vue'
import StartInfo from './components/StartInfo.vue'

const scanner = ref(undefined)
const decodedText = ref('')
const status = ref(undefined)
const errorInfo = ref('')
const counter = ref(3)
const timer = ref(undefined)

onMounted(() => {
    screen.orientation.onchange = () => {
        stopScanner()
        status.value = undefined
        decodedText.value = ''
        nextTick(startScanner)
    }
    startScanner()
})
onBeforeUnmount(() => {
    stopScanner()
    clearInterval(timer.value)
    timer.value = undefined
})
watch(decodedText, (nv) => {
    if(nv.length === 0) return
    if(nv.includes('pk-')) {
        check(`/api/pk/check?data=${nv}`)

    }
    else {
        check(`/api/srp/check?data=${nv}`)
    }
})

function check(url) {
    fetch(url)
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
}

const pass_nr = computed(() => {
    if(decodedText.value.includes('pk-'))
        return decodedText.value.split('-')[1] || ""
    else
        return decodedText.value.split('-')[0] || ""
})
const regnum = computed(() => {
    let d = new Date()
    if(decodedText.value.includes('pk-')) {
        switch(d.getDay()) {
            case 5: return decodedText.value.split('-')[2]
            case 6: return decodedText.value.split('-')[3]
            case 0: return decodedText.value.split('-')[4]
        }
        return decodedText.value.split('-')[2] || ""
    }
    else {
        switch(d.getDay()) {
            case 5: return decodedText.value.split('-')[1]
            case 6: return decodedText.value.split('-')[2]
            case 0: return decodedText.value.split('-')[3]
        }
        return decodedText.value.split('-')[1] || ""
    }
})

function startScanner() {
    scanner.value = new Html5Qrcode("preview")
    scanner.value.start(
        { facingMode: "environment" },
        { fps: 10 },
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
    <main>
        <div id="first">
            <div id="preview"></div>
        </div>

        <div id="second">
            <ErrorInfo v-if="status > 200"
                label="Opis przypadku"
                :text="errorInfo"
            />
            <SuccessInfo v-else-if="status === 200" />
            <StartInfo v-else />

            <div>
                <div class="input-group">
                    <input type="text" class="form-control" value="Numer identyfikatora">
                    <input type="text" class="form-control" value="Numer rejestracyjny">
                </div>
                <div class="input-group">
                    <input type="text" class="form-control layout" disabled readonly v-model="pass_nr" >
                    <input type="text" class="form-control layout" disabled readonly v-model="regnum" >
                </div>
            </div>

            <div class="btns-layout">
                <button 
                    :disabled="scanner?.getState() !== 3"
                    class="btn btn-success"
                    @click="onResumeScan"
                >
                    <FontAwesomeIcon :icon="faQrcode" />
                    <div>Skanuj ponownie</div>
                    <small v-if="status===200">Autostart za {{ counter }} sekund</small>
                </button>
            </div>
        </div>
    </main>
</template>

<style scoped>
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
    align-items: center;
    gap: 4pt;
}

#preview {
    max-width: 100%;
    max-height: 100%;
}

#second {
    padding: 4pt;
    display: flex;
    flex-direction: column;
    gap: 9pt;
}

input.layout {
    text-align: center;
    text-transform: uppercase;
    font-size: 1.4rem;
    font-weight: bold;
}

main {
    min-width: 100vw;
    max-width: 100vw;
    display: grid;
    grid-template-columns: 1fr;
    grid-template-rows: 1fr 1fr;
    gap: 1pt;
}

@media screen and (orientation: landscape) {
    main {
        min-width: 100vw;
        max-width: 100vw;
        grid-template-columns: 1fr 1fr;
        grid-template-rows: 1fr;
    }
}
</style>
