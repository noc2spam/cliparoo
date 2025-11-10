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
                        <input ref="importInput" type="file" accept=".json" @change="importSnippets" class="hidden">
                        <button @click="triggerImport"
                            class="bg-[#40228a] hover:bg-[#40228a95] text-white rounded-lg px-3 py-1 text-sm transition-colors">
                            Import
                        </button>
                    </div>
                    <div>
                        <button @click="exportSnippets" :disabled="!appState.snippets.length"
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
                <div v-if="appState.snippets.length" class="flex justify-between items-center py-2 mb-2">
                    <span class="text-white/70 text-sm">{{ appState.snippets.length }} snippet{{
                        appState.snippets.length === 1 ? '' : 's' }}</span>
                    <select v-model="appState.sortBy"
                        class="bg-white/20 text-white rounded px-2 py-1 text-sm border border-white/30">
                        <option value="newest">Newest First</option>
                        <option value="oldest">Oldest First</option>
                        <option value="title-asc">Title A-Z</option>
                        <option value="title-desc">Title Z-A</option>
                    </select>
                </div>
                <ul v-if="sortedSnippets.length" class="py-4 flex flex-col gap-4">
                    <li class="flex  flex-col md:flex-row items-start justify-between border border-gray-400/60 bg-[#fefae0] rounded-lg p-2"
                        v-for="(pair, key) in sortedSnippets" :key="pair.id">
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
                                    @click="appState.form.id = pair.id; appState.form.title = pair.title; appState.form.description = pair.description; appState.popupOpen = true">
                                    <IconsEdit class="h-6 w-6 md:h-8 md:w-8" />
                                </button>
                                <button title="Delete this item"
                                    class="bg-[#dd5e5e95] hover:bg-[#dd5e5e] text-white p-2 rounded-full "
                                    @click="confirmAction(pair.id as string)">
                                    <IconsDelete class="h-6 w-6 md:h-8 md:w-8" />
                                </button>
                            </div>
                        </div>
                    </li>
                </ul>
                <div v-if="!sortedSnippets.length" class="py-2">
                    You currently have no snippets saved.
                </div>
                <div class="flex  flex-col md:flex-row gap-2 items-center justify-between">
                    <button @click="openPopup"
                        class="bg-[#386c39] hover:bg-[#606c38] text-white rounded-lg px-4 py-2 md:w-auto w-full text-lg">Add
                        Item</button>
                    <button v-if="appState.snippets.length" @click="confirmAction('', true)"
                        class="bg-[#bd3f3c] hover:bg-[#bd693c] text-white rounded-lg px-4 text-lg py-2  md:w-auto w-full">Clear
                        Snippets</button>
                </div>
            </div>
        </div>
        <div v-if="appState.popupOpen" class="bg-gray-700/30 w-full h-full fixed inset-0" @click="closePopup"></div>
        <div v-if="appState.popupOpen"
            class="fixed bg-white border border-gray-400 max-w-[800px] w-[90vw] rounded-lg p-4 left-1/2 transform -translate-x-1/2 top-1/2 -translate-y-1/2 shadow-lg ">
            <div class="flex flex-col gap-2 w-full h-full">
                <h2 class="font-medium text-lg mb-4">{{ appState.form.title.length ? 'Edit Item' : 'Add Item' }}</h2>
                <input v-model="appState.form.title" placeholder="Title"
                    class="border border-gray-300 rounded-lg p-2 mb-2 w-full" />
                <textarea v-model="appState.form.description" placeholder="Description"
                    class="border border-gray-300 rounded-lg p-2 mb-2 w-full h-32 resize-none"></textarea>
                <button :disabled="appState.form.description.length === 0 || appState.form.title.length === 0"
                    @click="saveItem"
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
type Snippet = {
    id: string;
    title: string;
    description: string;
    createdAt: number;
}

type LegacySnippet = {
    title: string;
    description: string;
    createdAt?: number;
}

type SnippetInput = {
    id?: string;
} & LegacySnippet;

type SortOption = 'newest' | 'oldest' | 'title-asc' | 'title-desc';

