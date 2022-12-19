<template>
  <div id="app">
    <link
      href="https://cdn.jsdelivr.net/npm/remixicon@2.2.0/fonts/remixicon.css"
      rel="stylesheet"
    />
    <div id="text-editor">
      <div class="toolbar" v-if="editor">
        <div class="align-dropdown">
          <button class="dropbtn">
            {{ headingIdx ? `H${headingIdx}` : `Heading ▼` }}
          </button>
          <div class="dropdown-content">
            <a
              v-for="index in 6"
              :key="index"
              :class="{ active: editor.isActive('heading', { level: index }) }"
              :style="{ fontSize: 20 - index + 'px' }"
              role="button"
              @click="onHeadingClick(index)"
            >
              H{{ index }}
            </a>
          </div>
        </div>
        <button
          v-for="({ slug, option, active, icon }, index) in textActions"
          :key="index"
          :class="{ active: editor.isActive(active) }"
          @click="onActionClick(slug, option)"
        >
          <i :class="icon"></i>
        </button>
      </div>
      <EditorContent :editor="editor" />
      <div v-if="editor" class="footer">
        <span class="characters-count" :class="maxLimit ? limitWarning : ''">
          {{ charactersCount }}
          {{ maxLimit ? `/ ${maxLimit} characters` : "characters" }}
        </span>
        |
        <span class="words-count"> {{ wordsCount }} words </span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { Editor, EditorContent } from "@tiptap/vue-3";
import StarterKit from "@tiptap/starter-kit";
import TextAlign from "@tiptap/extension-text-align";
import Underline from "@tiptap/extension-underline";
import Subscript from "@tiptap/extension-subscript";
import Superscript from "@tiptap/extension-superscript";
import CharacterCount from "@tiptap/extension-character-count";

const props = defineProps({
  content: {
    type: String,
    default: "",
  },
  maxLimit: {
    type: Number,
    default: 200,
  },
});

const maxLimit = ref<number>(props.maxLimit);
const content = ref<string>(props.content);
const headingIdx = ref<Number>(0);
const editor = ref();
const textActions = ref([
  { slug: "bold", icon: "ri-bold", active: "bold" },
  { slug: "italic", icon: "ri-italic", active: "italic" },
  { slug: "underline", icon: "ri-underline", active: "underline" },
  { slug: "strike", icon: "ri-strikethrough", active: "strike" },
  {
    slug: "align",
    option: "left",
    icon: "ri-align-left",
    active: { textAlign: "left" },
  },
  {
    slug: "align",
    option: "center",
    icon: "ri-align-center",
    active: { textAlign: "center" },
  },
  {
    slug: "align",
    option: "right",
    icon: "ri-align-right",
    active: { textAlign: "right" },
  },
  {
    slug: "align",
    option: "justify",
    icon: "ri-align-justify",
    active: { textAlign: "justify" },
  },
  { slug: "bulletList", icon: "ri-list-unordered", active: "bulletList" },
  { slug: "orderedList", icon: "ri-list-ordered", active: "orderedList" },
  { slug: "subscript", icon: "ri-subscript-2", active: "subscript" },
  {
    slug: "superscript",
    icon: "ri-superscript-2",
    active: "superscript",
  },
  { slug: "undo", icon: "ri-arrow-go-back-line", active: "undo" },
  { slug: "redo", icon: "ri-arrow-go-forward-line", active: "redo" },
  { slug: "clear", icon: "ri-format-clear", active: "clear" },
  { slug: "code", icon: "ri-code-view", active: "code" },
]);

const charactersCount = computed(() => editor?.value.storage.characterCount.characters());
const wordsCount = computed(() => editor?.value.storage.characterCount.words());
const limitWarning = computed(() => {
  const isCloseToMax = +charactersCount >= maxLimit.value - 20;
  const isMax = +charactersCount === maxLimit.value;
  if (isCloseToMax && !isMax) return "warning";
  if (isMax) return "danger";
  return "";
});

