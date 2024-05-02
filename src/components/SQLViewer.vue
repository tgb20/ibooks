<script setup lang="ts">
import initSqlJs from "sql.js";
import { ref } from "vue";

interface BookEntry {
  ZTITLE?: string;
  ZAUTHOR?: string;
  ZISFINISHED?: boolean;
  ZDATEFINISHED?: number;
  ZPURCHASEDATE?: number;
  ZREADINGPROGRESS?: number;
  ZLASTOPENDATE?: number;
  ZEPUBID?: string;
  ZURL?: string;
}

const books = ref<BookEntry[]>([]);

const fileElement = ref();

const SQL = await initSqlJs({
  locateFile: (file) => `https://sql.js.org/dist/${file}`,
});

let db;

function process() {
  const f = fileElement.value.files[0];
  const r = new FileReader();
  r.onload = function () {
    if (r.result === null) {
      return;
    }
    //@ts-ignore
    const Uints = new Uint8Array(r.result);
    db = new SQL.Database(Uints);

    const query = db.exec(
      "select ZTITLE, ZAUTHOR, ZISFINISHED, ZDATEFINISHED, ZPURCHASEDATE, ZREADINGPROGRESS, ZLASTOPENDATE, ZEPUBID, ZURL from ZBKLIBRARYASSET where ZCONTENTTYPE = 1 and ZPURCHASEDATE not null order by ZLASTOPENDATE desc;"
    );

    books.value = query[0].values.map((row) => {
      let book: BookEntry = {};
      book.ZTITLE = row[0]?.toString();
      book.ZAUTHOR = row[1]?.toString();
      book.ZISFINISHED = Boolean(row[2]?.toString());
      book.ZDATEFINISHED = Number(row[3]?.toString());
      book.ZPURCHASEDATE = Number(row[4]?.toString());
      book.ZREADINGPROGRESS = Number(row[5]?.toString());
      book.ZLASTOPENDATE = Number(row[6]?.toString());
      book.ZEPUBID = row[7]?.toString();
      book.ZURL = row[8]?.toString();
      return book;
    });

    console.log(books);
  };
  r.readAsArrayBuffer(f);
}

function getDate(date: number) {
  return new Date((date + 978307200) * 1000).toLocaleDateString();
}
</script>
<template>
  <h1>iBooks to CSV</h1>
  <p>Open Finder</p>
  <p>Go -> Go To Folder</p>
  <pre>⇧⌘G</pre>
  <pre>~/Library/Containers/com.apple.iBooksX/Data/Documents/BKLibrary</pre>
  <input ref="fileElement" type="file" />
  <button @click="process">Process</button>
  <table>
    <thead>
      <tr>
        <th>Title</th>
        <th>Author</th>
        <th>Finished</th>
        <th>Date Finished</th>
        <th>Purchase Date</th>
        <th>Reading Progress</th>
        <th>Last Opened Date</th>
        <th>EPUB ID</th>
        <th>URL</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="book in books" :key="book.ZTITLE">
        <td>{{ book.ZTITLE }}</td>
        <td>{{ book.ZAUTHOR }}</td>
        <td>{{ book.ZISFINISHED }}</td>
        <td>{{ getDate(book.ZDATEFINISHED ?? 0) }}</td>
        <td>{{ getDate(book.ZPURCHASEDATE ?? 0) }}</td>
        <td>{{ book.ZREADINGPROGRESS }}</td>
        <td>{{ getDate(book.ZLASTOPENDATE ?? 0) }}</td>
        <td>{{ book.ZEPUBID }}</td>
        <td>{{ book.ZURL }}</td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped>
table,
th,
td {
  border: 1px solid;
}

table {
  border-collapse: collapse;
}
</style>