const appState = useState<{
    popupOpen: boolean;
    textCopied: boolean;
    confirmOpen: boolean;
    confirmResult: { key?: string | null, confirmed: boolean | undefined, clearAll?: boolean };
    form: {
        id?: string;
        title: string;
        description: string;
    },
    snippets: Snippet[];
    sortBy: SortOption;
}>('appState', () => ({
    popupOpen: false,
    textCopied: false,
    confirmOpen: false,
    confirmResult: { key: null, confirmed: false },
    snippets: [],
    sortBy: 'newest' as SortOption,
    form: {
        title: '',
        description: ''
    }
}));
const timeout = ref<number | null>(null);
const importInput = ref<HTMLInputElement | null>(null);
function generateId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
}
function clearSnippets() {
    localStorage.removeItem('snippets');
    appState.value = { ...appState.value, snippets: [] };
}
function closePopup() {
    appState.value = { ...appState.value, popupOpen: false, form: { title: '', description: '' } };
}
function commitAction(confirmed: boolean) {
    appState.value = { ...appState.value, confirmOpen: false, confirmResult: appState.value.confirmResult };
    if (!confirmed) {
        return;
    }
    if (appState.value.confirmResult.clearAll) {
        clearSnippets();
        return;
    }
    if (typeof appState.value.confirmResult.key === 'string') {
        removeItem(appState.value.confirmResult.key as string);
    }
}
function confirmAction(id: string, clearAll = false) {
    appState.value = {
        ...appState.value,
        confirmOpen: true, confirmResult: {
            key: id, confirmed: undefined,
            clearAll
        }
    };
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
function exportSnippets() {
    if (!appState.value.snippets.length) return;
    const dataStr = JSON.stringify(appState.value.snippets, null, 2);
    const dataBlob = new Blob([dataStr], { type: 'application/json' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(dataBlob);
    link.download = `cs-export-${Date.now()}.json`;
    link.click();
    URL.revokeObjectURL(link.href);
}
function importSnippets(event: Event) {
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (e) => {
        try {
            const result = e.target?.result as string;
            const importedData: unknown = JSON.parse(result);
            if (Array.isArray(importedData) && importedData.every(isValidSnippetInput)) {
                const migratedData: Snippet[] = importedData.map((snippet: SnippetInput) => {
                    if (!snippet.id) {
                        return {
                            ...snippet,
                            id: generateId(),
                            createdAt: snippet.createdAt || Date.now()
                        };
                    }
                    return snippet as Snippet;
                });
                appState.value = { ...appState.value, snippets: migratedData };
                localStorage.setItem('snippets', JSON.stringify(migratedData));
            } else {
                alert('Invalid file format. Please select a valid JSON file with snippet data.');
            }
        } catch (error) {
            alert('Error reading file. Please make sure it\'s a valid JSON file.');
        }
    };
    reader.readAsText(file);
    input.value = '';
}
function isValidSnippetInput(obj: unknown): obj is SnippetInput {
    return (
        typeof obj === 'object' &&
        obj !== null &&
        typeof (obj as SnippetInput).title === 'string' &&
        typeof (obj as SnippetInput).description === 'string' &&
        (typeof (obj as SnippetInput).createdAt === 'number' || typeof (obj as SnippetInput).createdAt === 'undefined') &&
        (typeof (obj as SnippetInput).id === 'string' || typeof (obj as SnippetInput).id === 'undefined')
    );
}
function openPopup() {
    appState.value = { ...appState.value, popupOpen: true };
}
function removeItem(id: string, clearAll = false) {
    if (clearAll) {
        clearSnippets();
        return;
    }
    const currentSnippets = appState.value.snippets.filter((item) => item.id !== id);
    appState.value = { ...appState.value, snippets: currentSnippets };
    localStorage.setItem('snippets', JSON.stringify(currentSnippets));
}
function saveItem() {
    if (appState.value.form.id) {
        const currentItem = appState.value.snippets.findIndex((item) => item.id === appState.value.form.id);
        if (currentItem !== -1 && appState.value.snippets[currentItem]) {
            appState.value.snippets[currentItem].title = appState.value.form.title;
            appState.value.snippets[currentItem].description = appState.value.form.description;
            appState.value = { ...appState.value, snippets: appState.value.snippets };
            localStorage.setItem('snippets', JSON.stringify(appState.value.snippets));
            closePopup();
            return;
        }
    }
    const newSnippet = {
        id: generateId(),
        title: appState.value.form.title,
        description: appState.value.form.description,
        createdAt: Date.now()
    };
    appState.value = {
        ...appState.value,
        snippets: [...appState.value.snippets, newSnippet]
    };
    localStorage.setItem('snippets', JSON.stringify(appState.value.snippets));
    closePopup();
}
function triggerImport() {
    importInput.value?.click();
}
const sortedSnippets = computed((): Snippet[] => {
    const snippets = [...appState.value.snippets];
    switch (appState.value.sortBy) {
        case 'newest':
            return snippets.sort((a, b) => b.createdAt - a.createdAt);
        case 'oldest':
            return snippets.sort((a, b) => a.createdAt - b.createdAt);
        case 'title-asc':
            return snippets.sort((a, b) => a.title.localeCompare(b.title));
        case 'title-desc':
            return snippets.sort((a, b) => b.title.localeCompare(a.title));
        default:
            return snippets;
    }
});
onMounted(() => {
    const rawSnippets = localStorage.getItem('snippets');
    if (rawSnippets) {
        try {
            const parsedSnippets: unknown = JSON.parse(rawSnippets);
            if (Array.isArray(parsedSnippets) && parsedSnippets.every(isValidSnippetInput)) {
                // Handle migration from old format without IDs
                const migratedSnippets: Snippet[] = parsedSnippets.map((snippet: SnippetInput) => {
                    if (!snippet.id) {
                        return {
                            ...snippet,
                            id: generateId(),
                            createdAt: snippet.createdAt || Date.now()
                        };
                    }
                    return snippet as Snippet;
                });
                appState.value.snippets = migratedSnippets;

                // Save migrated data back to localStorage if migration occurred
                const needsMigration = migratedSnippets.some((migrated: Snippet) =>
                    !parsedSnippets.find((original: SnippetInput) => original.id === migrated.id)
                );
                if (needsMigration) {
                    localStorage.setItem('snippets', JSON.stringify(migratedSnippets));
                }
            } else {
                console.warn('Invalid snippet data in localStorage, clearing storage');
                localStorage.removeItem('snippets');
            }
        } catch (error) {
            console.error('Failed to parse snippets from localStorage:', error);
            localStorage.removeItem('snippets');
        }
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