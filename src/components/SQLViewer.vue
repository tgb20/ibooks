<script setup lang="ts">
import initSqlJs from "sql.js";
import { ref } from "vue";
import { Check, Copy } from "lucide-vue-next";
import { Button } from "@/components/ui/button";

import { useClipboard } from "@vueuse/core";

const { copy } = useClipboard();

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
  ZBOOKDESCRIPTION?: string;
  ZVERSIONNUMBER?: number;
  ZRELEASEDATE?: number;
  ZGENRE?: string;
  ZSTOREID?: number;
  ZCOVERASPECTRATIO?: number;
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
      "select ZTITLE, ZAUTHOR, ZISFINISHED, ZDATEFINISHED, ZPURCHASEDATE, ZREADINGPROGRESS, ZLASTOPENDATE, ZEPUBID, ZURL, ZBOOKDESCRIPTION, ZVERSIONNUMBER, ZRELEASEDATE, ZGENRE, ZSTOREID, ZCOVERASPECTRATIO from ZBKLIBRARYASSET where ZCONTENTTYPE = 1 and ZPURCHASEDATE not null order by ZLASTOPENDATE desc;"
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
      book.ZBOOKDESCRIPTION = row[9]?.toString();
      book.ZVERSIONNUMBER = Number(row[10]?.toString());
      book.ZRELEASEDATE = Number(row[11]?.toString());
      book.ZGENRE = row[12]?.toString();
      book.ZSTOREID = Number(row[13]?.toString());
      book.ZCOVERASPECTRATIO = Number(row[14]?.toString());
      return book;
    });

    console.log(books);
  };
  r.readAsArrayBuffer(f);
}

const showCheck = ref(false);

function copyPath() {
  copy("~/Library/Containers/com.apple.iBooksX/Data/Documents/BKLibrary");
  showCheck.value = true;
  setTimeout(() => {
    showCheck.value = false;
  }, 500);
}

function getDate(date: number) {
  return new Date((date + 978307200) * 1000).toLocaleDateString();
}
</script>
<template>
  <div class="mt-4 container mx-auto text-center flex flex-col gap-4 w-full">
    <h1 class="text-4xl">iBooks Info</h1>
    <div class="mx-auto flex w-full max-w-[23rem] gap-2">
      <input
        id="npm-install"
        type="text"
        class="col-span-6 bg-gray-50 border border-gray-300 text-gray-500 text-sm rounded-lg block w-full p-2.5"
        value="~/Library/Containers/com.apple.iBooksX/Data/Documents/BKLibrary"
        disabled
        readonly
      />
      <Button @click="copyPath()" variant="outline" size="icon">
        <Copy v-if="!showCheck" class="w-4 h-4" />
        <Check v-if="showCheck" class="w-4 h-4" />
      </Button>
    </div>
    <p>Open Finder, Go → Go To Folder... (⇧⌘G)</p>
    <div
      class="mx-auto max-w-sm w-full h-20 rounded-xl border-dashed border-4 text-center content-center text-slate-500"
    >
      <p>Upload .sqlite</p>
      <input ref="fileElement" type="file" />
    </div>
    <Button class="mx-auto max-w-sm w-full" @click="process">Read</Button>
    <div
      class="mx-auto mt-10 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 grid-flow-row gap-4"
    >
      <div v-for="book in books" class="w-full max-w-sm bg-red-400 rounded-sm">
        {{ book.ZTITLE }}
      </div>
    </div>
  </div>
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
