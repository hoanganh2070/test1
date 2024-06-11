<script setup>
import { useRouter } from 'vue-router';
import InputForm from '../components/InputForm.vue';
import axios from 'axios';

import { ref } from 'vue';

const router = useRouter();

const username = ref('');
const password = ref('');
const progress = ref('');
const time = ref(0);
const count = ref(0);

axios.interceptors.request.use(function (config) {

    config.metadata = { startTime: new Date() }
    return config;
}, function (error) {
    return Promise.reject(error);
});


axios.interceptors.response.use(function (response) {

    response.config.metadata.endTime = new Date()
    response.duration = response.config.metadata.endTime - response.config.metadata.startTime
    return response;
}, function (error) {
    error.config.metadata.endTime = new Date();
    error.duration = error.config.metadata.endTime - error.config.metadata.startTime;
    return Promise.reject(error);
});


function randomPassword() {
    let result = '';
    const characters = '0123456789';
    const charactersLength = characters.length;
    for (let i = 0; i < 6; i++) {
        result += characters.charAt(Math.floor(Math.random() * charactersLength));
    }
    return result;
}

async function logIn() {

    let milliseconds = 0;
    const timer = setInterval(() => {
        milliseconds++;
        time.value = milliseconds;
    }, 1);
    progress.value = "Checking...";


    while (true) {

        await axios.post('http://localhost:3000/login', {
            username: username.value,
            password: randomPassword()
        }).then(res => {
            if (res.response.status === 200) {
            progress.value = "Success";
            document.getElementById('file').value = 1000000;

        }
        }).catch(async (err) => {
            console.log(err.duration);
            time.value += err.duration;
            if (err.response.status === 429) {
                progress.value = err.response.data.progress;
               
                await new Promise(resolve => setTimeout(resolve, 1000));

            }

            if (err.response.status === 401) {

                count.value = count.value + 1;
                document.getElementById('file').value = count.value;
                

            }
        });

        if (progress.value === "Success") {
            clearInterval(timer);
            break;}


       


    }
}

</script>

<template>

    <div class="w-screen h-screen flex flex-col lg:flex-row md:p-48 items-center overflow-x-hidden">

        <div class="flex w-full flex-col space-y-5 items-center">
            <form class="flex w-[350px] flex-col space-y-5 items-center">
                <div class="w-full">
                    <p class="text-2xl float-start">Log In</p>
                </div>
                <div>
                    <input v-model="username" type="text"
                        class="bg-gray-50 border border-gray-300 text-gray-900 text-md rounded-lg  block w-[350px] p-2.5 "
                        placeholder="Username" required>
                </div>
                <div>

                </div>


                <button @click="logIn()"
                    class="bg-slate-600 text-white p-2 w-32 rounded-lg hover:bg-black duration-200">Log In</button>

                <label for="file" v-text="time">
                </label>
                <p>ms</p>
                <label for="file" v-text="progress"></label>
                <progress id="file" max="1000000">70 %</progress>



            </form>

        </div>
    </div>

</template>