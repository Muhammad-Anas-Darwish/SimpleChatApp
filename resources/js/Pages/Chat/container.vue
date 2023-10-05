<script setup>
import AppLayout from '@/Layouts/AppLayout.vue';
import MessageContainer from './messageContainer.vue';
import InputMessage from './inputMessage.vue';
import ChatRoomSelection from './chatRoomSelection.vue';
import { ref, onMounted, watch, watchEffect } from 'vue';

const chatRooms = ref([]);
const currentRoom =  ref([]);
const messages = ref([]);

function connect() {
    if (currentRoom.value.id) {
        let vm = this;
        getMessages();
        window.Echo.private("chat." + currentRoom.id).listen('.message.new', e => {
            vm.getMessages();
        });
    }
}

function disconnect(room) {
    window.Echo.leave("chat." + room.id);
}

// watchEffect(
//     () => {
//         connect();
//     }
// );

watch(
    currentRoom, (val, oldVal) => {
        if (oldVal.id) {
            disconnect(oldVal);
        }
        connect();
    }
);

function getRooms() {
    axios.get('/chat/rooms').then(response => {
        chatRooms.value = response.data;
        setRoom(response.data[0]);
    }).catch(error => {
        console.log(error);
    });
};

const setRoom = (room) => {
    currentRoom.value = room;
};

const getMessages = () => {
    axios.get('chat/room/' + currentRoom.value.id + '/messages').then(response => {
        messages.value = response.data;
    }).catch(error => {
        console.log(error);
    });
};

// setup() {
onMounted(() => {
    getRooms();
});
// };


// export default {
//     components: {

//     }
//     data: function () {
//         return {
//             chatRooms: [],
//             currentRoom: [],
//             messages: [],
//         };
//     },
//     methods: {
//         getRooms() {
//             axios.get('/chat/rooms').then(response => {
//                 this.chatRooms = response.data;
//                 this.setRoom(response.data[0]);
//             }).catch(error => {
//                 console.log(error);
//             });
//         },
//         setRoom(room) {
//             this.currentRoom = room;
//             this.getMessages();
//         },
//         getMessages() {
//             axios.get('chat/room/' + this.currentRoom.id + '/messages').then(response => {
//                 this.messages = response.data;
//             }).catch(error => {
//                 console.log(error);
//             });
//         }
//     },
//     created() {
//         this.getRooms();
//     }
// }
</script>

<template>
    <AppLayout title="Dashboard">
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                <chat-room-selection v-if="currentRoom.id" :rooms="chatRooms" :currentRoom="currentRoom" v-on:roomchanged="setRoom($event)" />
            </h2>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                    <message-container :messages="messages" />
                    <input-message :room="currentRoom" v-on:messagesent="getMessages()" />
                </div>
            </div>
        </div>
    </AppLayout>
</template>
