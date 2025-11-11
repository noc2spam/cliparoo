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

                <div v-if="appState.alertOpen"
                    class="fixed z-30 bg-red-500 py-2 px-4 text-white top-0 text-center font-medium w-full left-0 animate-pulse">
                    {{ appState.alertMessage }}
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
                            <div class="flex items-center gap-2 mb-1">
                                <h2 class="text-lg font-medium">{{ pair.title }}</h2>
                                <span v-if="pair.isProtected"
                                    class="text-xs bg-yellow-200 text-yellow-800 px-2 py-1 rounded-full">🔐
                                    Protected</span>
                            </div>
                            <div v-if="!pair.isProtected" v-for="(line, index) in pair.description.split('\n')"
                                :key="`desc-${index}`">{{ line }}</div>
                            <div v-else-if="appState.unlockedSnippets.has(pair.id)"
                                v-for="(line, index) in getDecryptedContent(pair.id).split('\n')"
                                :key="`decr-${index}`">{{ line }}</div>
                            <div v-else class="text-gray-500 italic">Content is password protected. Click view or copy
                                to unlock.</div>
                        </div>
                        <div class="h-full col-span-2 grid place-items-start pt-3">
                            <div class="flex  gap-2">
                                <button title="View this item"
                                    class="text-white bg-[#9b59b6] hover:bg-[#8e44ad] rounded-full p-2"
                                    @click="handleView(pair)">
                                    <IconsView class="h-6 w-6 md:h-8 md:w-8" />
                                </button>
                                <button title="Copy this item to clipboard"
                                    class="text-white bg-[#77dd5e95] hover:bg-[#77dd5e] rounded-full p-2"
                                    @click="handleCopy(pair)">
                                    <IconsCopy class="h-6 w-6 md:h-8 md:w-8" />
                                </button>
                                <button title="Edit this item"
                                    class="bg-[#5eb5dd95] hover:bg-[#5eb5dd] text-white p-2 rounded-full "
                                    @click="handleEdit(pair)">
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
                <h2 class="font-medium text-lg mb-4">{{ appState.form.id?.length ? 'Edit Item' : 'Add Item' }}</h2>
                <input v-model="appState.form.title" placeholder="Title"
                    class="border border-gray-300 rounded-lg p-2 mb-2 w-full" />
                <textarea v-model="appState.form.description" placeholder="Description"
                    class="border border-gray-300 rounded-lg p-2 mb-2 w-full h-32 resize-none"></textarea>
                <div class="flex items-center gap-2 mb-2">
                    <input v-model="appState.form.isProtected" type="checkbox" id="isProtected" class="rounded" />
                    <label for="isProtected" class="text-sm text-gray-700">🔐 Password protect this snippet</label>
                </div>

                <div v-if="appState.form.isProtected" class="mb-2">
                    <input v-model="appState.form.password" type="password" placeholder="Enter password for protection"
                        class="border border-gray-300 rounded-lg p-2 w-full" />
                </div>

                <button
                    :disabled="appState.form.description.length === 0 || appState.form.title.length === 0 || (appState.form.isProtected && !appState.form.password.trim())"
                    @click="saveItem"
                    class="bg-blue-700 disabled:opacity-60 text-white rounded-lg px-4 py-2">Save</button>
                <button @click="closePopup" class="bg-gray-300 text-gray-800 rounded-lg px-4 py-2">Cancel</button>
            </div>
        </div>
        <div v-if="appState.passwordPromptOpen" class="bg-gray-700/30 w-full h-full fixed inset-0"
            @click="closePasswordPrompt"></div>
        <div v-if="appState.passwordPromptOpen"
            class="fixed bg-white border border-gray-400 max-w-[400px] w-[90vw] rounded-lg p-4 left-1/2 transform -translate-x-1/2 top-1/2 -translate-y-1/2 shadow-lg">
            <div class="flex flex-col gap-2 w-full h-full">
                <h2 class="font-medium text-lg mb-4">🔐 Password Required</h2>
                <p class="text-sm text-gray-600 mb-4">This snippet is password protected. Enter the password to {{
                    appState.passwordPrompt.action }}:</p>
                <input v-model="appState.passwordPrompt.password" type="password" placeholder="Enter password"
                    @keyup.enter="submitPassword" class="border border-gray-300 rounded-lg p-2 mb-2 w-full" />
                <button @click="submitPassword" :disabled="!appState.passwordPrompt.password.trim()"
                    class="bg-blue-700 disabled:opacity-60 text-white rounded-lg px-4 py-2">Submit</button>
                <button @click="closePasswordPrompt"
                    class="bg-gray-300 text-gray-800 rounded-lg px-4 py-2">Cancel</button>
            </div>
        </div>

        <div v-if="appState.viewPopupOpen" class="bg-gray-700/30 w-full h-full fixed inset-0" @click="closeViewPopup">
        </div>
        <div v-if="appState.viewPopupOpen"
            class="fixed bg-white border border-gray-400 max-w-[800px] w-[90vw] rounded-lg p-4 left-1/2 transform -translate-x-1/2 top-1/2 -translate-y-1/2 shadow-lg">
            <div class="flex flex-col gap-2 w-full h-full">
                <h2 class="font-medium text-lg mb-4">👁️ View Snippet</h2>
                <div class="mb-4">
                    <h3 class="font-semibold text-gray-800 mb-2">{{ appState.viewSnippet.title }}</h3>
                    <div class="bg-gray-50 border border-gray-200 rounded-lg p-4 max-h-96 overflow-y-auto">
                        <pre class="whitespace-pre-wrap text-sm">{{ appState.viewSnippet.description }}</pre>
                    </div>
                </div>
                <button @click="copyToClipboard(appState.viewSnippet.description)"
                    class="bg-green-600 hover:bg-green-700 text-white rounded-lg px-4 py-2">Copy to Clipboard</button>
                <button @click="closeViewPopup" class="bg-gray-300 text-gray-800 rounded-lg px-4 py-2">Close</button>
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
    isProtected?: boolean;
    encryptedData?: string;
}
type SortOption = 'newest' | 'oldest' | 'title-asc' | 'title-desc';
const appState = useState<{
    popupOpen: boolean;
    viewPopupOpen: boolean;
    textCopied: boolean;
    confirmOpen: boolean;
    passwordPromptOpen: boolean;
    alertOpen: boolean;
    alertMessage: string;
    confirmResult: { key?: string | null, confirmed: boolean | undefined, clearAll?: boolean };
    passwordPrompt: {
        snippetId: string | null;
        action: 'view' | 'edit' | null;
        password: string;
    };
    viewSnippet: {
        title: string;
        description: string;
    };
    form: {
        id?: string;
        title: string;
        description: string;
        isProtected: boolean;
        password: string;
    },
    snippets: Snippet[];
    sortBy: SortOption;
    unlockedSnippets: Set<string>;
    decryptedCache: Map<string, string>;
}>('appState', () => ({
    popupOpen: false,
    viewPopupOpen: false,
    textCopied: false,
    confirmOpen: false,
    passwordPromptOpen: false,
    alertOpen: false,
    alertMessage: '',
    confirmResult: { key: null, confirmed: false },
    passwordPrompt: {
        snippetId: null,
        action: null,
        password: ''
    },
    viewSnippet: {
        title: '',
        description: ''
    },
    snippets: [],
    sortBy: 'newest' as SortOption,
    unlockedSnippets: new Set(),
    decryptedCache: new Map(),
    form: {
        title: '',
        description: '',
        isProtected: false,
        password: ''
    }
}));
const timeout = ref<number | null>(null);
const importInput = ref<HTMLInputElement | null>(null);
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
function clearSnippets() {
    localStorage.removeItem('snippets');
    appState.value = { ...appState.value, snippets: [] };
}
function closeViewPopup() {
    appState.value.viewPopupOpen = false;
    appState.value.viewSnippet = { title: '', description: '' };
}
function closePasswordPrompt() {
    appState.value = {
        ...appState.value,
        passwordPromptOpen: false,
        passwordPrompt: {
            snippetId: null,
            action: null,
            password: ''
        }
    };
}
function closePopup() {
    appState.value = {
        ...appState.value,
        popupOpen: false,
        form: {
            title: '',
            description: '',
            isProtected: false,
            password: ''
        }
    };
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
function decrypt(encryptedText: string, password: string): string {
    try {
        const parts = encryptedText.split('|');
        if (parts.length !== 2) {
            throw new Error('Invalid encrypted data format');
        }
        const [encryptedData, validationHash] = parts;
        if (!encryptedData || !validationHash) {
            throw new Error('Invalid encrypted data format');
        }
        const key = password.split('').map(char => char.charCodeAt(0));
        const decoded = atob(encryptedData);
        const decrypted = decoded.split('').map((char, i) => {
            const keyValue = key[i % key.length] || 0;
            return String.fromCharCode(char.charCodeAt(0) ^ keyValue);
        }).join('');
        const expectedHash = btoa(decrypted.substring(0, 10) + password.length);
        if (expectedHash !== validationHash) {
            throw new Error('Invalid password');
        }
        return decrypted;
    } catch (error) {
        throw new Error('Invalid password or corrupted data');
    }
}
function encrypt(text: string, password: string): string {
    const key = password.split('').map(char => char.charCodeAt(0));
    const encrypted = text.split('').map((char, i) => {
        const keyValue = key[i % key.length] || 0;
        return String.fromCharCode(char.charCodeAt(0) ^ keyValue);
    }).join('');
    const validationHash = btoa(text.substring(0, 10) + password.length);
    return btoa(encrypted) + '|' + validationHash;
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
function generateId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
}
function getDecryptedContent(snippetId: string): string {
    const snippet = appState.value.snippets.find(s => s.id === snippetId);
    if (snippet && snippet.isProtected && appState.value.unlockedSnippets.has(snippetId)) {
        const cachedContent = appState.value.decryptedCache.get(snippetId);
        if (cachedContent) {
            return cachedContent;
        }
    }
    return snippet?.description || '';
}
function handleCopy(snippet: Snippet) {
    if (snippet.isProtected && !appState.value.unlockedSnippets.has(snippet.id)) {
        const password = window.prompt('Enter password to copy this snippet:');
        if (password) {
            try {
                const decryptedContent = decrypt(snippet.encryptedData || '', password);
                appState.value.unlockedSnippets.add(snippet.id);
                appState.value.decryptedCache.set(snippet.id, decryptedContent);
                copyToClipboard(decryptedContent);
            } catch (error) {
                showAlert('Incorrect password!');
            }
        }
    } else {
        const contentToCopy = snippet.isProtected
            ? getDecryptedContent(snippet.id)
            : snippet.description;
        copyToClipboard(contentToCopy);
    }
}
function handleEdit(snippet: Snippet) {
    if (snippet.isProtected && !appState.value.unlockedSnippets.has(snippet.id)) {
        const password = window.prompt('Enter password to edit this snippet:');
        if (password) {
            try {
                const decryptedContent = decrypt(snippet.encryptedData || '', password);
                appState.value.unlockedSnippets.add(snippet.id);
                appState.value.decryptedCache.set(snippet.id, decryptedContent);
                appState.value.form = {
                    id: snippet.id,
                    title: snippet.title,
                    description: decryptedContent,
                    isProtected: true,
                    password: password
                };
                appState.value.popupOpen = true;
            } catch (error) {
                showAlert('Incorrect password!');
            }
        }
    } else {
        const contentToEdit = snippet.isProtected
            ? getDecryptedContent(snippet.id)
            : snippet.description;
        appState.value.form = {
            id: snippet.id,
            title: snippet.title,
            description: contentToEdit,
            isProtected: snippet.isProtected || false,
            password: ''
        };
        appState.value.popupOpen = true;
    }
}
function handleView(snippet: Snippet) {
    if (snippet.isProtected && !appState.value.unlockedSnippets.has(snippet.id)) {
        const password = window.prompt('Enter password to view this snippet:');
        if (password) {
            try {
                const decryptedContent = decrypt(snippet.encryptedData || '', password);
                appState.value.unlockedSnippets.add(snippet.id);
                appState.value.decryptedCache.set(snippet.id, decryptedContent);
                appState.value.viewSnippet = {
                    title: snippet.title,
                    description: decryptedContent
                };
                appState.value.viewPopupOpen = true;
            } catch (error) {
                showAlert('Incorrect password!');
            }
        }
    } else {
        appState.value.viewSnippet = {
            title: snippet.title,
            description: snippet.isProtected ? getDecryptedContent(snippet.id) : snippet.description
        };
        appState.value.viewPopupOpen = true;
    }
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
                const migratedData: Snippet[] = importedData.map((snippet: Snippet) => {
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
                showAlert('Invalid file format. Please select a valid JSON file with snippet data.');
            }
        } catch (error) {
            showAlert('Error reading file. Please make sure it\'s a valid JSON file.');
        }
    };
    reader.readAsText(file);
    input.value = '';
}
function isValidSnippetInput(obj: unknown): obj is Snippet {
    return (
        typeof obj === 'object' &&
        obj !== null &&
        typeof (obj as Snippet).title === 'string' &&
        typeof (obj as Snippet).description === 'string' &&
        (typeof (obj as Snippet).createdAt === 'number' || typeof (obj as Snippet).createdAt === 'undefined') &&
        (typeof (obj as Snippet).id === 'string' || typeof (obj as Snippet).id === 'undefined') &&
        (typeof (obj as Snippet).isProtected === 'boolean' || typeof (obj as Snippet).isProtected === 'undefined') &&
        (typeof (obj as Snippet).encryptedData === 'string' || typeof (obj as Snippet).encryptedData === 'undefined')
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
    const { form } = appState.value;
    if (form.id) {
        const currentItem = appState.value.snippets.findIndex((item) => item.id === form.id);
        if (currentItem !== -1 && appState.value.snippets[currentItem]) {
            const snippet = appState.value.snippets[currentItem];
            snippet.title = form.title;
            snippet.isProtected = form.isProtected;

            if (form.isProtected) {
                snippet.encryptedData = encrypt(form.description, form.password);
                snippet.description = '';
            } else {
                snippet.description = form.description;
                snippet.encryptedData = undefined;
            }
            appState.value = { ...appState.value, snippets: appState.value.snippets };
            localStorage.setItem('snippets', JSON.stringify(appState.value.snippets));
            closePopup();
            return;
        }
    }
    const newSnippet: Snippet = {
        id: generateId(),
        title: form.title,
        description: form.isProtected ? '' : form.description,
        createdAt: Date.now(),
        isProtected: form.isProtected,
        encryptedData: form.isProtected ? encrypt(form.description, form.password) : undefined
    };
    appState.value = {
        ...appState.value,
        snippets: [...appState.value.snippets, newSnippet]
    };
    localStorage.setItem('snippets', JSON.stringify(appState.value.snippets));
    closePopup();
}
function showAlert(message: string) {
    appState.value.alertMessage = message;
    appState.value.alertOpen = true;
    if (timeout.value) {
        clearTimeout(timeout.value);
    }
    timeout.value = setTimeout(() => {
        appState.value.alertOpen = false;
    }, 3000);
}
function submitPassword() {
    const { snippetId, action, password } = appState.value.passwordPrompt;
    if (!snippetId || !action || !password) return;
    const snippet = appState.value.snippets.find(s => s.id === snippetId);
    if (!snippet || !snippet.isProtected || !snippet.encryptedData) return;
    try {
        const decryptedDescription = decrypt(snippet.encryptedData, password);

        if (action === 'view') {
            copyToClipboard(decryptedDescription);
        } else if (action === 'edit') {
            appState.value.form = {
                id: snippet.id,
                title: snippet.title,
                description: decryptedDescription,
                isProtected: snippet.isProtected,
                password: password
            };
            appState.value.popupOpen = true;
        }

        closePasswordPrompt();
    } catch (error) {
        showAlert('Invalid password!');
        appState.value.passwordPrompt.password = '';
    }
}
function triggerImport() {
    importInput.value?.click();
}
onMounted(() => {
    const rawSnippets = localStorage.getItem('snippets');
    if (rawSnippets) {
        try {
            const parsedSnippets: unknown = JSON.parse(rawSnippets);
            if (Array.isArray(parsedSnippets) && parsedSnippets.every(isValidSnippetInput)) {
                const migratedSnippets: Snippet[] = parsedSnippets.map((snippet: Snippet) => {
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
                const needsMigration = migratedSnippets.some((migrated: Snippet) =>
                    !parsedSnippets.find((original: Snippet) => original.id === migrated.id)
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