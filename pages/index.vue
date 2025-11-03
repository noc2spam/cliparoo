<template>
    <div>
        <h1>Clipboard Contents</h1>
        <ul>
            <li v-for="(value, key) in clipboard" :key="key">
                <div>
                    <strong>{{ key }}:</strong> {{ value }}
                </div>
                <div>
                    <button @click="copyToClipboard(value)">Copy</button>
                    <button @click="deleteClipboardItem(key as string)">Delete</button>
                </div>
            </li>
        </ul>
        <button @click="clearClipboard">Clear Clipboard</button>
        <button @click="addClipboardItem">Add Clipboard Item</button>
        <div v-if="popupOpen">
            <h2>Add Clipboard Item</h2>
            <input v-model="newKey" placeholder="Key" />
            <input v-model="newValue" placeholder="Value" />
            <button @click="saveClipboardItem">Save</button>
            <button @click="popupOpen = false">Cancel</button>
        </div>
    </div>
</template>
<script setup lang="ts">
function clearClipboard() {
    localStorage.removeItem('clipboard');
    for (const key in clipboard) {
        delete clipboard[key];
    }
}
const clipboard = reactive<{ [key: string]: string }>({});
const popupOpen = ref(false);
const newKey = ref('');
const newValue = ref('');
function addClipboardItem() {
    popupOpen.value = true;
}
function saveClipboardItem() {
    if (newKey.value && newValue.value) {
        clipboard[newKey.value] = newValue.value;
        localStorage.setItem('clipboard', JSON.stringify(clipboard));
        newKey.value = '';
        newValue.value = '';
        popupOpen.value = false;
    }
}
function deleteClipboardItem(key: string) {
    delete clipboard[key];
    localStorage.setItem('clipboard', JSON.stringify(clipboard));
}

function copyToClipboard(value: string) {
    navigator.clipboard.writeText(value).then(() => {
        alert('Copied to clipboard!');
    });
}

onMounted(() => {
    const rawClipboard = localStorage.getItem('clipboard');
    if (rawClipboard) {
        Object.assign(clipboard, JSON.parse(rawClipboard));
    }
})
</script>