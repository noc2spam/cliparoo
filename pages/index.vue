<template>
    <div class="mx-auto p-4 max-w-[600px] w-full">
        <div class="w-full p-4 bg-gray-100 border border-gray-400 rounded-lg">
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
            <button @click="clearClipboard">Clear Clipboard</button>
            <button @click="addClipboardItem">Add Clipboard Item</button>
            <div v-if="appState.popupOpen">
                <h2>Add Clipboard Item</h2>
                <input v-model="title" placeholder="Key" />
                <input v-model="description" placeholder="Value" />
                <button @click="saveClipboardItem">Save</button>
                <button @click="closePopup">Cancel</button>
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
    Object.assign(appState.value, { popupOpen: true });
}
function clearClipboard() {
    localStorage.removeItem('clipboard');
    Object.assign(clipboard, []);
}
function closePopup() {
    Object.assign(appState.value, { popupOpen: false, newKey: '', newValue: '' });
}
function copyToClipboard(value: string) {
    navigator.clipboard.writeText(value).then(() => {
        alert('Copied to clipboard!');
    });
}
function deleteClipboardItem(key: string) {
    const currentClipboard = clipboard.value.filter((item) => item.title !== key);

    Object.assign(clipboard, currentClipboard);
    localStorage.setItem('clipboard', JSON.stringify(currentClipboard));
}
function saveClipboardItem() {
    const currentClipboard = clipboard.value;
    currentClipboard.push({ title: title.value, description: description.value });
    Object.assign(clipboard.value, currentClipboard);
    localStorage.setItem('clipboard', JSON.stringify(currentClipboard));
    closePopup();
}
onMounted(() => {
    const rawClipboard = localStorage.getItem('clipboard');
    if (rawClipboard) {
        Object.assign(clipboard.value, JSON.parse(rawClipboard));
    }
})
</script>