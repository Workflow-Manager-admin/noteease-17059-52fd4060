<script>
  // Color scheme
  const COLOR_PRIMARY = '#4A90E2';
  const COLOR_SECONDARY = '#FFFFFF';
  const COLOR_ACCENT = '#F5A623';
  const CATEGORY_COLORS = ['#F5A623', '#50E3C2', '#B8E986', '#F8E71C', '#D0021B', '#9B9B9B'];

  // Note shape: { id, title, content, categories, created, updated }
  let notes = [];
  let search = '';
  let showModal = false;
  let currentNote = { id: null, title: '', content: '', categories: [] };
  let editing = false;
  let categoryInput = '';
  let uniqueId = 1; // For demo; normally use uuid

  // Load from localStorage on mount
  onMount(() => {
    const saved = localStorage.getItem('noteease-notes');
    if (saved) {
      notes = JSON.parse(saved);
      uniqueId = Math.max(1, ...notes.map(n => n.id)) + 1;
    }
  });

  // Save to localStorage whenever notes change
  $: localStorage.setItem('noteease-notes', JSON.stringify(notes));

  // PUBLIC_INTERFACE
  function addNote() {
    currentNote = { id: null, title: '', content: '', categories: [] };
    editing = false;
    categoryInput = '';
    showModal = true;
  }

  // PUBLIC_INTERFACE
  function editNote(note) {
    currentNote = { ...note };
    editing = true;
    categoryInput = '';
    showModal = true;
  }

  // PUBLIC_INTERFACE
  function saveNote() {
    if (!currentNote.title.trim() && !currentNote.content.trim()) {
      showModal = false;
      return;
    }
    if (editing) {
      notes = notes.map((n) => n.id === currentNote.id ? { ...currentNote, updated: new Date() } : n);
    } else {
      notes = [
        { ...currentNote, id: uniqueId++, created: new Date(), updated: new Date() },
        ...notes
      ];
    }
    showModal = false;
  }

  // PUBLIC_INTERFACE
  function deleteNote(id) {
    if (confirm('Delete this note?')) {
      notes = notes.filter(n => n.id !== id);
    }
  }

  // PUBLIC_INTERFACE
  function addCategory() {
    const val = categoryInput.trim();
    if (val && !currentNote.categories.includes(val)) {
      currentNote = { ...currentNote, categories: [...currentNote.categories, val] };
    }
    categoryInput = '';
  }

  // PUBLIC_INTERFACE
  function removeCategory(cat) {
    currentNote = { ...currentNote, categories: currentNote.categories.filter(c => c !== cat) };
  }

  // PUBLIC_INTERFACE
  function onEditorKey(event) {
    // Support basic text formatting via toolbar or keyboard shortcuts
    // (bold/italic via buttons below)
    return true;
  }

  // Text formatting functions
  function formatSelected(tag) {
    let textarea = document.getElementById('note-content');
    if (!textarea) return;
    const start = textarea.selectionStart, end = textarea.selectionEnd;
    let before = currentNote.content.slice(0, start);
    let selected = currentNote.content.slice(start, end);
    let after = currentNote.content.slice(end);

    let openTag, closeTag;
    if (tag === 'b') { openTag = '<b>'; closeTag = '</b>'; }
    if (tag === 'i') { openTag = '<i>'; closeTag = '</i>'; }
    if (tag === 'ul') {
      // For bullet, surround line with <ul><li>..</li></ul>
      openTag = '<ul><li>'; closeTag = '</li></ul>';
      // If multi-line, wrap each line in <li>
      let lines = selected.split('\n').map(line => line ? `<li>${line}</li>` : '').join('');
      selected = `<ul>${lines}</ul>`;
      currentNote.content = before + selected + after;
      return;
    }
    // Apply tag
    currentNote.content = before + openTag + selected + closeTag + after;
  }

  // PUBLIC_INTERFACE
  function filteredNotes() {
    if (!search.trim()) return notes;
    const s = search.trim().toLowerCase();
    return notes.filter(n =>
      n.title.toLowerCase().includes(s) ||
      n.content.toLowerCase().includes(s) ||
      (n.categories && n.categories.some(cat => cat.toLowerCase().includes(s)))
    );
  }

  // PUBLIC_INTERFACE
  function snippet(text) {
    // Return plain text, max 60 chars
    const s = text.replace(/<[^>]+>/g, '');
    return s.length > 65 ? s.substring(0, 65) + '…' : s;
  }

  import { onMount } from 'svelte';
</script>

