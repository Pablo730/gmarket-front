<template>
  <v-container>
    <v-card
      class="mx-auto my-3"
      max-width="1000">
      <v-container
        v-if="$store.state.userType === 'MANAGER'"
        class="d-flex justify-end ma-4">
        <base-write-post-button
          v-if="$store.state.userType === 'MANAGER'"
          fromBoard="notice">
        </base-write-post-button>
      </v-container>
      <v-row
        class="mx-6">
        <v-col cols="1">번호</v-col>
        <v-col cols="7">제목</v-col>
        <v-col cols="1">글쓴이</v-col>
        <v-col cols="2">작성일</v-col>
        <v-col cols="1">조회</v-col>
      </v-row>
      <v-card
        class="ma-1"
        elevation="1"
        v-for="board in boards"
        :key="board.board_id">
        <router-link 
          style="text-decoration: none; color: inherit;"
          :to="{name: '게시판', 
                params: {boardId : board.board_id},
                query: {board: 'notice'}}">
          <v-row
            class="mx-6">
            <v-col cols="1" v-text="board.board_id"></v-col>
            <v-col cols="7" v-text="board.title"></v-col>
            <v-col cols="1" v-text="board.name"></v-col>
            <v-col cols="2" v-text="dateFormatter(board.created_at)"></v-col>
            <v-col cols="1">1</v-col>
          </v-row>
        </router-link>
      </v-card>
      <v-divider class="my-3"/>
      <v-pagination
        v-model="page"
        :length="pageLength"
        :total-visible="10">
      </v-pagination>
    </v-card>
  </v-container>
</template>

<script>
export default {
  data: () => ({
    page: 1,
    pageLength: 1,
    boards: []
  }),
  watch: {
    page: function() {
      this.getNoticeBoards();
      this.$vuetify.goTo('#header');
    }
  },
  methods: {
    dateFormatter(date) {
      if(date === null || date === '') {
        return 'error';
      }

      return this.$moment(date).format('YYYY-MM-DD HH:mm')
    },
    async getNoticeBoards() {

      const APIURL = process.env.VUE_APP_API_URL;
      
      try {
        const res = await this.$axios.get(`${APIURL}/boards/notice/posts?page=${this.page}`);
        this.boards = res.data.data.board_list;
        this.pageLength = res.data.data.total_page
      } catch(err) {
        alert(`불러오기 실패\nerr: ${err}`)
      }
    }
  },
  mounted() {
    this.getNoticeBoards();
  }
}
</script>