<script>
import Editor from "@toast-ui/editor";
import "@toast-ui/editor/dist/toastui-editor.css";
import { apiClient } from "../utils/axios.js";
import commonUtil from "../utils/common-util.js";
import { onMounted } from "vue";
import { CONSTANTS } from "../utils/constants.js";
import router from "../router/index.js";

export default {
  components: {
    editor: Editor,
  },
  
  data() {
    return {
      title: "",
      selectedPhoto: null,
      isEditable: false,
    };
  },

  mounted() {
    const editor = new Editor({
      el: this.$refs.editorRef,
      initialEditType: "markdown",
      previewStyle: "vertical",
      height: "500px",
    });

    //  초기 텍스트 설정
    editor.setMarkdown("");

    this.editor = editor;

    // 에디터 내용 변경 이벤트 처리
    editor.on("change", () => {
      const content = editor.getMarkdown();
      console.log("Editor content:", content);
    });
  },
  methods: {
  async saveContent() {
  const title = this.title;
  const contents = this.editor.getMarkdown();
  if (title) {
        if (contents) {
          await apiClient("board/updateBoard", {
            postId: location.search,
            boardTitle: title,
            boardContents: contents,
            userIdx: JSON.parse(
              commonUtil.getLocalStorage(CONSTANTS.KEY_LIST.USER_INFO)
            ).userIdx,
          })
            .then((r) => {
              alert("글 수정 완료!");
              router.push("/");
            })
            .catch((e) => {
              alert("글이 수정되지 않았습니다.");
            });
        } else alert("내용을 입력해주세요");
      } else alert("제목을 입력해주세요");
    },
    handleFileChange(event) {
      const file = event.target.files[0];
      this.selectedPhoto = URL.createObjectURL(file);
    },
    savePhoto() {
      // 선택한 사진을 저장하는 로직을 구현하세요
      // 예: 서버에 사진을 업로드하거나, 상태 관리를 통해 홈 페이지에서 사용할 수 있도록 전달
      console.log(this.selectedPhoto);
    },
  },
  setup() {
    onMounted(async () => {
      const loginCheck = commonUtil.loginCheck();
      if (!loginCheck) {
        alert("로그인 후에 이용해주세요");
        await router.push("/");
      }
    });
  },
};
</script>


<template>
  <div class="edit-container">
    <input type="text" v-model="title" placeholder="제목" />
    <div class="editdiv" style="width: 100%" ref="editorRef" />
    <button class="upload" @click="saveContent">저장하기</button>
  </div>
</template>

<style scoped></style>