const emit = defineEmits<{
  (e: "update-content", value: string): void;
}>();

// watch(
//   () => content.value,
//   (value) => {
//     if (editor.value.getHTML() === value) return;
//     editor.value.commands.setContent(content.value, false);
//   }
// );

function onActionClick(slug: string, option: unknown | string = null) {
  const vm: any = editor.value.chain().focus();
  const actionTriggers: any = {
    bold: () => vm.toggleBold().run(),
    italic: () => vm.toggleItalic().run(),
    underline: () => vm.toggleUnderline().run(),
    strike: () => vm.toggleStrike().run(),
    bulletList: () => vm.toggleBulletList().run(),
    orderedList: () => vm.toggleOrderedList().run(),
    align: () => vm.setTextAlign(option).run(),
    subscript: () => vm.toggleSubscript().run(),
    superscript: () => vm.toggleSuperscript().run(),
    undo: () => vm.undo().run(),
    redo: () => vm.redo().run(),
    clear: () => {
      vm.clearNodes().run();
      vm.unsetAllMarks().run();
    },
    code: () => vm.toggleCodeBlock().run(),
  };
  actionTriggers[slug]();
}
function onHeadingClick(index: number) {
  const vm = editor.value.chain().focus();
  vm.toggleHeading({ level: index }).run();
  headingIdx.value = index;
}
onMounted(() => {
  editor.value = new Editor({
    content: content.value,
    extensions: [
      StarterKit,
      Underline,
      Subscript,
      Superscript,
      CharacterCount.configure({
        limit: maxLimit.value,
      }),
      TextAlign.configure({
        types: ["heading", "paragraph"],
      }),
    ],
  });
});
onBeforeUnmount(() => {
  editor.value.destroy();
});

watch(
  () => editor.value?.getHTML(),
  () => emit("update-content", editor.value?.getHTML())
);
</script>

<style lang="scss" scoped>
#text-editor {
  border: 1px solid #808080;
  .toolbar {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    border-bottom: 1px solid #808080;
    .noneHeader {
      display: none;
    }
    > button {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 32px;
      height: 32px;
      font-size: 20px;
      background: #fff;
      color: #333;
      border: none;
      border-radius: 2px;
      margin: 0.5em 4px;
      -webkit-appearance: none;
      cursor: pointer;
      &.active {
        background: #333;
        color: #fff;
      }
    }
  }
  .align-dropdown {
    position: relative;
    display: inline-block;
    margin: 0.5em 8px;
    min-width: 78.563px;
    > button {
      height: 32px;
      background: #fff;
      color: #333;
      border: none;
      border-radius: 2px;
      -webkit-appearance: none;
      cursor: pointer;
      width: 100%;
      margin: 0 auto;
    }
    > .dropdown-content {
      display: none;
      position: absolute;
      left: 0;
      right: 0;
      border: 1px solid #333;
      outline: 1px solid #fff;
      border-radius: 2px;
      background-color: #fff;
      z-index: 1;
      a {
        display: block;
        padding: 6px 12px;
        text-align: center;
        cursor: pointer;
        &:hover,
        &.active {
          background: #333;
          color: #fff;
        }
      }
    }
    &:hover .dropdown-content {
      display: block;
    }
  }
  .divider {
    width: 1px;
    height: 24px;
    background: #333;
    margin-right: 6px;
  }
  .footer {
    color: #808080;
    font-size: 14px;
    text-align: right;
    padding: 6px;
    .characters-count {
      &.warning {
        color: orange;
      }
      &.danger {
        color: red;
      }
    }
  }
  .ProseMirror {
    height: 300px;
    overflow-y: auto;
    padding-left: 0.5em;
    padding-right: 0.5em;
    outline: none;
    > p:first-child {
      margin-top: 0.5em;
    }
    > h1,
    h2,
    h3,
    h4,
    h5,
    h6 {
      &:first-child {
        margin-top: 0.5em;
      }
    }
  }
}
</style>
