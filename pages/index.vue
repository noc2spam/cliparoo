<template>
    <ClientOnly placeholder="loading....">
        <Confirm :show="appState.confirmOpen" @close="commitAction" />
        <div class="mx-auto p-4 max-w-[600px] w-full relative min-h-screen">
            <h1 class="text-7xl w-full text-center font-extrabold pt-5 text-sky-600">
                <span>Clip</span><span class="text-purple-600">-a-</span><span class="text-orange-600">roo</span><span
                    class="text-blue-600">!</span>
            </h1>
            <p class="text-center text-gray-500 pb-5 italic pt-4">localStorage based text snippet manager for
                the
                web.</p>
            <div class="w-full p-4 bg-gray-100 border border-gray-400 rounded-lg overflow-hidden relative">
                <div v-if="appState.textCopied"
                    class="absolute z-20 bg-green-600/95 py-2 px-2 text-white top-0 w-full left-0">Text copied to
                    clipboard!
                </div>
                <h2 class="text-xl font-bold">My Snippets</h2>
                <ul v-if="clipboard.length" class="py-4 flex flex-col gap-4">
                    <li class="grid grid-cols-8 border border-gray-400/60 bg-gray-200 rounded-lg p-2"
                        v-for="(pair, key) in clipboard" :key="key">
                        <div class="col-span-6">
                            <h2 class="text-lg font-medium">{{ pair.title }}</h2>
                            <div v-for="(line, index) in pair.description.split('\n')" :key="index">{{ line }}</div>
                        </div>
                        <div class="h-full col-span-2 grid place-items-center">
                            <div>
                                <button title="Copy this item to clipboard" class="text-gray-600 hover:text-gray-700"
                                    @click="copyToClipboard(pair.description)">
                                    <IconsCopy class="h-8 w-8" />
                                </button>
                                <button title="Edit this item" class="text-blue-600 hover:text-blue-700"
                                    @click="title = pair.title; description = pair.description; appState.popupOpen = true">
                                    <IconsEdit class="h-8 w-8" />
                                </button>
                                <button title="Delete this item" class="text-red-500 hover:text-red-600"
                                    @click="confirmAction(pair.title as string)">
                                    <IconsDelete class="h-8 w-8" />
                                </button>
                            </div>
                        </div>
                    </li>
                </ul>
                <div v-else class="py-2">
                    You currently have no snippets saved.
                </div>
                <div class="flex items-center justify-between">
                    <button @click="addClipboardItem"
                        class="bg-green-700 hover:bg-green-600 text-white rounded-lg px-4 py-2 text-lg">Add
                        Item</button>
                    <button v-if="clipboard.length" @click="confirmAction('', true)"
                        class="bg-red-600 hover:bg-red-500 text-white rounded-lg px-4 text-lg py-2">Clear
                        Clipboard</button>

                </div>
            </div>
        </div>
        <div v-if="appState.popupOpen" class="bg-gray-700/30 w-full h-full fixed inset-0" @click="closePopup"></div>
        <div v-if="appState.popupOpen"
            class="absolute bg-white border border-gray-400 max-w-[800px] w-[90vw] rounded-lg p-4 left-1/2 transform -translate-x-1/2 top-1/2 -translate-y-1/2 shadow-lg ">
            <div class="flex flex-col gap-2 w-full h-full">
                <h2 class="font-medium text-lg mb-4">{{ title.length ? 'Edit Item' : 'Add Item' }}</h2>
                <input v-model="title" placeholder="Title" class="border border-gray-300 rounded-lg p-2 mb-2 w-full" />
                <textarea v-model="description" placeholder="Description"
                    class="border border-gray-300 rounded-lg p-2 mb-2 w-full h-32 resize-none"></textarea>
                <button :disabled="description.length === 0 || title.length === 0" @click="saveItem"
                    class="bg-blue-500 disabled:opacity-60 text-white rounded-lg px-4 py-2">Save</button>
                <button @click="closePopup" class="bg-gray-300 text-gray-800 rounded-lg px-4 py-2">Cancel</button>
            </div>
        </div>
    </ClientOnly>
</template>
<script setup lang="ts">

const appState = useState<{
    popupOpen: boolean;
    textCopied: boolean;
    confirmOpen: boolean;
    confirmResult: { key?: string | null, confirmed: boolean | undefined, clearAll?: boolean };
}>('appState', () => ({
    popupOpen: false,
    textCopied: false,
    confirmOpen: false,
    confirmResult: { key: null, confirmed: false },
}));
const clipboard = ref<{ title: string, description: string }[]>([]);
const description = ref('');
const title = ref('');
const timeout = ref<number | null>(null);
function addClipboardItem() {
    appState.value = { ...appState.value, popupOpen: true };
}
function clearClipboard() {
    localStorage.removeItem('clipboard');
    clipboard.value = [];
}
function closePopup() {
    appState.value = { ...appState.value, popupOpen: false };
    title.value = '';
    description.value = '';
}
function commitAction(confirmed: boolean) {
    appState.value = { ...appState.value, confirmOpen: false };
    if (!confirmed) {
        return;
    }
    if (appState.value.confirmResult.clearAll) {
        clearClipboard();
    } else {
        if (typeof appState.value.confirmResult.key === 'string') {
            removeItem(appState.value.confirmResult.key as string);
        }
    }

    appState.value.confirmResult = { key: null, confirmed: false };
}
function confirmAction(key: string, clearAll = false) {
    appState.value.confirmResult = { key, confirmed: undefined, clearAll };
    appState.value = { ...appState.value, confirmOpen: true };
}
function copyToClipboard(value: string) {
    navigator.clipboard.writeText(value).then(() => {
        appState.value = { ...appState.value, textCopied: true };
        if (timeout.value) {
            clearTimeout(timeout.value);
        }
        timeout.value = setTimeout(() => {
            appState.value = { ...appState.value, textCopied: false };
        }, 2000);
    });
}
function removeItem(key: string, clearAll = false) {
    if (clearAll) {
        clearClipboard();
        return;
    }
    const currentClipboard = clipboard.value.filter((item) => item.title !== key);
    clipboard.value = currentClipboard;
    localStorage.setItem('clipboard', JSON.stringify(currentClipboard));
}
function saveItem() {
    const currentClipboard = clipboard.value;
    const currentItem = currentClipboard.findIndex((item) => item.title === title.value);
    if (currentItem !== -1 && currentClipboard[currentItem]) {
        currentClipboard[currentItem].description = description.value;
        clipboard.value = currentClipboard;
        localStorage.setItem('clipboard', JSON.stringify(currentClipboard));
        closePopup();
        return;
    }
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
<style>
body {
    font-family: Seravek, 'Gill Sans Nova', Ubuntu, Calibri, 'DejaVu Sans', source-sans-pro, sans-serif;
    font-weight: normal;
}
</style>