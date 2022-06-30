<template>
    <ion-page>
        <ion-header>Home Page</ion-header>
        <ion-content>
            <div>
                <p>Hi, how are you?</p>
                <ion-button @click="scan">Scan</ion-button>
                <p>{{ble_enabled}}</p>
                <p>Error:{{error}}</p>
                <p>Scanning: {{scanning}}</p>
                <p>Tried Scanning: {{tried_scanning}}</p>
                <p v-for="(result,i) in results" :key="i">{{result}}</p>
                <p>Console Messages:</p>
                <p v-for="(msg, i) in consoleMessages" :key="i">{{msg}}</p>
            </div>
        </ion-content>
    </ion-page>
</template>
<script setup lang="ts">
    import { IonPage } from '@ionic/vue';
    import { BleClient, ScanResult } from '@capacitor-community/bluetooth-le';
    import { onMounted, ref } from 'vue';

    const ble_enabled = ref(false)
    const error = ref('')
    const results = ref<ScanResult[]>([])
    const scanning = ref(false)
    const tried_scanning = ref(false)

    const consoleMessages = ref<string[]>([])


    const init = async () => {
        consoleMessages.value.push('initializing!')
        try {
            await BleClient.initialize()

            ble_enabled.value = await BleClient.isEnabled()

            await BleClient.startEnabledNotifications((enabled:boolean) => {
                ble_enabled.value = enabled
            })
        } catch (err) {
            error.value = (err as Error).message
        }
    }
    
    const scan = async () => {
        results.value = []
        scanning.value = true
        tried_scanning.value = true

        try {
            await BleClient.requestLEScan({}, result => {
                results.value.push(result)
            })

            setTimeout(() => {
                void BleClient.stopLEScan().then(() => {
                    scanning.value = false
                })
            }, 10000);
        } catch (err) {
            error.value = (err as Error).message
        }
    }

    onMounted(() => init())
</script>