<style>
  :global(body) {
    background: #f8faff;
    color: #222;
    font-family: 'Inter', 'Segoe UI', Arial, sans-serif;
    margin: 0;
  }
  .main {
    max-width: 700px;
    margin: 2em auto;
    background: {COLOR_SECONDARY};
    border-radius: 20px;
    box-shadow: 0 4px 24px 0 rgba(74,144,226, 0.11), 0 1.5px 1.5px 0 rgba(0,0,0,0.02);
    padding-bottom: 80px;
    min-height: 70vh;
    position: relative;
    overflow: visible;
  }
  .header-bar {
    background: {COLOR_PRIMARY};
    padding: 1.4em 2em 1em 2em;
    border-radius: 20px 20px 0 0;
    color: white;
    display: flex;
    flex-direction: column;
    gap: 0.5em;
    position: sticky;
    top: 0;
    z-index: 9;
  }
  .search-bar {
    background: rgba(255,255,255,0.9);
    border-radius: 8px;
    border: none;
    padding: 0.7em 1em;
    font-size: 1em;
    margin-top: 0.2em;
    outline: none;
    color: #333;
  }
  .notes-list {
    padding: 1.6em 2em 1em 2em;
    display: flex;
    flex-direction: column;
    gap: 1em;
  }
  .note-preview {
    cursor: pointer;
    background: #f2faff;
    border-radius: 13px;
    padding: 1em 1.2em;
    border: 1.8px solid #e4eefe;
    transition: box-shadow 0.15s, border 0.15s;
    box-shadow: 0 1px 4px 0 rgba(74,144,226,0.025);
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 1.2em;
  }
  .note-preview:hover {
    border-color: {COLOR_PRIMARY};
    background: #e6f1fc;
    box-shadow: 0 1.5px 8px 0 rgba(74,144,226,0.085);
  }
  .note-title {
    font-weight: bold;
    font-size: 1.08em;
    margin-bottom: 0.25em;
  }
  .note-categories {
    margin-bottom: 2px;
    display: flex;
    gap: 0.3em;
    flex-wrap: wrap;
  }
  .category-label {
    font-size: 0.85em;
    background: {COLOR_ACCENT};
    color: #fff;
    border-radius: 7px;
    padding: 2.5px 10px;
    letter-spacing: 0.02em;
    margin-right: 3px;
    margin-bottom: 2px;
    user-select: none;
    border: none;
    outline: none;
    display: inline-block;
  }
  .note-snippet {
    color: #5b6d8c;
    font-size: 0.99em;
    margin-top: 1px;
    max-width: 370px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  .fab {
    position: fixed;
    right: 3vw;
    bottom: 6vh;
    width: 62px;
    height: 62px;
    background: {COLOR_PRIMARY};
    color: white;
    display: flex;
    align-items: center;
    font-size: 2.3em;
    justify-content: center;
    box-shadow: 0 4px 12px rgba(74,144,226,0.22), 0 1.5px 1.5px 0 rgba(0,0,0,0.06);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    z-index: 100;
    transition: background 0.15s;
  }
  .fab:hover {
    background: #3071ba;
  }
  .modal-bg {
    background: rgba(24,50,85,0.20);
    position: fixed;
    z-index: 99;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .modal {
    background: {COLOR_SECONDARY};
    min-width: 330px;
    max-width: 97vw;
    border-radius: 15px;
    box-shadow: 0 8px 48px 0 rgba(74,144,226,0.21), 0 1.5px 1.5px 0 rgba(0,0,0,0.04);
    padding: 1.6em 2.3em;
    position: relative;
    display: flex;
    flex-direction: column;
    gap: 1.15em;
  }
  .modal h2 {
    margin-bottom: 0.2em;
    color: {COLOR_PRIMARY};
    font-weight: bold;
    font-size: 1.17em;
  }
  .close-btn {
    position: absolute;
    right: 1em; top: 1em;
    background: transparent;
    color: #aaa;
    border: none;
    font-size: 1.8em;
    cursor: pointer;
    z-index: 120;
  }
  .modal label {
    font-weight: 500;
    color: #426087;
    font-size: 0.92em;
    margin-bottom: 0.25em;
    margin-right: 4px;
    margin-left: 1px;
  }
  .modal input[type="text"], .modal textarea {
    width: 99%; font-size: 1em;
    border: 1.5px solid #e7ebf3;
    padding: 7px 9px;
    border-radius: 6px;
    background: #f9fafb;
    margin-bottom: 0.5em;
    color: #15386b;
    outline: none;
    transition: border 0.15s;
  }
  .modal input[type="text"]:focus, .modal textarea:focus {
    border: 1.5px solid {COLOR_PRIMARY};
  }
  .modal .toolbar button {
    margin-right: 7px;
    background: {COLOR_ACCENT};
    color: #fff;
    border: none;
    border-radius: 6px;
    font-size: 1.08em;
    padding: 2px 10px;
    cursor: pointer;
    transition: background 0.11s;
  }
  .modal .toolbar button:hover { background: #f09718; }
  .modal .category-input-group {
    display: flex; gap: 5px; align-items: center;
    margin-bottom: 2px;
  }
  .modal .note-categories .category-label {
    background: {COLOR_PRIMARY};
    font-weight: 600;
    color: #fff;
  }
  .category-remove {
    cursor: pointer; color: #c33;
    font-weight: bold;
    margin-left: 3.5px;
    font-size: 12px;
    background: none; border: none;
    vertical-align: middle;
    transition: color 0.12s;
  }
  .category-remove:hover { color: #D0021B; }
  .modal .actions {
    display: flex; gap: 1em; justify-content: flex-end;
    margin-top: 5px;
    margin-bottom: -0.2em;
  }
  .modal .actions button {
    background: {COLOR_PRIMARY};
    color: #fff;
    border: none;
    border-radius: 7px;
    padding: 9px 18px;
    font-weight: bold;
    cursor: pointer;
    font-size: 1.01em;
    transition: background 0.14s;
  }
  .modal .actions button.delete {
    background: #ff5555;
    margin-right: auto;
  }
  .modal .actions button.delete:hover {
    background: #d0021b;
  }
  .modal .actions button.cancel {
    background: #b4c0d6;
    color: #fff;
  }
  .modal .actions button.cancel:hover {
    background: #7891c9;
  }
</style>

<div class="main" style="background: {COLOR_SECONDARY};">
  <div class="header-bar" style="background: {COLOR_PRIMARY};">
    <div style="font-size:1.23em; font-weight: bold; letter-spacing: -.02em; line-height:1.3;">
      NoteEase
    </div>
    <input
      class="search-bar"
      placeholder="Search notes, content, or tags…"
      bind:value={search}
      autocomplete="off"
      spellcheck="false"
      aria-label="Search notes"
    />
  </div>
  <div class="notes-list">
    {#if filteredNotes().length === 0}
      <div style="margin:2.5em 0 3em 0;text-align:center; opacity:0.44">
        <span style="font-size:2.4em;">📝</span>
        <div style="margin-top:0.6em;">No notes found.</div>
      </div>
    {/if}
    {#each filteredNotes() as note}
      <div class="note-preview" on:click={() => editNote(note)}>
        <div style="flex:1;">
          <div class="note-title">{note.title || <span style="color:#aaa; font-weight:normal;">Untitled note</span>}</div>
          <div class="note-categories">
            {#if note.categories}
              {#each note.categories as cat, i}
                <span class="category-label" style="background:{CATEGORY_COLORS[i % CATEGORY_COLORS.length]};">{cat}</span>
              {/each}
            {/if}
          </div>
          <div class="note-snippet">{snippet(note.content)}</div>
        </div>
        <button class="category-remove" title="Delete note" on:click|stopPropagation={() => deleteNote(note.id)}>✕</button>
      </div>
    {/each}
  </div>
  <button class="fab" aria-label="Add Note" on:click={addNote} title="Add Note">+</button>
</div>

{#if showModal}
  <div class="modal-bg" on:click={() => showModal = false}>
    <div class="modal" on:click|stopPropagation>
      <button class="close-btn" aria-label="Close" on:click={() => showModal = false}>&times;</button>
      <h2>{editing ? 'Edit Note' : 'New Note'}</h2>
      <label for="note-title">Title</label>
      <input id="note-title" type="text" bind:value={currentNote.title} maxlength="80" autocomplete="off" spellcheck="true" placeholder="Note title…" />
      <label for="note-content">Content</label>
      <div class="toolbar" style="margin-bottom:6px;">
        <button title="Bold" type="button" on:click={() => formatSelected('b')}><b>B</b></button>
        <button title="Italic" type="button" on:click={() => formatSelected('i')}><i>I</i></button>
        <button title="Bullets" type="button" on:click={() => formatSelected('ul')}><span style="font-size:1.01em;">• List</span></button>
      </div>
      <textarea
        id="note-content"
        rows="6"
        bind:value={currentNote.content}
        style="min-height:95px;resize:vertical;"
        on:keydown={onEditorKey}
        autocomplete="off"
        spellcheck="true"
        placeholder="Note details… Supports bold, italic, bullets."
      ></textarea>
      <label for="note-cat">Categories / Tags</label>
      <div class="category-input-group">
        <input id="note-cat" type="text" bind:value={categoryInput} maxlength="22" style="flex:1"
          placeholder="Add category…" list="suggest-cats"
          on:keydown={(e) => { if (e.key==='Enter'){ addCategory(); e.preventDefault(); } }}/>
        <button type="button" on:click={addCategory} style="border-radius:6px;font-size:1.16em; background:{COLOR_PRIMARY}; color:white;">+</button>
        <datalist id="suggest-cats">
          {#each Array.from(new Set(notes.flatMap(x=>x.categories))) as catOption}
            <option value={catOption}>{catOption}</option>
          {/each}
        </datalist>
      </div>
      <div class="note-categories">
        {#each currentNote.categories as cat, i}
          <span class="category-label" style="background: {CATEGORY_COLORS[i%CATEGORY_COLORS.length]};">
            {cat}
            <button class="category-remove" title="Remove tag" type="button" on:click={() => removeCategory(cat)}>×</button>
          </span>
        {/each}
      </div>
      <div class="actions">
        {#if editing}
          <button class="delete" type="button" on:click={() => { deleteNote(currentNote.id); showModal=false; }}>Delete</button>
        {/if}
        <button class="cancel" type="button" on:click={() => showModal=false}>Cancel</button>
        <button type="submit" on:click={saveNote} style="background:{COLOR_PRIMARY};">{editing ? 'Update' : 'Save'}</button>
      </div>
    </div>
  </div>
{/if}
