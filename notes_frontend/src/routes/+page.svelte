<script lang="ts">
    import { onMount } from 'svelte';
    import { notes } from '$lib/stores/notes';
    import { auth } from '$lib/stores/auth';
    import type { Note } from '$lib/services/api';

    let selectedId: string | null = null;
    let selected: Note | null = null;
    let search = '';
    let saving = false;
    let savingError: string | null = null;

    const unsubscribe = notes.subscribe((s) => {
        if (selectedId) {
            selected = s.items.find((n) => n.id === selectedId) ?? null;
        }
    });

    onMount(() => {
        // load notes if authenticated
        if ($auth.token) {
            notes.refresh();
        }
        return () => unsubscribe();
    });

    function choose(note: Note) {
        selectedId = note.id;
        selected = note;
    }

    async function newNote() {
        const created = await notes.createEmpty();
        choose(created);
    }

    let titleDraft = '';
    let contentDraft = '';

    $: if (selected) {
        titleDraft = selected.title;
        contentDraft = selected.content;
    }

    async function saveDraft() {
        if (!selected) return;
        saving = true;
        savingError = null;
        try {
            await notes.save(selected.id, { title: titleDraft, content: contentDraft });
        } catch (e) {
            savingError = (e as Error).message;
        } finally {
            saving = false;
        }
    }

    async function removeNote() {
        if (!selected) return;
        const id = selected.id;
        await notes.remove(id);
        if (selectedId === id) {
            selectedId = null; selected = null;
        }
    }

    let searchDebounce: ReturnType<typeof setTimeout> | undefined;
    function onSearchInput(e: Event) {
        const target = e.target as HTMLInputElement | null;
        search = target?.value ?? '';
        if (searchDebounce) clearTimeout(searchDebounce);
        searchDebounce = setTimeout(() => notes.refresh(search), 250);
    }

    // helper
    function s(n: number) { return n === 1 ? '' : 's'; }
</script>

<svelte:head>
    <title>Notes</title>
</svelte:head>

{#if !$auth.token}
    <div class="guest card">
        <div>
            <h2>Welcome to Notes</h2>
            <p class="text-muted">Sign in to create and manage your notes.</p>
        </div>
        <a class="btn btn-primary" href="/auth">Sign in</a>
    </div>
{:else}
<div class="layout">
    <section class="list card">
        <div class="list-top">
            <input placeholder="Search notes…" oninput={onSearchInput} />
            <button class="btn btn-accent" onclick={newNote} title="New note">New</button>
        </div>

        {#if $notes.loading}
            <div class="empty">Loading…</div>
        {:else if $notes.error}
            <div class="empty error">{$notes.error}</div>
        {:else if $notes.items.length === 0}
            <div class="empty">No notes yet. Create your first note!</div>
        {:else}
            <div class="items">
                {#each $notes.items as n (n.id)}
                    <button
                        class="item"
                        class:active={selectedId === n.id}
                        onclick={() => choose(n)}
                        title={n.title}
                    >
                        <div class="item-title text-ellipsis">{n.title || 'Untitled'}</div>
                        <div class="item-meta">
                            <span class="text-muted">{new Date(n.updatedAt).toLocaleString()}</span>
                            {#if n.tags?.length}
                                <span class="tag">{n.tags.length} tag{s(n.tags.length)}</span>
                            {/if}
                        </div>
                    </button>
                {/each}
            </div>
        {/if}
    </section>

    <section class="editor card">
        {#if !selected}
            <div class="placeholder">
                <div class="ghost"></div>
                <p class="text-muted">Select a note from the list or create a new one.</p>
            </div>
        {:else}
            <div class="editor-head">
                <input
                    class="title-input"
                    placeholder="Title"
                    bind:value={titleDraft}
                    onblur={saveDraft}
                />
                <div class="actions">
                    <button class="btn btn-outline" onclick={saveDraft} disabled={saving}>Save</button>
                    <button class="btn btn-danger" onclick={removeNote}>Delete</button>
                </div>
            </div>

            {#if savingError}
                <div class="save-error">{savingError}</div>
            {/if}

            <textarea
                class="content"
                placeholder="Write your note here..."
                bind:value={contentDraft}
                onblur={saveDraft}
                rows={18}
            />
        {/if}
    </section>
</div>
{/if}

<style>
.layout {
    display: grid;
    grid-template-columns: minmax(240px, var(--list-width)) 1fr;
    gap: 18px;
    height: calc(100dvh - 36px);
}
.list, .editor { padding: 12px; display: flex; flex-direction: column; min-height: 0; }
.list-top {
    display: flex; gap: 10px; align-items: center; padding: 6px;
    border-bottom: 1px solid var(--color-border); margin: -12px -12px 8px -12px;
}
.list-top input { flex: 1; }
.items { display: flex; flex-direction: column; gap: 6px; overflow: auto; padding: 6px; }
.item {
    display: flex; flex-direction: column; align-items: flex-start; gap: 4px;
    padding: 10px 12px; border: 1px solid var(--color-border); border-radius: var(--radius-sm);
    background: #fff; cursor: pointer; text-align: left;
}
.item:hover { box-shadow: var(--shadow-sm); }
.item.active { outline: 2px solid #e8f2fd; border-color: #d7e9fd; }
.item-title { font-weight: 600; }
.item-meta { display: flex; gap: 10px; align-items: center; font-size: 12px; }

.editor .editor-head {
    display: flex; gap: 10px; align-items: center; padding: 8px;
    border-bottom: 1px solid var(--color-border); margin: -12px -12px 8px -12px;
}
.title-input { font-size: 18px; font-weight: 600; }
.content {
    flex: 1; resize: vertical; min-height: 300px; margin: 6px 0 0 0;
}
.placeholder { height: 100%; display: grid; place-items: center; }
.placeholder .ghost {
    width: 60px; height: 60px; border-radius: 16px; background: var(--color-surface);
    border: 1px dashed var(--color-border); margin-bottom: 10px;
}

.guest {
    margin: 30px auto;
    max-width: 720px;
    padding: 20px;
    display: flex; align-items: center; justify-content: space-between;
}
.guest h2 { margin: 0 0 4px 0; }

.save-error {
    background: #fff3e0;
    color: #6a4f00;
    border: 1px solid #ffe0b2;
    border-radius: var(--radius-sm);
    padding: 8px 10px;
    font-size: 13px;
    margin: 6px 0;
}

@media (max-width: 980px) {
    .layout { grid-template-columns: 1fr; }
    .editor { min-height: 50dvh; }
}
</style>
