<script setup lang="ts">
import initSqlJs from "sql.js";
import { ref } from "vue";
import { Check, Copy } from "lucide-vue-next";
import { Button } from "@/components/ui/button";
import {
  Table,
  TableBody,
  TableCaption,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";

import { useClipboard } from "@vueuse/core";
import { useDropZone } from "@vueuse/core";

const { copy } = useClipboard();

interface BookEntry {
  ZTITLE?: string;
  ZAUTHOR?: string;
  ZISFINISHED?: boolean;
  ZDATEFINISHED?: number;
  ZPURCHASEDATE?: number;
  ZREADINGPROGRESS?: number;
  ZEPUBID?: string;
  ZCOVERASPECTRATIO?: number;
}

const books = ref<BookEntry[]>([]);

const dropZoneRef = ref<HTMLDivElement>();

const SQL = await initSqlJs({
  locateFile: (file) => `https://sql.js.org/dist/${file}`,
});

const fileName = ref("Upload BKLibrary-SOMENUMBERS.sqlite");

let db;

function onDrop(files: File[] | null) {
  if (files && files.length > 0) {
    const file = files[0];
    fileName.value = file.name;
    if (file.name.startsWith("BKLibrary") && file.name.endsWith(".sqlite")) {
      process(file);
    }
  }
}

useDropZone(dropZoneRef, {
  onDrop,
});

function process(file: File) {
  const f = file;
  const r = new FileReader();
  r.onload = function () {
    if (r.result === null) {
      return;
    }
    //@ts-ignore
    const Uints = new Uint8Array(r.result);
    db = new SQL.Database(Uints);

    const query = db.exec(
      "select ZTITLE, ZAUTHOR, ZISFINISHED, ZDATEFINISHED, ZPURCHASEDATE, ZREADINGPROGRESS, ZEPUBID, ZCOVERASPECTRATIO from ZBKLIBRARYASSET where ZCONTENTTYPE = 1 and ZPURCHASEDATE not null order by ZLASTOPENDATE desc;"
    );

    books.value = query[0].values.map((row) => {
      let book: BookEntry = {};
      book.ZTITLE = row[0]?.toString();
      book.ZAUTHOR = row[1]?.toString();
      book.ZISFINISHED = Boolean(row[2]?.toString());
      book.ZDATEFINISHED = Number(row[3]?.toString());
      book.ZPURCHASEDATE = Number(row[4]?.toString());
      book.ZREADINGPROGRESS = Number(row[5]?.toString());
      book.ZEPUBID = row[7]?.toString();
      book.ZCOVERASPECTRATIO = Number(row[14]?.toString());
      return book;
    });
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
    <div class="mx-auto flex w-full max-w-md gap-2">
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
      ref="dropZoneRef"
      class="mx-auto max-w-sm w-full h-20 rounded-xl border-dashed border-4 text-center content-center text-slate-500"
    >
      <p>{{ fileName }}</p>
    </div>
    <Table v-if="books.length > 0" class="mx-auto w-full max-w-md">
      <TableCaption>A list of your Apple Books.</TableCaption>
      <TableHeader>
        <TableRow>
          <TableHead>Title</TableHead>
          <TableHead>Author</TableHead>
          <TableHead>Date Purchased</TableHead>
          <TableHead>Finished?</TableHead>
          <TableHead>Date Finished</TableHead>
          <TableHead>% Read</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        <TableRow v-for="book in books" :key="book.ZEPUBID">
          <TableCell class="font-medium">
            {{ book.ZTITLE }}
          </TableCell>
          <TableCell>{{ book.ZAUTHOR }}</TableCell>
          <TableCell>{{ getDate(book.ZPURCHASEDATE ?? 0) }}</TableCell>
          <TableCell>{{ book.ZISFINISHED ? "✅" : "❌" }}</TableCell>
          <TableCell v-if="book.ZISFINISHED">{{
            getDate(book.ZDATEFINISHED ?? 0)
          }}</TableCell>
          <TableCell v-else="book.ZISFINISHED">Not Finished</TableCell>
          <TableCell
            >{{ ((book.ZREADINGPROGRESS ?? 0) * 100).toFixed(0) }}%</TableCell
          >
        </TableRow>
      </TableBody>
    </Table>
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
