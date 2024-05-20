<template>
  <div>
    <div v-if="!editingNote">
      <input type="text" v-model="newNote" placeholder="Enter note">
      <button @click="addNote">Add Note</button>
    </div>
    <div v-else>
      <input type="text" v-model="editingNote.title" placeholder="Edit note">
      <button @click="updateNote">Update Note</button>
    </div>
    <ul>
      <li v-for="note in notes" :key="note._id">
        {{ note.title }}
        <button @click="deleteNote(note)">Delete</button>
        <button @click="editNote(note)">Edit</button>
      </li>
    </ul>
  </div>
</template>

<script>
import PouchDB from 'pouchdb';
export default {
  name: 'HelloWorld',
  data() {
    return {
      notes: [],
      newNote: '',
      editingNote: null,
      db: new PouchDB('my-notes'),
    };
  },
  mounted() {
    this.getNotes();
  },
  methods: {
    getNotes() {
      this.db.allDocs({ include_docs: true }).then((result) => {
        this.notes = result.rows.map((row) => row.doc);
      }).catch((err) => {
        console.error(err);
      });
    },
    addNote() {
      if (!this.newNote.trim()) return;

      const note = {
        _id: new Date().toISOString(),
        title: this.newNote,
      };

      this.db.put(note).then(() => {
        this.notes.push(note);
        this.newNote = '';
      }).catch((err) => {
        console.error(err);
      });
    },
    deleteNote(note) {
      this.db.get(note._id).then((latestNote) => {
        return this.db.remove(latestNote);
      }).then(() => {
        this.notes = this.notes.filter((n) => n._id !== note._id);
      }).catch((err) => {
        console.error(err);
      });
    },
    editNote(note) {
      this.editingNote = { ...note };
    },
    updateNote() {
      if (!this.editingNote || !this.editingNote.title.trim()) return;

      this.db.put(this.editingNote).then(() => {
        const index = this.notes.findIndex((note) => note._id === this.editingNote._id);
        if (index !== -1) {
          this.notes.splice(index, 1, this.editingNote);
        }
        this.editingNote = null;
      }).catch((err) => {
        console.error(err);
      });
    },
  },
}
</script>

<style>
/* Add some basic styling */
ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin: 0.5em 0;
}

button {
  margin-left: 0.5em;
}
</style>
