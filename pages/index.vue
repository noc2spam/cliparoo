<template>
    <ClientOnly placeholder="loading....">
        <Confirm :show="appState.confirmOpen" @close="commitAction" />
        <div class="mx-auto p-4 max-w-[800px] w-full relative min-h-[80vh]">
            <h1 class="text-7xl w-full text-center font-extrabold pt-5 text-">
                <span class="text-[#3c40bd]">Clip</span><span class="text-[#bd3f3c]">-a-</span><span
                    class="text-[#bd943c]">roo</span><span class="text-[#3cbd56]">!</span>
            </h1>
            <p class="text-center text-gray-700 pb-5 italic pt-4">localStorage based text snippet manager for
                the
                web.</p>
            <div class="w-full p-4 bg-[#dda15e] shadow-xl rounded-lg overflow-hidden relative">
                <div class="absolute top-4 right-4 flex gap-2">
                    <div>
                        <input ref="importInput" type="file" accept=".json" @change="importClipboard" class="hidden">
                        <button @click="triggerImport"
                            class="bg-[#40228a] hover:bg-[#40228a95] text-white rounded-lg px-3 py-1 text-sm transition-colors">
                            Import
                        </button>
                    </div>
                    <div>
                        <button @click="exportClipboard" :disabled="!clipboard.length"
                            class="bg-[#8a3022] hover:bg-[#8a302295] disabled:bg-gray-400 disabled:cursor-not-allowed text-white rounded-lg px-3 py-1 text-sm transition-colors">
                            Export
                        </button>
                    </div>
                </div>
                <div v-if="appState.textCopied"
                    class="fixed z-20 bg-[#5edd6b] py-2 px-2 text-white top-0 text-center font-medium w-full left-0">
                    Text copied
                    to
                    clipboard!
                </div>
                <h2 class="text-xl font-medium text-white">My Snippets</h2>
                <ul v-if="clipboard.length" class="py-4 flex flex-col gap-4">
                    <li class="flex  flex-col md:flex-row items-start justify-between border border-gray-400/60 bg-[#fefae0] rounded-lg p-2"
                        v-for="(pair, key) in clipboard" :key="key">
                        <div class="col-span-6">
                            <h2 class="text-lg font-medium">{{ pair.title }}</h2>
                            <div v-for="(line, index) in pair.description.split('\n')" :key="index">{{ line }}</div>
                        </div>
                        <div class="h-full col-span-2 grid place-items-start pt-3">
                            <div class="flex  gap-2">
                                <button title="Copy this item to clipboard"
                                    class="text-white bg-[#77dd5e95] hover:bg-[#77dd5e] rounded-full p-2"
                                    @click="copyToClipboard(pair.description)">
                                    <IconsCopy class="h-6 w-6 md:h-8 md:w-8" />
                                </button>
                                <button title="Edit this item"
                                    class="bg-[#5eb5dd95] hover:bg-[#5eb5dd] text-white p-2 rounded-full "
                                    @click="title = pair.title; description = pair.description; appState.popupOpen = true">
                                    <IconsEdit class="h-6 w-6 md:h-8 md:w-8" />
                                </button>
                                <button title="Delete this item"
                                    class="bg-[#dd5e5e95] hover:bg-[#dd5e5e] text-white p-2 rounded-full "
                                    @click="confirmAction(pair.title as string)">
                                    <IconsDelete class="h-6 w-6 md:h-8 md:w-8" />
                                </button>
                            </div>
                        </div>
                    </li>
                </ul>
                <div v-else class="py-2">
                    You currently have no snippets saved.
                </div>
                <div class="flex  flex-col md:flex-row gap-2 items-center justify-between">
                    <button @click="addClipboardItem"
                        class="bg-[#386c39] hover:bg-[#606c38] text-white rounded-lg px-4 py-2 md:w-auto w-full text-lg">Add
                        Item</button>
                    <button v-if="clipboard.length" @click="confirmAction('', true)"
                        class="bg-[#bd3f3c] hover:bg-[#bd693c] text-white rounded-lg px-4 text-lg py-2  md:w-auto w-full">Clear
                        Clipboard</button>
                </div>
            </div>
        </div>
        <div v-if="appState.popupOpen" class="bg-gray-700/30 w-full h-full fixed inset-0" @click="closePopup"></div>
        <div v-if="appState.popupOpen"
            class="fixed bg-white border border-gray-400 max-w-[800px] w-[90vw] rounded-lg p-4 left-1/2 transform -translate-x-1/2 top-1/2 -translate-y-1/2 shadow-lg ">
            <div class="flex flex-col gap-2 w-full h-full">
                <h2 class="font-medium text-lg mb-4">{{ title.length ? 'Edit Item' : 'Add Item' }}</h2>
                <input v-model="title" placeholder="Title" class="border border-gray-300 rounded-lg p-2 mb-2 w-full" />
                <textarea v-model="description" placeholder="Description"
                    class="border border-gray-300 rounded-lg p-2 mb-2 w-full h-32 resize-none"></textarea>
                <button :disabled="description.length === 0 || title.length === 0" @click="saveItem"
                    class="bg-blue-700 disabled:opacity-60 text-white rounded-lg px-4 py-2">Save</button>
                <button @click="closePopup" class="bg-gray-300 text-gray-800 rounded-lg px-4 py-2">Cancel</button>
            </div>
        </div>
        <div>
            <p class="text-center text-gray-500 italic">Source code available on <a
                    href="https://github.com/noc2spam/cliparoo" class="text-blue-500 hover:underline"
                    target="_blank">GitHub</a>.</p>
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
const importInput = ref<HTMLInputElement | null>(null);
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
function exportClipboard() {
    if (!clipboard.value.length) return;

    const dataStr = JSON.stringify(clipboard.value, null, 2);
    const dataBlob = new Blob([dataStr], { type: 'application/json' });

    const link = document.createElement('a');
    link.href = URL.createObjectURL(dataBlob);
    link.download = `cs-export-${Date.now()}.json`;
    link.click();

    // Clean up the URL object
    URL.revokeObjectURL(link.href);
}
function importClipboard(event: Event) {
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (e) => {
        try {
            const result = e.target?.result as string;
            const importedData = JSON.parse(result);

            if (Array.isArray(importedData)) {
                clipboard.value = importedData;
                localStorage.setItem('clipboard', JSON.stringify(importedData));
            } else {
                alert('Invalid file format. Please select a valid JSON file.');
            }
        } catch (error) {
            alert('Error reading file. Please make sure it\'s a valid JSON file.');
        }
    };
    reader.readAsText(file);

    // Clear the input value so the same file can be imported again
    input.value = '';
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
function triggerImport() {
    importInput.value?.click();
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
    @apply font-normal bg-[#fefae0];
}
</style>