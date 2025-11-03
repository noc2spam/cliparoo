<template>
    <div class="mx-auto p-4 max-w-[600px] w-full relative min-h-screen">
        <div class="w-full p-4 bg-gray-100 border border-gray-400 rounded-lg ">
            <h1 class="text-xl font-bold">My Clipboard</h1>
            <ul v-if="clipboard.length">
                <li v-for="(pair, key) in clipboard" :key="key">
                    <div>
                        <strong>{{ pair.title }}:</strong> {{ pair.description }}
                    </div>
                    <div>
                        <button @click="copyToClipboard(pair.description)">Copy</button>
                        <button @click="deleteClipboardItem(pair.title as string)">Delete</button>
                    </div>
                </li>
            </ul>
            <div v-else>
                <p>Your clipboard is empty.</p>
            </div>
            <div class="flex items-center justify-between">
                <button v-if="clipboard" @click="clearClipboard"
                    class="bg-red-500 text-white rounded-lg px-4 py-2">Clear
                    Clipboard</button>
                <button @click="addClipboardItem" class="bg-blue-500 text-white rounded-lg px-4 py-2">Add Item</button>
            </div>
            <div v-if="appState.popupOpen"
                class="absolute bg-white border border-gray-400 rounded-lg p-4 left-1/2 transform -translate-x-1/2 top-1/2 -translate-y-1/2 shadow-lg">
                <div class="flex flex-col gap-2 w-64 h-64">
                    <h2>Add Item</h2>
                    <input v-model="title" placeholder="Title"
                        class="border border-gray-300 rounded-lg p-2 mb-2 w-full" />
                    <input v-model="description" placeholder="Value"
                        class="border border-gray-300 rounded-lg p-2 mb-2 w-full" />
                    <button @click="saveClipboardItem" class="bg-blue-500 text-white rounded-lg px-4 py-2">Save</button>
                    <button @click="closePopup" class="bg-gray-300 text-gray-800 rounded-lg px-4 py-2">Cancel</button>
                </div>
            </div>
        </div>
    </div>
</template>
<script setup lang="ts">

const appState = useState('appState', () => ({
    popupOpen: false
}));
const clipboard = ref<{ title: string, description: string }[]>([]);
const title = ref('');
const description = ref('')
function addClipboardItem() {
    appState.value = { popupOpen: true };
}
function clearClipboard() {
    localStorage.removeItem('clipboard');
    clipboard.value = [];
}
function closePopup() {
    appState.value = { popupOpen: false };
    title.value = '';
    description.value = '';
}
function copyToClipboard(value: string) {
    navigator.clipboard.writeText(value).then(() => {
        alert('Copied to clipboard!');
    });
}
function deleteClipboardItem(key: string) {
    const currentClipboard = clipboard.value.filter((item) => item.title !== key);
    clipboard.value = currentClipboard;
    localStorage.setItem('clipboard', JSON.stringify(currentClipboard));
}
function saveClipboardItem() {
    const currentClipboard = clipboard.value;
    currentClipboard.push({ title: title.value, description: description.value });
    clipboard.value = currentClipboard;
    localStorage.setItem('clipboard', JSON.stringify(currentClipboard));
    closePopup();
}
onMounted(() => {
    const rawClipboard = localStorage.getItem('clipboard');
    if (rawClipboard) {
        Object.assign(clipboard.value, JSON.parse(rawClipboard));
    }
})
useSeoMeta({
    title: 'Cliparoo - Store & manage reused text snippets for easy copy-paste',
    description: 'Cliparoo is your go-to solution for managing and storing text snippets, making copy-pasting a breeze. Data is stored locally for your privacy.'
});
</script>